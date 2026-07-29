---
title: "Week 3 Worklog"
date: 2026-06-29
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

* Develop the remaining 3 microservices (Payment, Enrollment, Learning Services) using Node.js/Express and Prisma ORM.
* Integrate the third-party payment gateway Stripe to handle course purchases.
* Write unit tests using Jest to ensure stability and accuracy of the Node.js APIs.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | - Develop the Payment Service: integrate Stripe SDK to create Checkout Sessions for online purchases | 29/06/2026 | 29/06/2026 | Stripe API Reference |
| 2 | - Set up REST API endpoint to listen to Stripe Webhook events (`checkout.session.completed`) to update payment status and record transactions | 30/06/2026 | 30/06/2026 | Stripe Webhook Developers Guide |
| 3 | - Develop the Enrollment Service: construct APIs to verify course enrollment status and record registration entries upon payment | 01/07/2026 | 01/07/2026 | Enrollment System Logic |
| 4 | - Develop the Learning Service: build APIs to mark lessons as completed and calculate overall course progress percentage | 02/07/2026 | 02/07/2026 | Learning Progress Specifications |
| 5 | - Write and run Unit Tests using the Jest framework for the 3 Node.js microservices before Docker packaging | 03/07/2026 | 03/07/2026 | Jest Testing Framework Documentation |

### Week 3 Achievements:

* Finished the source code for all 5 microservices, with local integrated services interacting smoothly.
* Successfully integrated Stripe payment gateway and tested webhooks locally using Stripe CLI.
* APIs for Payment, Enrollment, and Learning Services were verified for quality via the Jest test suite.

<!-- TODO: Insert screenshot - [Local Stripe Webhook CLI dashboard showing event logs for checkout.session.completed with status 200] -->
<!-- TODO: Insert screenshot - [Terminal window displaying successful Jest test suite runs for Node.js services] -->
