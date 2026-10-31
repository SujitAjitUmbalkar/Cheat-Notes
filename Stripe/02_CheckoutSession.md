# Stripe Checkout Session – Notes

## 1. What is Checkout Session?

A Checkout Session represents a customer's checkout process in Stripe. It contains the information required to complete a purchase, such as products, prices, quantities, currency and redirect URLs.

Stripe creates a hosted checkout page where customers enter their payment details and complete payment.

Example: A customer adds a laptop and a mouse to the cart. Your Spring Boot backend creates one Checkout Session containing both products. Stripe displays both products on its checkout page.

## 2. How Checkout Session Works

```
Customer clicks Checkout
          |
          v
Spring Boot Backend
          |
          v
Create Checkout Session
          |
          v
Stripe returns Checkout URL
          |
          v
Frontend redirects customer
          |
          v
Customer completes checkout
          |
          v
Stripe processes payment
          |
          v
Webhook notifies backend
          |
          v
Update order status
```

## 3. Checkout Session Modes

Checkout Session supports three modes:

| Mode           | Purpose                                          | Example                         |
| -------------- | ------------------------------------------------ | ------------------------------- |
| `PAYMENT`      | One-time payment                                 | Buying a laptop                 |
| `SUBSCRIPTION` | Recurring payment                                | Monthly membership              |
| `SETUP`        | Save payment method without charging immediately | Save a card for future payments |

The mode tells Stripe what kind of checkout you want to create. It determines how Stripe handles the payment and which Stripe objects are created.

### 3.1 PAYMENT Mode

```
.setMode(SessionCreateParams.Mode.PAYMENT)
```

Used when the customer pays once for a product or multiple products.

Examples:

- Buying a laptop.
- Buying clothes.
- Purchasing an online course.
- Booking a hotel.

How it works:

```
Customer selects products
          |
          v
Checkout Session (PAYMENT)
          |
          v
Customer completes payment
          |
          v
PaymentIntent
          |
          v
Payment successful
          |
          v
Order = PAID
```

Important points:

- Used for one-time purchases.
- Supports multiple products through `addLineItem()`.
- Stripe generally creates an associated PaymentIntent.
- No automatic recurring charges.
- Payment status can be checked using `session.getPaymentStatus()`.

Code:

```
SessionCreateParams params =
        SessionCreateParams.builder()

        // One-time payment
        .setMode(SessionCreateParams.Mode.PAYMENT)

        .setSuccessUrl("http://localhost:8080/success")
        .setCancelUrl("http://localhost:8080/cancel")

        .addLineItem(
            SessionCreateParams.LineItem.builder()
                .setQuantity(1L)

                .setPriceData(
                    SessionCreateParams.LineItem.PriceData.builder()
                        .setCurrency("inr")
                        .setUnitAmount(50000L) // ₹500

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
```

### 3.2 SUBSCRIPTION Mode

```
.setMode(SessionCreateParams.Mode.SUBSCRIPTION)
```

Used when the customer pays repeatedly according to a billing interval.

Examples:

- Netflix monthly subscription.
- Spotify yearly subscription.
- SaaS monthly subscription.
- Premium membership.

Suppose:

```
Premium Plan = ₹499/month
```

How it works:

```
Customer selects Premium Plan
             |
             v
Checkout Session (SUBSCRIPTION)
             |
             v
Customer completes checkout
             |
             v
Stripe creates Subscription
             |
             v
Initial payment
             |
             v
Subscription Active
             |
             v
Next billing date
             |
             v
Stripe charges again
```

Important concepts:

A. Product and Price

In Stripe, Product represents what you sell, and Price defines how much you charge and how frequently.

Example:

```
Product: Premium Plan

Price: ₹499

Billing interval: Monthly

Price ID: price_xxxxxxxxx
```

You can create Products and Prices in the [Stripe Dashboard](https://dashboard.stripe.com/test/products?utm_source=chatgpt.com).

B. Recurring Price

A subscription requires a recurring Price.

For example:

| Plan       | Amount | Interval |
| ---------- | ------ | -------- |
| Basic      | ₹199   | Monthly  |
| Premium    | ₹499   | Monthly  |
| Enterprise | ₹4,999 | Yearly   |

C. Automatic recurring billing

You don't need to write a scheduled Java task to charge the customer every month.

Stripe manages recurring billing, invoices and payment attempts.

D. Subscription object

Stripe creates a Subscription object to manage the recurring billing relationship.

You should store its ID and status in your database.

Code – Using an existing Stripe Price ID:

```
SessionCreateParams params =
        SessionCreateParams.builder()

        // Recurring payment
        .setMode(SessionCreateParams.Mode.SUBSCRIPTION)

        .setSuccessUrl(
            "http://localhost:8080/success"
        )

        .setCancelUrl(
            "http://localhost:8080/cancel"
        )

        // Existing recurring Price ID
        .addLineItem(
            SessionCreateParams.LineItem.builder()
                .setPrice("price_xxxxxxxxx")
                .setQuantity(1L)
                .build()
        )

        .build();
```

Code – Creating recurring price information dynamically:

```
SessionCreateParams.LineItem lineItem =
        SessionCreateParams.LineItem.builder()

        .setQuantity(1L)

        .setPriceData(
            SessionCreateParams.LineItem.PriceData.builder()

                .setCurrency("inr")

                .setUnitAmount(49900L) // ₹499

                .setProductData(
                    SessionCreateParams.LineItem.PriceData
                        .ProductData.builder()
                        .setName("Premium Plan")
                        .build()
                )

                // Define recurring billing
                .setRecurring(
                    SessionCreateParams.LineItem.PriceData.Recurring
                        .builder()
                        .setInterval(
                            SessionCreateParams.LineItem.PriceData
                                .Recurring.Interval.MONTH
                        )
                        .setIntervalCount(1L)
                        .build()
                )

                .build()
        )
        .build();
```

Add this line item to a Checkout Session with `SUBSCRIPTION` mode.

Recurring intervals can be:

```
DAY
WEEK
MONTH
YEAR
```

Subscription Webhook Events:

| Event                           | Meaning                   |
| ------------------------------- | ------------------------- |
| `checkout.session.completed`    | Checkout completed        |
| `customer.subscription.created` | Subscription created      |
| `customer.subscription.updated` | Subscription changed      |
| `customer.subscription.deleted` | Subscription ended        |
| `invoice.paid`                  | Invoice successfully paid |
| `invoice.payment_failed`        | Invoice payment failed    |

Remember: `checkout.session.completed` alone should not be treated as proof that every subscription invoice has been paid. Use subscription and invoice events to maintain the subscription status.

### 3.3 SETUP Mode

```
.setMode(SessionCreateParams.Mode.SETUP)
```

Used when you want to collect and save a customer's payment method without charging them immediately.

Example:

Suppose a customer registers on your application and adds a debit card.

You want to save the card today but charge the customer only when they purchase something later.

How it works:

```
Customer clicks Add Card
          |
          v
Checkout Session (SETUP)
          |
          v
Customer enters card details
          |
          v
Stripe saves payment method
          |
          v
SetupIntent succeeds
          |
          v
Payment method available
for future payments
```

Important points:

- No immediate payment is collected.
- Stripe creates an associated SetupIntent.
- Used for saving payment methods.
- Useful for future or off-session payments.
- Future charges must still follow the required customer consent and authentication rules.

Code:

```
SessionCreateParams params =
        SessionCreateParams.builder()

        // Save payment method
        .setMode(SessionCreateParams.Mode.SETUP)

        .setSuccessUrl(
            "http://localhost:8080/setup/success"
        )

        .setCancelUrl(
            "http://localhost:8080/setup/cancel"
        )

        .build();
```

After successful setup, the Checkout Session can provide the SetupIntent ID through `session.getSetupIntent()`.

You can retrieve the SetupIntent to get the saved PaymentMethod ID.

### 3.4 Comparison of All Three Modes

| Feature            | PAYMENT          | SUBSCRIPTION                                         | SETUP               |
| ------------------ | ---------------- | ---------------------------------------------------- | ------------------- |
| Purpose            | One-time payment | Recurring payment                                    | Save payment method |
| Immediate charge   | Yes, generally   | Usually, unless trial or other configuration applies | No                  |
| Main Stripe object | PaymentIntent    | Subscription                                         | SetupIntent         |
| Recurring billing  | No               | Yes                                                  | No                  |
| Example            | Buy laptop       | Monthly membership                                   | Save card           |

Which mode should you use?

- Buying products → `PAYMENT`
- Monthly/yearly membership → `SUBSCRIPTION`
- Saving card for future use → `SETUP`

## 4. Important Checkout Session Methods

These methods are used when creating a Checkout Session with Stripe's Java SDK.

| Method                          | Purpose                                              |
| ------------------------------- | ---------------------------------------------------- |
| `SessionCreateParams.builder()` | Starts building session parameters                   |
| `setMode()`                     | Sets payment, subscription or setup mode             |
| `setSuccessUrl()`               | URL to redirect to after successful checkout         |
| `setCancelUrl()`                | URL to redirect to when checkout is cancelled        |
| `addLineItem()`                 | Adds a product to the session                        |
| `setQuantity()`                 | Sets product quantity                                |
| `setPrice()`                    | Uses an existing Stripe Price ID                     |
| `setPriceData()`                | Defines price information directly                   |
| `setCurrency()`                 | Sets currency, such as `inr`                         |
| `setUnitAmount()`               | Sets amount in the currency's smallest unit          |
| `setProductData()`              | Supplies product details                             |
| `setCustomer()`                 | Associates an existing Stripe Customer               |
| `setCustomerEmail()`            | Prefills the customer's email                        |
| `setClientReferenceId()`        | Associates a reference, such as your order ID        |
| `setMetadata()`                 | Adds custom key-value information                    |
| `build()`                       | Builds the session parameters                        |
| `create(params)`                | Sends the request to Stripe to create a session      |
| `session.getId()`               | Gets the Checkout Session ID                         |
| `session.getUrl()`              | Gets the hosted Checkout URL                         |
| `session.getPaymentStatus()`    | Gets the payment status                              |
| `session.getStatus()`           | Gets the Checkout Session status                     |
| `session.getPaymentIntent()`    | Gets the associated PaymentIntent ID, when available |
| `session.getSubscription()`     | Gets the associated Subscription ID, when available  |

Note: The first group of methods configures the session before creation. The `session.get...()` methods read information from the created Session object.

## 5. Basic Checkout Session Code

```
SessionCreateParams params =
        SessionCreateParams.builder()

        // One-time payment
        .setMode(SessionCreateParams.Mode.PAYMENT)

        .setSuccessUrl(
            "http://localhost:8080/success"
            + "?session_id={CHECKOUT_SESSION_ID}"
        )

        .setCancelUrl(
            "http://localhost:8080/cancel"
        )

        // Add product
        .addLineItem(
            SessionCreateParams.LineItem.builder()
                .setQuantity(1L)

                .setPriceData(
                    SessionCreateParams.LineItem.PriceData
                        .builder()
                        .setCurrency("inr")
                        .setUnitAmount(50000L) // ₹500

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

// Create Checkout Session
Session session = stripeClient.v1()
        .checkout()
        .sessions()
        .create(params);

// Get Stripe checkout URL
String checkoutUrl = session.getUrl();
```

The frontend redirects the customer to `checkoutUrl`.

## 6. Single Product vs Multiple Products

A Checkout Session can contain multiple line items.

```
SessionCreateParams.Builder builder =
        SessionCreateParams.builder()
            .setMode(SessionCreateParams.Mode.PAYMENT)
            .setSuccessUrl("http://localhost:8080/success")
            .setCancelUrl("http://localhost:8080/cancel");

// Add first product
builder.addLineItem(createLineItem("Laptop", 5000000L, 1));

// Add second product
builder.addLineItem(createLineItem("Mouse", 100000L, 2));

// Build one session containing both products
SessionCreateParams params = builder.build();
```

Here, the laptop costs ₹50,000 and the mouse costs ₹1,000 each.

- `addLineItem()` adds each product to the same session.
- `setQuantity()` determines how many units are purchased.
- Stripe calculates the checkout total.

In a real e-commerce application, fetch the product prices from your database rather than trusting prices sent by the frontend.

## 7. `setPrice()` vs `setPriceData()`

| `setPrice()`                        | `setPriceData()`                                      |
| ----------------------------------- | ----------------------------------------------------- |
| Uses an existing Stripe Price ID    | Defines price information in the request              |
| Price is configured in Stripe       | Price and product details can be provided dynamically |
| Useful for fixed products and plans | Useful for dynamically priced checkout items          |

Example:

```
// Use existing Stripe Price
.setPrice("price_xxxxxxxxx")
```

```
// Define price dynamically
.setPriceData(
    SessionCreateParams.LineItem.PriceData.builder()
        .setCurrency("inr")
        .setUnitAmount(50000L)
        .setProductData(
            SessionCreateParams.LineItem.PriceData.ProductData
                .builder()
                .setName("Laptop")
                .build()
        )
        .build()
)
```

For subscriptions, an existing Price ID is commonly used because it already defines the recurring billing interval.

## 8. Important Checkout Session Properties

### `id`

```
session.getId();
```

Returns the Session ID, such as `cs_test_...`. Store it against your order for tracking.

### `url`

```
session.getUrl();
```

Returns the hosted Checkout URL. Send it to the frontend for redirection.

### `paymentStatus`

```
session.getPaymentStatus();
```

Common values include:

- `paid`
- `unpaid`
- `no_payment_required`

This describes the payment status, not the overall Checkout Session status.

### `status`

```
session.getStatus();
```

Common values include:

- `open`
- `complete`
- `expired`

A completed Checkout Session does not always mean the payment has succeeded. Check the payment status as well.

## 9. Checkout Session and Related Objects

| Object           | Responsibility                             |
| ---------------- | ------------------------------------------ |
| Checkout Session | Manages the checkout process               |
| PaymentIntent    | Manages a one-time payment lifecycle       |
| PaymentMethod    | Represents a customer's payment method     |
| Customer         | Represents the customer in Stripe          |
| Subscription     | Represents recurring billing               |
| Price            | Defines product price and billing interval |
| SetupIntent      | Manages saving a payment method            |
| Webhook Event    | Notifies your backend about Stripe events  |

For `PAYMENT` mode, Stripe generally creates or manages an associated PaymentIntent. For `SUBSCRIPTION` mode, Stripe creates a Subscription as part of the checkout flow. For `SETUP` mode, Stripe uses a SetupIntent.

## 10. Important Security and Implementation Rules

1. Never expose your Stripe secret key to the frontend.
2. Never trust product prices sent by the frontend.
3. Store the Checkout Session ID with your order.
4. Use webhooks to confirm payment and update order status.
5. Verify webhook signatures.
6. Handle duplicate webhook events safely.
7. Do not mark an order paid merely because the customer reaches the success URL.
8. Use `client_reference_id` or metadata to associate a session with your order, but still verify the association on the backend.
9. Use HTTPS in production.
10. Handle expired sessions and failed payments.

## 11. Final Revision

```
Checkout Session
      |
      |-- Mode
      |     |-- PAYMENT
      |     |     |-- One-time purchase
      |     |     |-- PaymentIntent
      |     |
      |     |-- SUBSCRIPTION
      |     |     |-- Recurring billing
      |     |     |-- Subscription
      |     |
      |     |-- SETUP
      |           |-- Save payment method
      |           |-- SetupIntent
      |
      |-- Line Items
      |     |-- Product
      |     |-- Price
      |     |-- Quantity
      |
      |-- Success URL
      |-- Cancel URL
      |-- Customer / Metadata
      |
      |-- Create Session
      |
      |-- Session ID
      |-- Checkout URL
      |-- Payment Status
      |-- Session Status
```

Remember: Checkout Session manages the checkout experience. Your Spring Boot backend manages products, carts and orders, while Stripe handles payment processing. Webhooks connect Stripe's payment results back to your application.
