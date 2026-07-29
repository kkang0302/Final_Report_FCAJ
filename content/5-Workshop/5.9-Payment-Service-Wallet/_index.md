---
title : "Payment Service & Wallet"
date : 2024-01-01 
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

# Deploying the Payment Service & Stripe-Integrated Shared Wallet

The financial engine of CourShare is the **Payment Service** (Node.js/Express, connecting to the RDS PostgreSQL schema `payment_db`). This service handles deposits/withdrawals using the **Stripe** payment gateway and processes wallet-to-wallet transactions when buying/selling courses.

## 1. Database Design for Shared Wallet
We design two critical database tables within the `payment_db` schema (using Prisma ORM):
* Bảng `wallets`: Stores the balance (`balance`) of each user (`userId`). A single user account maintains both loaded cash balances and earnings accrued from selling courses.
* Bảng `transactions`: Records transaction histories. Transaction types include: `DEPOSIT` (adding funds via Stripe), `WITHDRAW` (withdrawing to bank), `PURCHASE` (buying a course - debiting the student), and `EARN` (selling a course - crediting the instructor).

## 2. Wallet Funding Workflow (Stripe Integration)
When a user wishes to deposit real currency into their internal wallet:

1. **Create Checkout Session:** The client calls the Payment Service API, which uses the Stripe SDK to instantiate a checkout page:
   ```javascript
   const session = await stripe.checkout.sessions.create({
     payment_method_types: ['card'],
     line_items: [{
       price_data: {
         currency: 'usd',
         product_data: { name: 'CourShare Wallet Deposit' },
         unit_amount: amount * 100, // Value in cents
       },
       quantity: 1,
     }],
     mode: 'payment',
     success_url: `${process.env.FRONTEND_URL}/payment/success`,
     cancel_url: `${process.env.FRONTEND_URL}/payment/cancel`,
     metadata: { userId }, // Metadata passed to identify the user upon webhook callback
   });
   ```
2. The backend responds with `session.url` and the React frontend redirects the user to Stripe's secure checkout page.

## 3. Stripe Webhook Endpoint (Updating Wallet Balance)
Once the user enters their card info and executes the payment, Stripe fires an HTTP POST request (Webhook) to the API Gateway/Payment Service at the `/webhook` endpoint.

This endpoint processes:
1. **Signature Verification:** Verifies the webhook signature using your Stripe webhook secret to prevent spoofing:
   ```javascript
   const sig = req.headers['stripe-signature'];
   const event = stripe.webhooks.constructEvent(req.body, sig, process.env.WEBHOOK_SECRET);
   ```
2. **Event Processing:** When receiving the `checkout.session.completed` event:
   - Extract the `userId` and `amount` from the session metadata.
   - Run a PostgreSQL transaction: increment the user's balance in the `wallets` table and insert a new `DEPOSIT` record in `transactions`.

## 4. Purchasing Workflow (Internal Wallet-to-Wallet Transactions)
When a student decides to purchase a course from an instructor:
1. The client issues a purchase request containing `courseId` and `instructorId` to the Payment Service.
2. The Payment Service executes a SQL transaction:
   - Check the buyer's wallet balance. If insufficient, reject the transaction.
   - Debit the buyer's wallet balance.
   - Credit the instructor's wallet balance (`instructorId`).
   - Create two transaction ledger entries: `PURCHASE` (for the student) and `EARN` (for the instructor).
3. **Ensuring Eventual Consistency:**
   After the local database transaction commits successfully, the Payment Service publishes a message containing `userId` and `courseId` to the SQS queue. The **Enrollment Service** polls this queue and automatically registers the student, unlocking the course. This asynchronous message flow ensures data integrity and robustness even in case of transient network faults.

<!-- TODO: chèn screenshot - [Stripe Checkout page displaying invoice details for the CourShare Wallet Deposit] -->

<!-- TODO: chèn screenshot - [Stripe CLI terminal output forwarding webhook events to local port 8084 during development and debugging] -->

<!-- TODO: chèn screenshot - [Database query showing the wallets table updated correctly: debited student wallet and credited instructor wallet balances] -->
