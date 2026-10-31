# Stripe Payment Integration – Spring Boot Notes

Concise documentation · Concepts + working code structure · Revision friendly

## 1. What is Stripe?

Stripe is an online payment processing platform that allows applications to accept payments from customers.

Example: A customer purchases a product worth ₹500. Stripe processes the payment by communicating with the payment networks and banks.

Stripe handles:

- Payment processing.
- Secure payment collection.
- Bank authorization.
- Payment status.
- Payment-related events.

In simple words: Stripe acts as a bridge between your application, customers and the payment system.

## 2. How Stripe Payment Works

[Integrating Stripe Checkout a complete practical Guide](https://images.openai.com/static-rsc-4/6w9cfpSo-LhjH0h3DsHCbh4IrgtqwavGQO46hSQ7R1PKY6h5-ULumH0HWbmbE5Ps2n7MDuUiVnFBH9HhHImfJIgQtL1ZDMqgsVgk-2ogUgZ67qipwORK0azdhgysfAe1cNz0tJkRblgV87kAT2hRcC1c5-4nVgsNc6P9kVp099k?purpose=inline)

[linkedin.com](https://www.linkedin.com/pulse/integrating-stripe-checkout-complete-practical-guide-singh-srjlc?utm_source=chatgpt.com)

1. Customer clicks Pay Now.
2. Frontend calls Spring Boot backend.
3. Backend creates a Checkout Session using Stripe.
4. Stripe returns a Checkout URL.
5. Frontend redirects customer to Stripe.
6. Customer enters payment details and completes payment.
7. Stripe processes the payment.
8. Stripe sends a webhook to Spring Boot.
9. Backend verifies the webhook and updates order status to `PAID`.

```
Customer
   |
   v
Frontend
   |
   v
Spring Boot Backend
   |
   v
Stripe Checkout Session
   |
   v
Stripe Payment Page
   |
   v
Payment Processing
   |
   v
Stripe Webhook
   |
   v
Spring Boot Backend
   |
   v
Database: Order = PAID
```

## 3. Important Stripe Concepts

### 3.1 API Keys

Stripe provides two important API keys:

| Key                        | Usage    |
| -------------------------- | -------- |
| Publishable Key (`pk_...`) | Frontend |
| Secret Key (`sk_...`)      | Backend  |

- Publishable key identifies your Stripe account.
- Secret key authenticates backend API requests.
- Never expose the secret key or commit it to GitHub.

### 3.2 Checkout Session

A Checkout Session represents one customer's checkout process.

It contains:

- Product details.
- Price and quantity.
- Currency.
- Success URL.
- Cancel URL.

Stripe returns a Checkout URL where customers complete payment.

### 3.3 PaymentIntent

PaymentIntent represents and manages the payment lifecycle.

Important statuses:

| Status                    | Meaning                    |
| ------------------------- | -------------------------- |
| `requires_payment_method` | Payment method required    |
| `requires_confirmation`   | Payment needs confirmation |
| `requires_action`         | Additional action required |
| `processing`              | Payment processing         |
| `succeeded`               | Payment successful         |
| `canceled`                | Payment cancelled          |

Relationship: When using Checkout Sessions in payment mode, Stripe manages the associated PaymentIntent automatically.

### 3.4 PaymentMethod

Represents the payment method used by the customer.

Examples:

- Credit card.
- Debit card.
- Other supported payment methods.

Stripe securely handles sensitive payment information.

### 3.5 Webhook

A webhook is an HTTP request sent by Stripe to your backend when a particular event occurs.

For example, after successful payment, Stripe sends an event to your application.

Important events:

| Event                                      | Meaning                   |
| ------------------------------------------ | ------------------------- |
| `checkout.session.completed`               | Checkout completed        |
| `checkout.session.async_payment_succeeded` | Delayed payment succeeded |
| `checkout.session.async_payment_failed`    | Delayed payment failed    |
| `payment_intent.succeeded`                 | Payment successful        |
| `payment_intent.payment_failed`            | Payment failed            |

Important: Webhooks are used to confirm payment and perform backend operations.

### 3.6 Idempotency

Idempotency prevents the same operation from being processed multiple times.

For example, Stripe may send the same webhook again if it doesn't receive a successful acknowledgment.

Store processed Stripe event IDs in your database to avoid duplicate processing.

## 4. Types of Stripe Integration

| Integration       | Purpose                              |
| ----------------- | ------------------------------------ |
| Checkout Sessions | Standard checkout and e-commerce     |
| PaymentIntents    | Custom payment UI                    |
| SetupIntents      | Save payment methods for future use  |
| Stripe Billing    | Recurring payments and subscriptions |
| Stripe Connect    | Marketplace payments                 |

For normal e-commerce applications, Checkout Sessions are a good starting point.

# 5. Spring Boot Implementation

## Step 1. Add Dependency

Add Stripe Java SDK in `pom.xml`.

```
<dependency>
    <groupId>com.stripe</groupId>
    <artifactId>stripe-java</artifactId>
    <version>34.1.0</version>
</dependency>
```

## Step 2. Configure API Keys

`application.properties`

```
stripe.secret-key=${STRIPE_SECRET_KEY}

stripe.webhook-secret=${STRIPE_WEBHOOK_SECRET}
```

Use environment variables instead of hardcoding secret keys.

## Step 3. Create StripeConfig

`config/StripeConfig.java`

```
@Configuration
public class StripeConfig {

    @Value("${stripe.secret-key}")
    private String secretKey;

    @Bean
    public StripeClient stripeClient() {

        // Creates Stripe client using secret key
        return new StripeClient(secretKey);
    }
}
```

Why `@Bean`?

- Spring creates the `StripeClient` object.
- Spring manages its lifecycle.
- We can inject it wherever required.

## Step 4. Create PaymentService

`service/PaymentService.java`

```
@Service
public class PaymentService {

    private final StripeClient stripeClient;

    public PaymentService(StripeClient stripeClient) {
        this.stripeClient = stripeClient;
    }

    public String createCheckoutSession()
            throws StripeException {

        SessionCreateParams params =
                SessionCreateParams.builder()

                // One-time payment
                .setMode(SessionCreateParams.Mode.PAYMENT)

                // Redirect after successful checkout
                .setSuccessUrl(
                    "http://localhost:8080/success"
                )

                // Redirect after cancellation
                .setCancelUrl(
                    "http://localhost:8080/cancel"
                )

                // Product information
                .addLineItem(
                    SessionCreateParams.LineItem.builder()

                    .setQuantity(1L)

                    .setPriceData(
                        SessionCreateParams.LineItem.PriceData
                            .builder()

                        .setCurrency("inr")

                        // ₹500 = 50000 paise
                        .setUnitAmount(50000L)

                        .setProductData(
                            SessionCreateParams.LineItem.PriceData
                                .ProductData.builder()
                                .setName("Laptop")
                                .build()
                        )
                        .build()
                    )
                    .build()
                )
                .build();

        // Creates Checkout Session through Stripe API
        Session session = stripeClient.v1()
                .checkout()
                .sessions()
                .create(params);

        // Returns Stripe Checkout URL
        return session.getUrl();
    }
}
```

Important methods:

| Method            | Purpose                                  |
| ----------------- | ---------------------------------------- |
| `setMode()`       | Defines payment type                     |
| `setSuccessUrl()` | Successful checkout redirect             |
| `setCancelUrl()`  | Cancelled checkout redirect              |
| `setQuantity()`   | Product quantity                         |
| `setCurrency()`   | Payment currency                         |
| `setUnitAmount()` | Product amount in smallest currency unit |
| `create()`        | Creates Checkout Session                 |
| `getUrl()`        | Returns Stripe Checkout URL              |

Currency conversion:

```
₹500 = 50000 paise
₹100 = 10000 paise
₹50  = 5000 paise
```

## Step 5. Create PaymentController

`controller/PaymentController.java`

```
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

        String checkoutUrl =
                paymentService.createCheckoutSession();

        // Sends Checkout URL to frontend
        return Map.of("url", checkoutUrl);
    }
}
```

API:

```
POST /api/payments/checkout
```

Response:

```
{
    "url": "https://checkout.stripe.com/..."
}
```

## Step 6. Redirect Customer to Stripe

Frontend JavaScript:

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

The customer is now redirected to Stripe's hosted payment page.

## Step 7. Success and Cancel Endpoints

```
@GetMapping("/success")
public String success() {
    return "Payment process completed. Verifying...";
}

@GetMapping("/cancel")
public String cancel() {
    return "Payment cancelled.";
}
```

Remember: Reaching the success URL does not prove that payment succeeded. Verify payment through the webhook.

## Step 8. Implement Webhook

`controller/StripeWebhookController.java`

```
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

            // Verify Stripe webhook signature
            event = Webhook.constructEvent(
                    payload,
                    signature,
                    webhookSecret
            );

        } catch (SignatureVerificationException e) {

            return ResponseEntity.badRequest()
                    .body("Invalid signature");
        }

        // Check event type
        if ("checkout.session.completed"
                .equals(event.getType())) {

            Session session = (Session) event
                    .getDataObjectDeserializer()
                    .getObject()
                    .orElse(null);

            if (session != null &&
                    "paid".equals(session.getPaymentStatus())) {

                // Payment successfully confirmed
                System.out.println("Payment successful!");

                // Update order status in database
                // order.setStatus("PAID");
            }
        }

        return ResponseEntity.ok("Webhook received");
    }
}
```

Important webhook methods:

| Method                        | Purpose                    |
| ----------------------------- | -------------------------- |
| `constructEvent()`            | Verifies webhook signature |
| `getType()`                   | Gets event type            |
| `getDataObjectDeserializer()` | Retrieves event data       |
| `getPaymentStatus()`          | Checks payment status      |

Handle delayed payment success and failure events separately when using payment methods that support delayed confirmation.

# 6. Testing Stripe Integration

Stripe provides a test mode to test payments without real money.

Step 1. Create an account: [Stripe Dashboard](https://dashboard.stripe.com/register?utm_source=chatgpt.com).

Step 2. Get test API keys from Developers → API keys.

Step 3. Set environment variables in Windows CMD:

```
set STRIPE_SECRET_KEY=sk_test_xxxxx
```

Step 4. Install and authenticate Stripe CLI.

```
stripe login
```

Step 5. Forward webhook events to localhost:

```
stripe listen --forward-to localhost:8080/api/payments/webhook
```

Copy the generated webhook signing secret:

```
set STRIPE_WEBHOOK_SECRET=whsec_xxxxx
```

Restart Spring Boot after setting both variables.

Step 6. Call API through Postman:

```
POST http://localhost:8080/api/payments/checkout
```

Step 7. Open the returned URL and use the test card:

Stripe test card

# 4242 4242 4242 4242

&#x20;Copy card number

| Field  | Value                |
| ------ | -------------------- |
| Expiry | Any future date      |
| CVC    | Any 3 digits         |
| ZIP    | Any valid test value |

Step 8. Complete payment and check Spring Boot logs for webhook confirmation.

# 7. Database Integration (Real-world application)

In an actual e-commerce application, maintain order and payment information in the database.

Example Order Entity:

```
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    private BigDecimal amount;

    private String status;

    private String stripeSessionId;
}
```

Order status lifecycle:

```
PENDING
   |
   v
Payment Processing
   |
   v
PAID / FAILED
```

When payment succeeds, update the corresponding order to `PAID`.

Important: Store the Stripe Session ID to connect the order with its payment.

### Important production rules

1. Calculate product prices on the backend, never trust frontend amounts.
2. Verify webhook signatures.
3. Store Stripe event IDs to prevent duplicate processing.
4. Never mark orders paid based only on success URL.
5. Use `BigDecimal` or integer minor units for monetary calculations, not `Double`.
6. Never store raw card numbers or CVC.
7. Use HTTPS in production.
8. Use appropriate idempotency keys for payment creation.
9. Handle Stripe API failures and webhook retries.
10. Make order fulfillment safe against duplicate events.

# 8. Important Interview Questions

| Question                  | Answer                                       |
| ------------------------- | -------------------------------------------- |
| What is Stripe?           | Payment processing platform                  |
| What is Checkout Session? | Represents customer's checkout process       |
| What is PaymentIntent?    | Manages payment lifecycle                    |
| What is PaymentMethod?    | Represents customer's payment method         |
| What is Webhook?          | Event notification sent by Stripe to backend |
| Why use StripeClient?     | To communicate with Stripe APIs              |
| Why use a webhook?        | To reliably process payment events           |
| What is Idempotency?      | Prevents duplicate processing                |
| What is Success URL?      | Redirects customer after checkout            |
| What is Webhook Secret?   | Verifies webhook authenticity                |

## 9. Final Revision – Complete Flow

1. Customer clicks Pay Now.
2. Frontend calls PaymentController.
3. PaymentService creates Checkout Session.
4. Stripe returns Checkout URL.
5. Frontend redirects customer to Stripe.
6. Customer completes payment.
7. Stripe sends webhook event.
8. Webhook controller verifies signature.
9. Backend checks payment status.
10. Database order status becomes `PAID`.

Most important distinction to remember:

- `PaymentController` → Your application communicates with Stripe.
- `StripeWebhookController` → Stripe communicates with your application.
- `PaymentService` → Contains payment creation logic.
- `StripeConfig` → Creates and configures StripeClient.

## 10. Official Documentation

- [Stripe Checkout Quickstart](https://docs.stripe.com/checkout/quickstart?utm_source=chatgpt.com)
- [Checkout Session API](https://docs.stripe.com/api/checkout/sessions/create?utm_source=chatgpt.com)
- [Stripe Webhooks](https://docs.stripe.com/webhooks?utm_source=chatgpt.com)
- [Stripe Testing](https://docs.stripe.com/testing?utm_source=chatgpt.com)
- [Stripe Java SDK](https://github.com/stripe/stripe-java?utm_source=chatgpt.com)

LEARNING TAKEAWAY

Stripe integration mainly involves three things:

1. Create Checkout Session – Initiate payment.
2. Redirect to Stripe – Customer completes payment.
3. Handle Webhook – Confirm payment and update database.

These three concepts form the foundation of Stripe payment integration in Spring Boot.
