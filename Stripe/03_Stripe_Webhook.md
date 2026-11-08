# Stripe Webhooks – Complete Notes (Spring Boot)

<img width="1536" height="1024" alt="03_Stripe Webhooks_ From Payment to Backend_" src="https://github.com/user-attachments/assets/37a64ed4-f253-49ef-b546-89972f56cbc0" />

---

## 1. Introduction to Webhooks

A webhook is a mechanism that allows one application to automatically notify another application when a specific event occurs. In Stripe, webhooks are used to notify your Spring Boot backend about payment-related events.

### Example Flow: Course Purchase

1. **Frontend Session:** The frontend creates a Checkout Session.
2. **Customer Checkout:** The customer completes payment on Stripe's checkout page.
3. **Payment Processing:** Stripe processes the payment.
4. **Event Dispatch:** Stripe sends an event to your Spring Boot backend.
5. **Backend Update:** Your backend verifies the event and updates the order status to `PAID`.

> **Important:** Stripe does not depend on the customer returning to your success URL to notify your backend. It sends a separate, reliable HTTP POST request directly to your webhook endpoint.

---

## 2. Why Do We Need Webhooks?

Consider this situation:

* A customer pays ₹1,000, and the payment is successful.
* Before returning to your website's success URL, the customer accidentally or intentionally closes the browser.
* If your backend relies solely on the success URL, it will never receive the confirmation.

**Solution:** Webhooks provide direct server-to-server communication, ensuring your backend is notified even if the customer closes the browser.

### Without vs. With Webhooks

| Approach | Flow | Outcome |
| --- | --- | --- |
| **Without Webhook** | Customer pays on Stripe $\rightarrow$ Customer closes browser | Backend may not receive payment confirmation |
| **With Webhook** | Customer pays on Stripe $\rightarrow$ Stripe sends event to backend | Backend reliably updates order status to `PAID` |

---

## 3. Important Stripe Webhook Events

Stripe provides different events for different activities across the payment lifecycle:

| Event Name | Meaning |
| --- | --- |
| `checkout.session.completed` | Checkout Session completed |
| `payment_intent.succeeded` | Payment succeeded |
| `payment_intent.payment_failed` | Payment failed |
| `checkout.session.expired` | Checkout Session expired |
| `customer.subscription.created` | Subscription created |
| `customer.subscription.deleted` | Subscription cancelled / deleted |

* **Primary Use Case:** For our course-selling application, we will primarily work with `checkout.session.completed`.
* **Pro-Tip:** A completed Checkout Session and a successfully paid payment are not always equivalent for every payment method. For delayed payment methods, make sure to handle the appropriate asynchronous success and failure events as well.

---

## 4. Overall Working Flow

<img width="1672" height="941" alt="03_Stripe_Webhook_overallFlow" src="https://github.com/user-attachments/assets/d19a7f1a-1c49-4c8c-9211-002196e60c55" />

### Understand these steps:

1. **Selection:** Customer selects a course.
2. **Session Creation:** Spring Boot creates a Checkout Session.
3. **Checkout:** Customer completes payment.
4. **Event Generation:** Stripe generates `checkout.session.completed`.
5. **Webhook Dispatch:** Stripe sends an HTTP POST request to our webhook URL.
6. **Signature Verification:** Spring Boot verifies the signature.
7. **Fulfillment:** Backend updates the order in the database.

### Important Difference:

* **Checkout Session API:** Used to initiate the payment.
* **Webhook:** Used to receive asynchronous notifications about payment events.

---

## 5. Stripe CLI – Installation and Setup

### What is Stripe CLI?

Stripe CLI (Command Line Interface) is a tool provided by Stripe that helps developers test Stripe integrations locally.

**The Main Problem:**
Your Spring Boot application runs on your computer, usually at:
`http://localhost:8080`
Stripe's servers cannot directly access your localhost.

**The Solution:** Stripe CLI creates a secure tunnel between Stripe and your local application.

### Step 1: Install Stripe CLI (Windows)

Since you're using Windows CMD, open CMD and execute:

```bash
winget install Stripe.StripeCLI

```

Verify installation:

```bash
stripe --version

```

*(If the command isn't recognized, reopen CMD and try again.)*

### Step 2: Login to Stripe

Execute:

```bash
stripe login

```

* A browser opens.
* Authorize Stripe CLI.
* The CLI gets connected to your Stripe account.

### Step 3: Start Listening for Webhook Events

Suppose your Spring Boot webhook endpoint is:
`http://localhost:8080/api/stripe/webhook`

Execute:

```bash
stripe listen --forward-to localhost:8080/api/stripe/webhook

```

**What happens?**

* Stripe CLI starts listening for events.
* It forwards received events to your local Spring Boot endpoint.
* It displays a webhook signing secret.

You'll see something similar to:

```text
Ready! Your webhook signing secret is:
whsec_xxxxxxxxxxxxxxxxx

```

> **Important:** Copy this `whsec_` secret. We'll use it in Spring Boot to verify incoming webhook requests.


--- 
# NOTE
 To listen only to the specific webhook events your application actually handles (rather than flooding your terminal with every event Stripe generates), use the `--events` (or `-e`) flag.

### The Command

Run the Stripe CLI listen command with your required event types separated by commas:

```bash
stripe listen --forward-to localhost:8080/api/stripe/webhook --events checkout.session.completed,customer.subscription.updated,customer.subscription.deleted

```

### Common Events for Subscription/Course-Selling Apps:

* **`checkout.session.completed`**: Fires when a customer finishes the checkout process; used to grant access or record the initial order.
* **`customer.subscription.updated`**: Fires when a subscription changes (e.g., renewed, upgraded, or payment fails).
* **`customer.subscription.deleted`**: Fires when a subscription is fully canceled or expires; used to revoke user access.

* **Event Filtering:** By default, `stripe listen` forwards every single event generated in your test environment to your local server. Filtering with `--events` restricts the stream to only the specific webhooks your code implements, keeping your terminal logs clean, focused, and free of unnecessary background noise.

---

### Remember These Three Commands:

| Command | Purpose |
| --- | --- |
| `stripe --version` | Verify installation |
| `stripe login` | Connect Stripe account |
| `stripe listen` | Listen and forward events |

*Installation reference: Stripe CLI official repository.*

---

## 6. Spring Boot Implementation

Let's implement the webhook in our existing Stripe course-selling project.

### Step 1: Add Webhook Secret

In `application.properties`:

```properties
stripe.webhook.secret=${STRIPE_WEBHOOK_SECRET}

```

Set the environment variable to the `whsec_...` value obtained from the Stripe CLI.

> **Note:** This is different from your Stripe API secret key (`sk_test_...`).

| Secret Type | Purpose |
| --- | --- |
| `sk_test_...` | Communicate with Stripe API |
| `whsec_...` | Verify incoming webhook requests |

---

### Step 2: Create Webhook Controller

Create `controller/StripeWebhookController.java`:

```java
@RestController
@RequestMapping("/api/stripe")
@RequiredArgsConstructor
public class StripeWebhookController {

    private final StripeWebhookService stripeWebhookService;

    @PostMapping("/webhook")
    public ResponseEntity<String> handleWebhook(
            @RequestBody String payload,
            @RequestHeader("Stripe-Signature") String sigHeader
    ) {
        try {
            stripeWebhookService.handleEvent(
                    payload, sigHeader
            );
            return ResponseEntity.ok("Webhook received");
        } catch (Exception e) {
            return ResponseEntity.badRequest()
                    .body("Webhook verification failed");
        }
    }
}

```

**Understand the parameters:**

* `@RequestBody String payload`: Receives the complete JSON event sent by Stripe.
* `@RequestHeader("Stripe-Signature")`: Receives the signature Stripe sends with the request.
* `handleEvent()`: Passes the event to our service for verification and processing.

We receive the raw JSON body because Stripe's signature verification requires the original payload.

---

### Step 3: Create Webhook Service

Create `service/StripeWebhookService.java`:

```java
@Service
@RequiredArgsConstructor
public class StripeWebhookService {

    @Value("${stripe.webhook.secret}")
    private String endpointSecret;

    public void handleEvent(
            String payload,
            String sigHeader
    ) throws Exception {

        // 1. Verify webhook signature
        Event event = Webhook.constructEvent(
                payload,
                sigHeader,
                endpointSecret
        );

        // 2. Check event type
        if ("checkout.session.completed"
                .equals(event.getType())) {

            // 3. Extract Checkout Session
            StripeObject stripeObject = event
                    .getDataObjectDeserializer()
                    .getObject()
                    .orElse(null);

            if (stripeObject instanceof Session session) {

                // 4. Get Checkout Session ID
                String sessionId = session.getId();

                System.out.println(
                        "Payment completed: " + sessionId
                );

                // TODO: Find order and mark it PAID
            }
        }
    }
}

```

**Imports needed:**

```java
import com.stripe.model.Event;
import com.stripe.model.Session;
import com.stripe.model.StripeObject;
import com.stripe.net.Webhook;

```

You already have the Stripe Java SDK from your Checkout Session integration, so you don't need a separate webhook SDK.

---

### Step 4: What does `Webhook.constructEvent()` do?

This is the most important line:

```java
Event event = Webhook.constructEvent(
        payload,
        sigHeader,
        endpointSecret
);

```

It performs two important tasks:

1. Verifies that the request's signature is valid using the webhook secret.
2. Converts the verified JSON payload into a Stripe Event object.

If verification fails, it throws an exception.

**Why is verification necessary?**
Imagine someone sends a fake HTTP request to your webhook URL saying that a customer paid ₹10,000. Without signature verification, your backend might incorrectly mark the order as paid. Signature verification helps ensure that the request was genuinely signed by Stripe.

---

### Step 5: Extract Event Information

* `event.getType();` $\rightarrow$ Returns the event type, such as `checkout.session.completed`.
* `event.getDataObjectDeserializer().getObject().orElse(null);` $\rightarrow$ Extracts the event's actual Stripe object (in this case, a Checkout Session).
* `session.getId();` $\rightarrow$ Gets its Checkout Session ID to find the corresponding order in your database.

---

### Step 6: Update Order Status

In a real application, your next step would be:

```java
// Find order using your stored session ID
// Verify payment status
// Update order status to PAID
// Save order in database

```

> **Tip:** Do not blindly mark every received event as paid. Verify the payment state and associate the event with the correct order.

---

## 7. Testing Our Webhook

1. Keep the `stripe listen` command running in your first CMD window.
2. Open a second CMD window and execute:
```bash
stripe trigger checkout.session.completed

```



**What happens?**

1. Stripe CLI generates a test Checkout Session event.
2. It forwards the event to your Spring Boot application.
3. Your controller receives the POST request.
4. The signature gets verified.
5. Your service processes the event.

You should see the event being received in your Spring Boot console.

**Other useful testing commands:**

```bash
stripe trigger payment_intent.succeeded
stripe trigger payment_intent.payment_failed

```

*Note: These commands generate test events. They don't represent an actual purchase through your website.*

---

## 8. Important Things to Remember

1. **Never trust the frontend for payment confirmation.** A frontend success message does not prove that payment was successful.
2. **Always verify the webhook signature.** Never process an unverified webhook.
3. **Handle duplicate events.** Stripe can send the same event more than once. Store processed event IDs and avoid processing the same event twice.
4. **Handle delayed payments.** For payment methods that complete later, also handle:
* `checkout.session.async_payment_succeeded`
* `checkout.session.async_payment_failed`


5. **Return appropriate HTTP responses.** Return `2xx` when the event has been successfully handled. Return an error when processing has failed so Stripe can retry. Don't convert every internal server error into a `400`; distinguish invalid signatures from temporary processing failures.
6. **Don't expose secrets.** Never commit `sk_test_...` or `whsec_...` to GitHub.

*Reference: Stripe Webhooks Documentation and Stripe Checkout Fulfillment.*

---

## 9. Complete Revision – What Have We Learned?

| Topic | What to Remember |
| --- | --- |
| **Webhook** | Stripe sends an HTTP POST notification |
| **CLI** | Helps test webhooks locally |
| `stripe login` | Connects CLI to Stripe |
| `stripe listen` | Forwards events to localhost |
| `whsec_` | Webhook signing secret |
| **Controller** | Receives the webhook request |
| **Service** | Verifies and processes events |
| `constructEvent()` | Verifies signature and parses event |
| `checkout.session.completed` | Checkout completed event |
| **Database** | Update order after verifying payment |

---

## 10. Webhook Infographic

<img width="1279" height="720" alt="01_Stripe_Payment_Workflow" src="https://github.com/user-attachments/assets/50f07581-1023-46d7-aeab-4322c35ef579" />

---

## 11. Final Takeaway & Recommendation

> **The one concept you should never forget:**
> **Checkout Session** is used to start a payment, whereas **Webhook** is used to receive payment-related updates from Stripe.

### My Recommendation

First implement the controller and service, then run `stripe listen` and `stripe trigger`. Don't move ahead until you see a test event successfully reaching your Spring Boot application.

--- 
# Stripe Webhooks – Additional Notes (Event Types)

## 1. How does Stripe decide which events to send?

Stripe automatically generates events when something happens in your Stripe account.

Examples:

- Checkout Session completed.
- Payment succeeded.
- Payment failed.
- Subscription renewed.

However, we decide which events we want Stripe to deliver to our backend.

We configure this in:

`Stripe Dashboard → Developers → Webhooks → Add endpoint → Select events`

We can select multiple event types for one webhook endpoint.

Important: Selecting an event doesn't trigger it. Stripe generates it when the corresponding activity happens.

## 2. Does Stripe send multiple events for one payment?

Yes!

For example, when a customer purchases a course using Stripe Checkout, Stripe may generate multiple events related to that payment.

| Event                        | Meaning                    |
| ---------------------------- | -------------------------- |
| `checkout.session.completed` | Checkout process completed |
| `payment_intent.succeeded`   | Payment succeeded          |
| `charge.succeeded`           | Charge succeeded           |

These events represent different stages or objects involved in the same payment.

### Important difference

`checkout.session.completed` does not always mean payment has succeeded.

For normal card payments, it generally means payment is complete.

For delayed payment methods, the customer might complete checkout while payment is still processing.

In that case, Stripe can send:

- `checkout.session.async_payment_succeeded`
- `checkout.session.async_payment_failed`

Therefore, always check the Checkout Session's `payment_status`.

## 3. What happens when Stripe sends multiple events?

Our backend receives and processes each event independently.

Flow:

Stripe generates an event

Backend receives webhook request

Verify signature and identify event type

Execute corresponding business logic

Wait for next event and process it separately

For example:

| Event received                             | Backend action                        |
| ------------------------------------------ | ------------------------------------- |
| `checkout.session.completed`               | Check payment status and update order |
| `payment_intent.succeeded`                 | Process only if needed                |
| `checkout.session.async_payment_succeeded` | Confirm delayed payment               |
| `checkout.session.async_payment_failed`    | Handle payment failure                |

Important: We don't have to perform an action for every event. We can ignore events that are irrelevant to our application.

## 4. How do we handle multiple events safely?

There are two important practices:

A. Avoid duplicate processing

Stripe can deliver the same event more than once.

Store the Stripe event ID in your database and ensure that an already processed event isn't processed again.

B. Avoid duplicate business operations

Suppose both `checkout.session.completed` and `payment_intent.succeeded` relate to the same purchase.

Don't grant course access twice.

Design your order update logic to be idempotent (safe to execute multiple times).

## 5. Which event should we mostly use?

For your current course-selling application using Stripe Checkout:

| Situation                        | Recommended event                          |
| -------------------------------- | ------------------------------------------ |
| Normal one-time card payment     | `checkout.session.completed`               |
| Delayed payment success          | `checkout.session.async_payment_succeeded` |
| Delayed payment failure          | `checkout.session.async_payment_failed`    |
| Custom PaymentIntent integration | `payment_intent.succeeded`                 |
| Subscription renewal             | `invoice.paid`                             |

You don't need to handle every possible Stripe event.

Choose events according to the features your application supports.

## Quick revision

Remember these four concepts:

1. Stripe automatically generates events when activities happen.
2. We select which events Stripe should deliver to our webhook endpoint.
3. Our backend processes each received event according to its event type.
4. We must handle duplicate events safely to avoid repeating business operations.

One-line concept:

Stripe sends events → Backend receives them → Identifies event type → Executes corresponding logic → Waits for the next event.
