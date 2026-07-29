---
title : "Authentication with Cognito"
date : 2024-01-01 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

# User Identity and Authentication Management with Amazon Cognito

The CourShare platform manages user identity centrally using an **Amazon Cognito User Pool**. We configure a single authentication layer supporting both student and instructor roles under one user account, enabling users to seamlessly switch roles.

## Step 1: Initialize the Cognito User Pool
1. Navigate to the **Cognito Console** -> **Create user pool**.
2. **Configure sign-in experience:**
   - Select **Cognito user pool**.
   - Sign-in providers: Select **Email** (allowing users to log in using their email addresses).
3. **Configure security requirements:**
   - Password policy: Keep default settings or customize complexity.
   - Multi-factor authentication (MFA): Select **No MFA** for testing purposes (or set to Optional).
   - User recovery: Select **Email only**.
4. **Configure sign-up experience:**
   - Enable **Self-service sign-up** (permitting users to register themselves).
   - Attribute verification: Select **Send email message, verify email address**.
5. **Configure message delivery:**
   - Email: Select **Send email with Cognito** to use AWS's default transactional mailer (capped at 50 emails/day, optimal for testing).

## Step 2: Add the Custom Attribute (custom:role)
To support our shared wallet model with dual roles (Student and Instructor on the same account), we configure a custom metadata attribute:
1. In the **Configure attributes** step, click **Add custom attribute**.
2. Attribute name: `role`.
3. Type: **String**.
4. Min length: `1`, Max length: `20`.
5. Ensure the attribute is set to mutable so instructors can update their roles later.

## Step 3: Configure the App Client
1. In the **Integrate app** step:
   - User pool name: `courshare-user-pool`.
   - App client name: `courshare-react-spa`.
   - Client secret: Select **Don't generate a client secret** (This option is mandatory because Frontend React SPAs run directly on client browsers and cannot securely store client secrets).
2. Complete the remaining steps and click **Create user pool**.

## Step 4: Validate JWT Tokens at the Identity Service
When the React client logs in successfully with Cognito, Cognito issues 3 tokens: ID Token, Access Token, and Refresh Token.
The client attaches the Access Token (JWT) in the headers: `Authorization: Bearer <JWT_TOKEN>` for subsequent API requests.

The Identity Service (Java Spring Boot) validates this token signature by configuring Spring Security:
1. Spring Boot retrieves the JWK Set URI from Cognito:
   ```yaml
   spring:
     security:
       oauth2:
         resourceserver:
           jwt:
             issuer-uri: https://cognito-idp.us-east-1.amazonaws.com/us-east-1_xxxxxx
             jwk-set-uri: https://cognito-idp.us-east-1.amazonaws.com/us-east-1_xxxxxx/.well-known/jwks.json
   ```
2. The Spring Security framework automatically pulls Cognito's public verification keys to assert token validity and expiration.
3. User claims like `username`, `email`, and `custom:role` are parsed to enforce role-based access control.

<!-- TODO: chèn screenshot - [Amazon Cognito User Pool Console displaying the User Pool ID and App Client ID parameters] -->

<!-- TODO: chèn screenshot - [Cognito App Client Attributes configuration screen showing read and write permissions enabled for custom:role] -->
