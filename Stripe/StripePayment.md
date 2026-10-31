# Stripe Payment Integration in Spring Boot

Let's learn Stripe from the beginning and then implement a complete payment flow in Spring Boot with code.

We'll cover it in this order:

1. What is Stripe?
2. How online payments work.
3. Important Stripe concepts.
4. How Stripe works internally.
5. How to integrate Stripe with Spring Boot.
6. Complete code implementation.
7. How to test payments.
8. Important things to remember in real-world applications.

We'll use Java + Spring Boot + Stripe Java SDK.

# 1. What is Stripe?

Stripe is a payment processing platform that allows businesses to accept online payments from customers.

Suppose you build an e-commerce application.

A customer:

- Selects a product worth ₹2,000.
- Adds it to the cart.
- Clicks Pay Now.
- Enters card details.
- Completes the payment.

But how does your Spring Boot application actually receive that money?

You cannot directly connect your Java application to every bank, card network, and payment provider.

This is where Stripe comes in.

Stripe acts as a middleman between your application, the customer, and the payment system.

[What is Stripe, how does it work, and is it right for your business?](https://images.openai.com/static-rsc-4/C0PqhVupK1gG8dNwWFjYQTyaHixWdbvVAjPErXS9Vrce4e0u6peKYKrDyCmhCeXB-MdiCLu8qTLeK74Ueh6NPu_B0t4iLd2pTjmQpnfbOt62PzZlx73rRPrU8NWP0ug7tuN99awztqejl2NFJxswOscecyI3GlgPf0BLevWwoKk?purpose=inline)

[whop.com](https://whop.com/blog/what-is-stripe/?utm_source=chatgpt.com)

For example:

1. Customer enters payment details.
2. Stripe securely processes the payment.
3. Stripe communicates with the payment networks and banks.
4. The bank approves or declines the transaction.
5. Stripe informs your application about the payment result.

Your application doesn't need to handle the customer's raw card details.

In simple words: Stripe provides APIs and services that allow your application to accept and manage online payments.

# 2. How does an online payment actually work?

Let's understand this with a real-world example.

Suppose you are building an e-commerce application.

A customer wants to purchase a laptop for ₹50,000.

[From Authorization to Settlement: The Transaction Journey](https://images.openai.com/static-rsc-4/CVfL9F22WHaRWILx_xgtRNAeXwteTyQSNV6VvRfrcL99kHHuV0NSRIX7i890HowvjFqjD7gfVwYS4N7yfB6Ri5eM6c7YEr8MpsO0RPzvPWI3Siw5JPzCjt_qMWCR3a7R1aLQ9fraS5hhwhm_b0J0M-b2t1rHf8PzQd-VklQ7tH8?purpose=inline)

[processout.com](https://www.processout.com/blog/the-secret-life-of-a-transaction?utm_source=chatgpt.com)

The payment flow:

1. Customer clicks Pay Now.
2. Your Spring Boot backend creates a payment request with Stripe.
3. Stripe provides a secure payment experience.
4. Customer enters card details and confirms payment.
5. Stripe contacts the relevant payment network and bank.
6. The bank approves or declines the payment.
7. Stripe updates the payment status.
8. Your backend receives confirmation, usually through a webhook.
9. Your application updates the order status.

For example:

```
Customer
   |
   | Clicks Pay Now
   v
Spring Boot Backend
   |
   | Creates Payment
   v
Stripe
   |
   | Processes Payment
   v
Bank
   |
   | Payment Approved
   v
Stripe
   |
   | Sends Webhook
   v
Spring Boot Backend
   |
   | Updates Order Status
   v
Order = PAID
```

Important: A successful response when creating a payment does not mean the customer has paid. You must confirm the actual payment result.

# 3. Important Stripe concepts

Before writing code, understand these concepts. They are the foundation of the integration.

## 3.1 API Keys

Stripe provides API keys that allow your application to communicate with Stripe.

There are two commonly used keys:

| Key                        | Purpose          |
| -------------------------- | ---------------- |
| Publishable key (`pk_...`) | Used on frontend |
| Secret key (`sk_...`)      | Used on backend  |

Publishable key:

- Can be used in frontend JavaScript.
- Identifies your Stripe account.
- Does not give permission to perform sensitive backend operations.

Secret key:

- Used in Spring Boot.
- Allows your backend to make authenticated Stripe API requests.
- Must never be exposed to frontend or GitHub.

Think of the secret key as a password for your Stripe account's API.

## 3.2 Checkout Session

A Checkout Session represents one customer's payment checkout process.

Suppose a customer wants to purchase a laptop.

Your backend creates a Checkout Session with details such as:

- Product name.
- Product price.
- Quantity.
- Currency.
- Success URL.
- Cancel URL.

Stripe returns a Checkout URL.

Your frontend redirects the customer to that URL.

Stripe displays its hosted payment page, where the customer enters payment details.

[docs.stripe.com](https://docs.stripe.com/payments/accept-a-payment?locale=en-GB\&utm_source=chatgpt.com)

Remember: Checkout Session is the object we will create in our Spring Boot application.

## 3.3 PaymentIntent

A PaymentIntent represents Stripe's process of collecting a payment.

For example:

```
Customer wants to pay ₹500
          |
          v
PaymentIntent created
          |
          v
Payment confirmation
          |
          v
Bank authentication
          |
          v
Payment successful
```

A PaymentIntent manages the payment lifecycle.

It can have different statuses:

| Status                    | Meaning                                |
| ------------------------- | -------------------------------------- |
| `requires_payment_method` | Payment method is needed               |
| `requires_confirmation`   | Payment needs confirmation             |
| `requires_action`         | Additional customer action is required |
| `processing`              | Payment is processing                  |
| `succeeded`               | Payment succeeded                      |
| `canceled`                | Payment was canceled                   |

Relationship: When you create a Checkout Session in payment mode, Stripe manages the associated PaymentIntent for you.

You don't need to manually create a PaymentIntent in our Checkout implementation.

## 3.4 PaymentMethod

PaymentMethod represents the payment method used by the customer.

Examples:

- Credit card.
- Debit card.
- Other supported payment methods.

Stripe securely manages sensitive payment information.

## 3.5 Webhook

This is one of the most important concepts.

Suppose a customer pays ₹500.

After payment, Stripe needs to inform your Spring Boot application that the payment succeeded.

But what if:

- The customer closes the browser?
- The internet disconnects?
- The customer never reaches your success page?

Your application still needs to know the payment result.

This is why Stripe provides Webhooks.

A webhook is an HTTP request sent by Stripe to an endpoint in your backend when a particular event happens.

For example:

```
Stripe
  |
  | Payment completed
  |
  | POST /api/payments/webhook
  v
Spring Boot
  |
  | Receives event
  |
  | Verifies event
  |
  | Updates database
  v
Order status = PAID
```

Some important Stripe events:

| Event                                      | Meaning                   |
| ------------------------------------------ | ------------------------- |
| `checkout.session.completed`               | Checkout completed        |
| `checkout.session.async_payment_succeeded` | Delayed payment succeeded |
| `checkout.session.async_payment_failed`    | Delayed payment failed    |
| `payment_intent.succeeded`                 | Payment succeeded         |
| `payment_intent.payment_failed`            | Payment failed            |

We will implement a webhook in our project.

# 4. Which Stripe API should we use?

Stripe offers different ways to integrate payments.

| Integration       | When to use                                    |
| ----------------- | ---------------------------------------------- |
| Checkout Sessions | Most normal e-commerce applications            |
| PaymentIntents    | Custom payment UI and advanced payment control |
| SetupIntents      | Save a payment method for future payments      |
| Stripe Billing    | Subscriptions and recurring payments           |
| Stripe Connect    | Marketplaces and payments to multiple sellers  |

For our Spring Boot e-commerce application, we will use Checkout Sessions.

Why?

Because Stripe handles most of the checkout experience, including the payment form and authentication.

We don't have to build the entire payment form ourselves.

This is also the recommended approach for most standard checkout integrations.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



# 5. Let's implement Stripe in Spring Boot

Practical implementation

We will build a simple payment system.

Our example:

A customer purchases a product worth ₹500.

Our application will:

1. Receive a checkout request.
2. Create a Checkout Session using Stripe.
3. Return the Checkout URL.
4. Redirect the customer to Stripe.
5. Customer completes payment.
6. Stripe sends a webhook.
7. Spring Boot receives the webhook.
8. Our application updates the order status.
[Integrating Stripe Checkout a complete practical Guide](https://images.openai.com/static-rsc-4/6w9cfpSo-LhjH0h3DsHCbh4IrgtqwavGQO46hSQ7R1PKY6h5-ULumH0HWbmbE5Ps2n7MDuUiVnFBH9HhHImfJIgQtL1ZDMqgsVgk-2ogUgZ67qipwORK0azdhgysfAe1cNz0tJkRblgV87kAT2hRcC1c5-4nVgsNc6P9kVp099k?purpose=inline)
[linkedin.com](https://www.linkedin.com/pulse/integrating-stripe-checkout-complete-practical-guide-singh-srjlc?utm_source=chatgpt.com)



## Step 1. Create a Stripe account

Open:

[Stripe Dashboard – Create account](https://dashboard.stripe.com/register?utm_source=chatgpt.com)

After creating your account:
1. Open the Stripe Dashboard.
2. Enable Test Mode.
3. Navigate to Developers → API keys.
4. Copy your test publishable key.
5. Copy your test secret key.


You will get keys similar to:

```
Publishable key:
pk_test_xxxxxxxxx

Secret key:
sk_test_xxxxxxxxx
```

These are examples, not actual keys.

We will use the secret key in Spring Boot.

## Step 2. Add Stripe dependency

Open your `pom.xml`.

Add the official Stripe Java SDK:

```

<dependency>
    <groupId>com.stripe</groupId>
    <artifactId>stripe-java</artifactId>
    <version>34.1.0</version>
</dependency>

```

This SDK provides Java classes and methods to communicate with Stripe APIs.

For example:

```
StripeClient client = new StripeClient(secretKey);
```

Instead of manually writing HTTP requests, we can use Stripe's Java SDK.

The version above is listed in the official SDK repository.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



## Step 3. Configure Stripe

In `application.properties`:

```
stripe.secret-key=${STRIPE_SECRET_KEY}

stripe.webhook-secret=${STRIPE_WEBHOOK_SECRET}
```

We are reading keys from environment variables instead of directly writing them in our source code.

For local Windows CMD, you can set the secret key like this:

```
set STRIPE_SECRET_KEY=sk_test_your_key
```

Remember: this command sets the variable only for the current CMD session.

## Step 4. Create StripeConfig

Create:

`config/StripeConfig.java`

```

package com.example.demo.config;

import com.stripe.StripeClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class StripeConfig {

    @Value("${stripe.secret-key}")
    private String secretKey;

    @Bean
    public StripeClient stripeClient() {

        // Creates a Stripe client using our secret key
        return new StripeClient(secretKey);
    }
}

```

Let's understand this.

Why do we create a StripeClient bean?

`StripeClient` is provided by the Stripe SDK.

It allows our application to communicate with Stripe.

By creating it as a Spring bean:

- Spring creates the object.
- Spring manages its lifecycle.
- We can inject it into our service using constructor injection.
- We don't need to create a new client in every method.

Notice that we are using the modern `StripeClient` approach rather than the older static `Stripe.setApiKey()` approach.

## Step 5. Create PaymentService

This is where the actual payment session is created.

Create:

`service/PaymentService.java`

```

package com.example.demo.service;

import com.stripe.StripeClient;
import com.stripe.exception.StripeException;
import com.stripe.model.checkout.Session;
import com.stripe.param.checkout.SessionCreateParams;

import org.springframework.stereotype.Service;

@Service
public class PaymentService {

    private final StripeClient stripeClient;

    public PaymentService(StripeClient stripeClient) {
        this.stripeClient = stripeClient;
    }

    public String createCheckoutSession() throws StripeException {

        // Creating Checkout Session parameters
        SessionCreateParams params =
                SessionCreateParams.builder()

                // This is a one-time payment
                .setMode(SessionCreateParams.Mode.PAYMENT)

                // Where customer goes after successful checkout
                .setSuccessUrl(
                    "http://localhost:8080/success"
                )

                // Where customer goes if checkout is cancelled
                .setCancelUrl(
                    "http://localhost:8080/cancel"
                )

                // Adding product details
                .addLineItem(
                    SessionCreateParams.LineItem.builder()

                        // Customer wants 1 product
                        .setQuantity(1L)

                        // Product price information
                        .setPriceData(
                            SessionCreateParams.LineItem.PriceData.builder()

                                // Currency is INR
                                .setCurrency("inr")

                                // ₹500 = 50000 paise
                                .setUnitAmount(50000L)

                                // Product details
                                .setProductData(
                                    SessionCreateParams.LineItem.PriceData.ProductData
                                        .builder()
                                        .setName("Laptop")
                                        .build()
                                )
                                .build()
                        )
                        .build()
                )
                .build();

        // Sending request to Stripe
        Session session = stripeClient.v1()
                .checkout()
                .sessions()
                .create(params);

        // Returning Stripe checkout URL
        return session.getUrl();
    }
}

```

Let's understand the important parts.

A. Mode

```
.setMode(SessionCreateParams.Mode.PAYMENT)
```

This tells Stripe that we are collecting a one-time payment.

For example, a customer purchases a laptop once.

B. Success URL

```
.setSuccessUrl("http://localhost:8080/success")
```

After completing checkout, Stripe redirects the customer here.

C. Cancel URL

```
.setCancelUrl("http://localhost:8080/cancel")
```

If the customer cancels checkout, Stripe redirects them here.

D. LineItem

```
.setQuantity(1L)
```

Represents the quantity of the product.

E. Currency and Amount

```
.setCurrency("inr")
.setUnitAmount(50000L)
```

Stripe generally expects amounts in the smallest currency unit.

For INR:

```
₹500 = 50000 paise
₹100 = 10000 paise
₹50  = 5000 paise
```

F. Stripe API call

```
Session session = stripeClient.v1()
        .checkout()
        .sessions()
        .create(params);
```

This is where the actual API request happens.

Your Spring Boot backend sends the checkout details to Stripe.

Stripe creates a Checkout Session and returns its information.

G. Checkout URL

```
return session.getUrl();
```

Stripe returns a URL where the customer can complete payment.

This is what we need to send to our frontend.

## Step 6. Create PaymentController

Now we need an API endpoint that our frontend can call.

Create:

`controller/PaymentController.java`

```

package com.example.demo.controller;

import com.example.demo.service.PaymentService;
import com.stripe.exception.StripeException;

import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequestMapping("/api/payments")
public class PaymentController {

    private final PaymentService paymentService;

    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @PostMapping("/checkout")
    public Map<String, String> createCheckout()
            throws StripeException {

        // Create Stripe Checkout Session
        String checkoutUrl =
                paymentService.createCheckoutSession();

        // Send URL to frontend
        return Map.of("url", checkoutUrl);
    }
}

```

Our API:

```
POST http://localhost:8080/api/payments/checkout
```

Example response:

```
{
    "url": "https://checkout.stripe.com/c/pay/cs_test_..."
}
```

The URL is generated by Stripe. It is not something we manually create.

## Step 7. Redirect customer to Stripe

Suppose you have a frontend button:

```
<button onclick="makePayment()">
    Pay Now
</button>
```

JavaScript:

```
async function makePayment() {

    const response = await fetch(
        "http://localhost:8080/api/payments/checkout",
        {
            method: "POST"
        }
    );

    const data = await response.json();

    // Redirect customer to Stripe
    window.location.href = data.url;
}
```

What happens here?

1. Customer clicks Pay Now.
2. Frontend calls our Spring Boot API.
3. Backend creates a Checkout Session.
4. Backend returns the Stripe URL.
5. Frontend redirects the customer to Stripe.
6. Customer completes payment on Stripe's website.

[docs.stripe.com](https://docs.stripe.com/payments/checkout?locale=en-GB\&utm_source=chatgpt.com)

At this point, the customer is on Stripe's payment page, not your application's payment form.

## Step 8. Create Success and Cancel endpoints

For demonstration, create two endpoints:

```
@GetMapping("/success")
public String success() {
    return "Payment process completed. Verifying payment...";
}

@GetMapping("/cancel")
public String cancel() {
    return "Payment cancelled.";
}
```

These endpoints demonstrate the redirects.

Important concept:

Don't mark an order as PAID just because the customer reaches `/success`.

A customer can visit that URL without completing a valid payment.

The actual payment confirmation should come from a verified Stripe webhook.

## Step 9. Implement Webhook

This is the most important part of our integration.

Create:

`controller/StripeWebhookController.java`

```

package com.example.demo.controller;

import com.stripe.exception.SignatureVerificationException;
import com.stripe.model.Event;
import com.stripe.model.checkout.Session;
import com.stripe.net.Webhook;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/payments")
public class StripeWebhookController {

    @Value("${stripe.webhook-secret}")
    private String webhookSecret;

    @PostMapping("/webhook")
    public ResponseEntity<String> handleWebhook(

            @RequestBody String payload,

            @RequestHeader("Stripe-Signature")
            String signature

    ) {

        Event event;

        try {

            // Verify that the webhook actually came from Stripe
            event = Webhook.constructEvent(
                    payload,
                    signature,
                    webhookSecret
            );

        } catch (SignatureVerificationException e) {

            // Reject invalid webhook signatures
            return ResponseEntity.badRequest()
                    .body("Invalid signature");
        }

        // Check which event Stripe has sent
        if ("checkout.session.completed".equals(event.getType())) {

            Session session = (Session) event
                    .getDataObjectDeserializer()
                    .getObject()
                    .orElse(null);

            if (session != null &&
                    "paid".equals(session.getPaymentStatus())) {

                // Payment is confirmed
                System.out.println("Payment successful!");

                System.out.println(
                        "Session ID: " + session.getId()
                );

                // TODO: Find order in database
                // TODO: Update order status to PAID
            }
        }

        if ("checkout.session.async_payment_succeeded"
                .equals(event.getType())) {

            // Handle delayed payment success here
            System.out.println("Delayed payment successful!");
        }

        if ("checkout.session.async_payment_failed"
                .equals(event.getType())) {

            // Handle delayed payment failure here
            System.out.println("Delayed payment failed!");
        }

        return ResponseEntity.ok("Webhook received");
    }
}

```

Let's understand what is happening.

A. Webhook endpoint

```
@PostMapping("/webhook")
```

Stripe will send an HTTP POST request to this endpoint when an event occurs.

B. Payload

```
@RequestBody String payload
```

Contains the raw event data sent by Stripe.

C. Signature

```
@RequestHeader("Stripe-Signature")
String signature
```

Stripe sends a signature header with the webhook request.

We use it to verify that the request is genuine.

D. Signature verification

```
event = Webhook.constructEvent(
        payload,
        signature,
        webhookSecret
);
```

This verifies the webhook using the signing secret.

Never trust a webhook just because someone sends a request to your endpoint.

E. Event type

```
event.getType()
```

Tells us what happened.

For example:

```
checkout.session.completed
```

means the Checkout Session completed.

F. Payment status

```
session.getPaymentStatus()
```

We check whether Stripe reports the payment as `paid`.

Only then should we fulfill the order.

Stripe recommends using webhooks for post-payment actions rather than relying on the success-page redirect.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.stripe.com\&sz=32)

Stripe Documentation+1



# 6. Complete payment architecture

Let's connect all the classes we created.

Frontend

Customer clicks Pay Now

POST /api/payments/checkout

PaymentController

Receives checkout request

PaymentService

Creates Checkout Session

Stripe API

Returns Checkout URL

Stripe Checkout

Customer completes payment

Webhook event

StripeWebhookController

Verifies payment event

Database

Updates order status to PAID

Notice that we have two different controllers:

- `PaymentController`: Your application sends a request to Stripe.
- `StripeWebhookController`: Stripe sends a request to your application.

This distinction is very important.

# 7. How to test the payment?

We have written the integration. Now let's test it.

Stripe provides a testing environment so we don't need to use real money.

## Step 1. Run Spring Boot

Start your application.

Your backend should run on:

```
http://localhost:8080
```

## Step 2. Install Stripe CLI

Stripe CLI helps us receive webhook events on our local machine.

Documentation:

[Stripe CLI installation and documentation](https://docs.stripe.com/stripe-cli?utm_source=chatgpt.com)

After installing it, open CMD and authenticate:

```
stripe login
```

Then run:

```
stripe listen --forward-to localhost:8080/api/payments/webhook
```

You will get output similar to:

```
Ready! Your webhook signing secret is:

whsec_xxxxxxxxx
```

Set this value in CMD:

```
set STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxx
```

Restart your Spring Boot application after setting the environment variables.

## Step 3. Call checkout API

Using Postman:

```
POST http://localhost:8080/api/payments/checkout
```

You should receive:

```
{
    "url": "https://checkout.stripe.com/..."
}
```

Open that URL in your browser.

## Step 4. Use test card details

Stripe provides special test card details.

Test Visa card

Test mode

# 4242 4242 4242 4242

&#x20;Copy card number

| Field  | Value                |
| ------ | -------------------- |
| Expiry | Any future date      |
| CVC    | Any 3 digits         |
| ZIP    | Any valid test value |

Complete the payment.

You should be redirected to your success URL.

At the same time, Stripe CLI will forward the webhook to your Spring Boot application.

Check your application logs.

You should see:

```
Payment successful!
Session ID: cs_test_...
```

You can also view test payments in your [Stripe Dashboard – Test Payments](https://dashboard.stripe.com/test/payments?utm_source=chatgpt.com).

# 8. Important concepts for real-world applications

Our example demonstrates the basic integration. However, there are some things you must understand before using it in an actual e-commerce application.

## 8.1 Why should we store payment details in our database?

Suppose you have an `Order` entity.

You might have:

```
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    private Double amount;

    private String status;

    private String stripeSessionId;
}
```

Possible order statuses:

```
PENDING
PAID
FAILED
CANCELLED
```

When the customer clicks Pay Now:

```
Order status = PENDING
```

After Stripe confirms successful payment:

```
Order status = PAID
```

You can store the Stripe Checkout Session ID to connect your order with its payment.

Important: Don't use `Double` for real monetary calculations. Use `BigDecimal` or integer amounts in the smallest currency unit.

## 8.2 Why should the backend calculate the amount?

Look at our example:

```
.setUnitAmount(50000L)
```

We have hardcoded ₹500.

But imagine that your frontend sends:

```
{
    "productId": 101,
    "amount": 500
}
```

A malicious customer could modify the request:

```
{
    "productId": 101,
    "amount": 1
}
```

If your backend blindly trusts the frontend amount, the customer might purchase a ₹500 product for ₹1.

Therefore:

1. Frontend sends the product ID or order ID.
2. Backend fetches the actual product price from the database.
3. Backend calculates the final amount.
4. Backend creates the Stripe Checkout Session.

Never trust the frontend for payment amounts.

## 8.3 What if Stripe sends the same webhook multiple times?

Stripe can retry webhook deliveries.

Suppose the same successful payment event reaches your application twice.

If you process it twice, you might:

- Send two confirmation emails.
- Ship the same order twice.
- Update other business records twice.

This is why we use idempotency.

Store the Stripe event ID in your database and process each event only once.

Also make your order-fulfillment operation safe to repeat.

## 8.4 What is an Idempotency Key?

Suppose your backend sends a payment creation request to Stripe.

Stripe processes it, but your application loses the response due to a network problem.

Your backend retries the request.

Without protection, you might accidentally create duplicate payment sessions.

An idempotency key allows Stripe to recognize a retry of the same operation.

Example:

```
Idempotency-Key: order-101-payment
```

For production, generate and persist an appropriate unique key per payment attempt.

The official Stripe Java SDK supports idempotency keys through request options.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 8.5 What is the difference between Success URL and Webhook?

This is a very common interview question.

| Success URL                          | Webhook                                   |
| ------------------------------------ | ----------------------------------------- |
| Stripe redirects the customer        | Stripe sends an HTTP request              |
| Used for customer navigation         | Used for backend processing               |
| Depends on customer reaching the URL | Works even if customer closes the browser |
| Not sufficient to confirm payment    | Verified event used to confirm payment    |

Remember: Success URL is for the user. Webhook is for your backend.

# 9. Important security rules

Before deploying your payment system:

1. Never expose your secret API key.
2. Never commit API keys to GitHub.
3. Never trust the amount received from frontend.
4. Always verify webhook signatures.
5. Store processed webhook event IDs to prevent duplicate processing.
6. Never mark orders PAID based only on a frontend redirect.
7. Use HTTPS in production.
8. Don't store raw card numbers or CVC in your database.

Also, configure your Spring Security CSRF rules appropriately for the webhook endpoint. The webhook should remain publicly reachable by Stripe, while its signature must be verified.

For production, you should also handle network failures, webhook retries, payment reconciliation and order concurrency.

# 10. What happens in an actual e-commerce project?

Let's connect this with the Spring Boot concepts you already know.

Imagine your application has these services:

```
API Gateway
    |
    v
Order Service
    |
    v
Payment Service
    |
    v
Stripe
```

A typical microservices payment flow:

1. Customer places an order.
2. Order Service creates an order with `PENDING` status.
3. Payment Service requests a Checkout Session from Stripe.
4. Customer completes payment.
5. Stripe sends a webhook to Payment Service.
6. Payment Service verifies the event.
7. Payment Service updates the payment status.
8. Order Service is informed that the order is paid.
9. Order is ready for fulfillment.

In a microservices application, services can communicate using Kafka, RabbitMQ, Feign, or another suitable mechanism.

For example:

```
Stripe Webhook
      |
      v
Payment Service
      |
      | PaymentSucceeded event
      v
Kafka
      |
      v
Order Service
      |
      v
Order = PAID
```

This is how payment integration connects with your existing Spring Boot microservices knowledge.

# 11. Important Stripe documentation

Stripe documentation changes over time. You might see different code in your teacher's videos because older SDK versions use different APIs.

These are useful official resources:

- [Checkout Quickstart](https://docs.stripe.com/checkout/quickstart?utm_source=chatgpt.com)
- [Create Checkout Session API](https://docs.stripe.com/api/checkout/sessions/create?utm_source=chatgpt.com)
- [Stripe Webhooks](https://docs.stripe.com/webhooks?utm_source=chatgpt.com)
- [Stripe Testing](https://docs.stripe.com/testing?utm_source=chatgpt.com)
- [Stripe Java SDK](https://github.com/stripe/stripe-java?utm_source=chatgpt.com)

One additional note: Stripe account availability, supported payment methods and INR settlement depend on your business country and account eligibility. Check the current [Stripe global availability](https://stripe.com/global?utm_source=chatgpt.com) before planning a live Indian integration.

# 12. Final revision notes

These are the important points to remember.

| Concept          | Meaning                                             |
| ---------------- | --------------------------------------------------- |
| Stripe           | Online payment processing platform                  |
| API Key          | Authenticates your application with Stripe          |
| StripeClient     | Java SDK client used to communicate with Stripe     |
| Checkout Session | Represents a customer's checkout process            |
| PaymentIntent    | Manages the payment lifecycle                       |
| PaymentMethod    | Represents the customer's payment method            |
| Webhook          | Stripe sends event notifications to your backend    |
| Idempotency      | Prevents duplicate processing of the same operation |
| Success URL      | Redirects customer after checkout                   |
| Webhook Secret   | Used to verify webhook signatures                   |

Complete flow:

```
1. Customer clicks Pay Now

2. Frontend calls Spring Boot API

3. PaymentController receives request

4. PaymentService creates Checkout Session

5. Stripe returns Checkout URL

6. Frontend redirects customer to Stripe

7. Customer completes payment

8. Stripe sends Webhook

9. Spring Boot verifies Webhook

10. PaymentService updates database

11. Order status becomes PAID
```

## Check your understanding

Stripe knowledge test

4/4 answered

1\. Which Stripe object represents the customer's checkout process?

PaymentMethod

Checkout Session

Webhook

Correct

Checkout Session represents the customer's checkout process.

2\. Why do we use a webhook?

To display the payment form

To inform our backend about payment events

To generate product prices

Correct

Stripe sends webhook events to inform our backend about payment results.

3\. Should we mark an order PAID when the customer reaches the success URL?

Yes

No, verify the payment through the webhook

Correct

The redirect is not reliable proof of payment. Verify the Stripe event and payment status.

4\. Where should the actual product price be calculated?

Frontend

Backend using database information

Customer's browser

Correct

The backend must calculate the amount using trusted product or order information.

Check answers

## Your score: 4/4

My recommendation: First understand and implement this Checkout Session flow in your Spring Boot project. Once it works, learn PaymentIntent separately, because that is where you get more control over custom payment forms and payment processing.
