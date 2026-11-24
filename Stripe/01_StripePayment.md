
# Stripe Payment Integration with Spring Boot

# 1. What is Stripe?

**Stripe** is an online payment processing platform that allows applications to accept payments from customers.

In simple words, Stripe acts as a bridge between:

- Your application.
- Your customer.
- Banks and payment networks.

### Example

Suppose a customer purchases a mouse worth ₹500 from your e-commerce website.

1. Customer selects the mouse.
2. Customer clicks Pay Now.
3. Your application sends payment information to Stripe.
4. Stripe displays a secure payment page.
5. Customer enters payment details.
6. Stripe processes the payment.
7. Stripe informs your backend about the payment result.

### What does Stripe handle?

- Payment processing.
- Secure payment collection.
- Bank authorization.
- Payment status.
- Payment-related events.

**Remember:** We don't need to build our own payment processing system. Stripe provides the services and APIs to handle it.

---

# 2. How Stripe Payment Works

First, understand the complete flow.

### Complete payment flow

1. Customer selects a product on the frontend.
2. Customer clicks Pay Now.
3. Frontend sends the selected product ID to the Spring Boot backend.
4. Backend fetches the product details from the database.
5. Backend creates a Checkout Session using Stripe.
6. Stripe returns a Checkout URL to the backend.
7. Backend sends that URL to the frontend.
8. Frontend redirects the customer to Stripe.
9. Customer enters payment details and completes payment.
10. Stripe processes the payment.
11. Stripe sends a webhook to Spring Boot.
12. Backend verifies the webhook and updates the order status to `PAID`.

### Flow diagram

```
Customer
   |
   v
Frontend
   |
   | Product ID
   v
PaymentController
   |
   v
PaymentService
   |
   | Fetch product details
   v
Database
   |
   | Product name and price
   v
PaymentService
   |
   | Create Checkout Session
   v
Stripe
   |
   | Checkout URL
   v
PaymentController
   |
   v
Frontend
   |
   | Redirect
   v
Stripe Payment Page
   |
   v
Customer completes payment
   |
   v
Stripe Webhook
   |
   v
StripeWebhookController
   |
   v
Verify payment
   |
   v
Database: Order = PAID
```

**Important:** There are two directions of communication:

- PaymentController → Your backend communicates with Stripe to create a payment session.
- StripeWebhookController → Stripe communicates with your backend to inform it about payment events.

---

# 3. Important Stripe Concepts

## 3.1 API Keys

Stripe provides two important API keys.

| Key                        | Usage    |
| -------------------------- | -------- |
| Publishable Key (`pk_...`) | Frontend |
| Secret Key (`sk_...`)      | Backend  |

**Publishable Key:**

- Used on the frontend.
- Identifies your Stripe account.
- Designed to be publicly accessible.

**Secret Key:**

- Used on the backend.
- Authenticates requests made to Stripe APIs.
- Must never be exposed to the frontend.
- Never commit it to GitHub.

## 3.2 Checkout Session

**Definition:** A Checkout Session represents one customer's checkout process.

It contains information such as:

- Product details.
- Product price.
- Quantity.
- Currency.
- Success URL.
- Cancel URL.

When the backend creates a Checkout Session, Stripe returns a Checkout URL.

The customer opens this URL to complete the payment on Stripe's hosted payment page.

**Remember:** Checkout Session represents the complete checkout process, not just the payment amount.

## 3.3 PaymentIntent

**Definition:** PaymentIntent represents and manages the payment lifecycle.

It helps Stripe track what is happening with a payment.

Important statuses:

| Status                    | Meaning                                |
| ------------------------- | -------------------------------------- |
| `requires_payment_method` | Payment method is required             |
| `requires_confirmation`   | Payment needs confirmation             |
| `requires_action`         | Additional customer action is required |
| `processing`              | Payment is being processed             |
| `succeeded`               | Payment successful                     |
| `canceled`                | Payment cancelled                      |

**Relationship between Checkout Session and PaymentIntent:**

When we use Checkout Sessions in `PAYMENT` mode, Stripe manages the associated PaymentIntent automatically.

We generally don't need to create the PaymentIntent separately for this basic Checkout integration.

## 3.4 PaymentMethod

**Definition:** PaymentMethod represents the payment method used by the customer.

Examples:

- Credit card.
- Debit card.
- Other supported payment methods.

Stripe securely handles sensitive payment information.

## 3.5 Webhook

**Definition:** A webhook is an HTTP request sent by Stripe to our backend when a particular event occurs.

Example:

Suppose a customer successfully pays ₹500.

Stripe sends a webhook notification to our Spring Boot application to inform it about the payment event.

Important events:

| Event                                      | Meaning                   |
| ------------------------------------------ | ------------------------- |
| `checkout.session.completed`               | Checkout completed        |
| `checkout.session.async_payment_succeeded` | Delayed payment succeeded |
| `checkout.session.async_payment_failed`    | Delayed payment failed    |
| `payment_intent.succeeded`                 | Payment successful        |
| `payment_intent.payment_failed`            | Payment failed            |

**Why do we need webhooks?**

Because our backend needs a reliable way to know whether the payment actually succeeded.

We should not mark an order as paid just because the customer reaches the success URL.

## 3.6 Idempotency

**Definition:** Idempotency means preventing the same operation from being processed multiple times.

Example:

Stripe may send the same webhook event again if it does not receive a successful response from our backend.

If we process that event repeatedly, we might accidentally:

- Update the order multiple times.
- Send duplicate confirmation emails.
- Deliver the same product multiple times.

**Solution:** Store processed Stripe event IDs in the database and avoid processing the same event again.

---

# 4. Types of Stripe Integration

| Integration       | Purpose                              |
| ----------------- | ------------------------------------ |
| Checkout Sessions | Standard checkout and e-commerce     |
| PaymentIntents    | Custom payment UI                    |
| SetupIntents      | Save payment methods for future use  |
| Stripe Billing    | Recurring payments and subscriptions |
| Stripe Connect    | Marketplace payments                 |

For normal e-commerce applications, Checkout Sessions are a good starting point.

---

# 5. Spring Boot Implementation

Now let's implement Stripe payment integration.

We will use an example of an e-commerce application.

Suppose our database contains:

| Product ID | Product Name | Price   |
| ---------- | ------------ | ------- |
| 101        | Laptop       | ₹50,000 |
| 102        | Mouse        | ₹500    |
| 103        | Keyboard     | ₹1,500  |

A customer can select any product.

Our backend will fetch the selected product from the database and create a Checkout Session for that product.

**Important:** We will not hardcode the product name or price in PaymentService.

## Step 1. Add Stripe Dependency

Add the Stripe Java SDK dependency to `pom.xml`.

```
<dependency>
    <groupId>com.stripe</groupId>
    <artifactId>stripe-java</artifactId>
    <version>34.1.0</version>
</dependency>
```

This dependency allows our Spring Boot application to communicate with Stripe using Java classes and methods.

## Step 2. Configure API Keys

Add the following properties to `application.properties`.

```
stripe.secret-key=${STRIPE_SECRET_KEY}

stripe.webhook-secret=${STRIPE_WEBHOOK_SECRET}
```

We use environment variables instead of directly writing secret keys in the properties file.

- `STRIPE_SECRET_KEY` → Used to communicate with Stripe APIs.
- `STRIPE_WEBHOOK_SECRET` → Used to verify webhook requests.

Never expose these secrets publicly.

## Step 3. Create StripeConfig

File: `config/StripeConfig.java`

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

### What is happening here?

**1. `@Configuration`**

Tells Spring Boot that this class contains configuration.

**2. `@Value("${stripe.secret-key}")`**

Reads the Stripe secret key from application properties.

**3. `@Bean`**

Tells Spring Boot to create and manage the returned object.

**4. `StripeClient`**

StripeClient is the object we use to communicate with Stripe APIs.

### Why do we use @Bean?

- Spring creates the StripeClient object.
- Spring manages its lifecycle.
- We can inject it into other classes using constructor injection.

For example, we will inject StripeClient into PaymentService.

## Step 4. Create Product Entity

File: `entity/Product.java`

Before creating a Checkout Session, we need product information.

Our product information will be stored in the database.

```
@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // Price stored in paise
    private Long price;

    // Getters and setters
}
```

Example database records:

| ID  | Name     | Price (Paise) |
| --- | -------- | ------------- |
| 101 | Laptop   | 5000000       |
| 102 | Mouse    | 50000         |
| 103 | Keyboard | 150000        |

Why store the price in paise?

Because Stripe expects the amount in the smallest currency unit for INR.

Examples:

```
₹500   = 50000 paise
₹100   = 10000 paise
₹50    = 5000 paise
₹50000 = 5000000 paise
```

## Step 5. Create ProductRepository

File: `repository/ProductRepository.java`

```
public interface ProductRepository
        extends JpaRepository<Product, Long> {
}
```

### Why do we need ProductRepository?

It helps us communicate with the product database.

For example:

```
productRepository.findById(102L);
```

This searches for the product whose ID is 102.

If the product exists, we get its details.

If it doesn't exist, we can handle that situation.

## Step 6. Create CheckoutRequest DTO

File: `dto/CheckoutRequest.java`

This is an important part of our implementation.

Remember:

The customer selects a product on the frontend.

The frontend must tell the backend which product the customer selected.

For this, we create a DTO.

```
public class CheckoutRequest {

    private Long productId;

    public Long getProductId() {
        return productId;
    }

    public void setProductId(Long productId) {
        this.productId = productId;
    }
}
```

### What is this DTO doing?

Suppose the customer selects Mouse.

The frontend sends:

```
{
    "productId": 102
}
```

Spring Boot converts this JSON into a CheckoutRequest object.

Now we can retrieve the product ID using:

```
request.getProductId();
```

It returns:

```
102
```

**Important:** The frontend sends only the product ID. It does not decide the actual product price.

Why?

Because a user can manipulate frontend requests.

For example, a user might try sending:

```
{
    "productId": 102,
    "price": 1
}
```

If the backend trusts the frontend price, someone could purchase a ₹500 mouse for ₹1.

Therefore:

**Always fetch the actual product price from the backend database. Never trust the frontend price.**

## Step 7. Create PaymentService

File: `service/PaymentService.java`

PaymentService contains the main payment creation logic.

Its responsibilities are:

1. Receive the selected product ID.
2. Fetch the product from the database.
3. Get the product name and price.
4. Create Stripe Checkout Session.
5. Return the Checkout URL.

```
@Service
public class PaymentService {

    private final StripeClient stripeClient;
    private final ProductRepository productRepository;

    public PaymentService(
            StripeClient stripeClient,
            ProductRepository productRepository) {

        this.stripeClient = stripeClient;
        this.productRepository = productRepository;
    }

    public String createCheckoutSession(Long productId)
            throws StripeException {

        // 1. Fetch selected product from database
        Product product = productRepository
                .findById(productId)
                .orElseThrow(() ->
                    new RuntimeException("Product not found")
                );

        // 2. Create Checkout Session parameters
        SessionCreateParams params =
                SessionCreateParams.builder()

                // One-time payment
                .setMode(SessionCreateParams.Mode.PAYMENT)

                // Redirect after successful checkout
                .setSuccessUrl(
                    "http://localhost:3000/success"
                )

                // Redirect after cancellation
                .setCancelUrl(
                    "http://localhost:3000/cancel"
                )

                // Product information
                .addLineItem(
                    SessionCreateParams.LineItem.builder()

                    .setQuantity(1L)

                    .setPriceData(
                        SessionCreateParams.LineItem.PriceData
                            .builder()

                        .setCurrency("inr")

                        // Dynamic price from database
                        .setUnitAmount(product.getPrice())

                        .setProductData(
                            SessionCreateParams.LineItem.PriceData
                                .ProductData.builder()

                                // Dynamic product name from database
                                .setName(product.getName())

                                .build()
                        )
                        .build()
                    )
                    .build()
                )
                .build();

        // 3. Create Checkout Session through Stripe API
        Session session = stripeClient.v1()
                .checkout()
                .sessions()
                .create(params);

        // 4. Return Stripe Checkout URL
        return session.getUrl();
    }
}
```

### Understand the important parts

**1. Constructor injection**

```
public PaymentService(
        StripeClient stripeClient,
        ProductRepository productRepository)
```

Spring Boot automatically provides both objects.

- StripeClient communicates with Stripe.
- ProductRepository communicates with our database.

**2. Fetch product**

```
Product product = productRepository
        .findById(productId)
        .orElseThrow(() ->
            new RuntimeException("Product not found")
        );
```

Suppose productId is 102.

The database returns:

```
ID: 102
Name: Mouse
Price: 50000 paise
```

**3. Set payment mode**

```
.setMode(SessionCreateParams.Mode.PAYMENT)
```

Defines that this is a one-time payment.

**4. Set success URL**

```
.setSuccessUrl("http://localhost:3000/success")
```

After successful checkout, Stripe redirects the customer to this URL.

**5. Set cancel URL**

```
.setCancelUrl("http://localhost:3000/cancel")
```

If the customer cancels checkout, Stripe redirects them to this URL.

**6. Set quantity**

```
.setQuantity(1L)
```

The customer is purchasing one unit.

Here quantity is fixed at 1 for simplicity. We can make it dynamic when implementing a shopping cart.

**7. Set currency**

```
.setCurrency("inr")
```

Specifies Indian Rupees.

**8. Set product price**

```
.setUnitAmount(product.getPrice())
```

This is dynamic.

It gets the price from the database.

**9. Set product name**

```
.setName(product.getName())
```

This also gets the product name from the database.

**10. Create Checkout Session**

```
Session session = stripeClient.v1()
        .checkout()
        .sessions()
        .create(params);
```

This sends the Checkout Session parameters to Stripe.

Stripe creates the session and returns a Session object.

**11. Get Checkout URL**

```
return session.getUrl();
```

Returns the URL where the customer can complete payment.

### Important methods revision

| Method            | Purpose                                       |
| ----------------- | --------------------------------------------- |
| `setMode()`       | Defines payment type                          |
| `setSuccessUrl()` | Successful checkout redirect                  |
| `setCancelUrl()`  | Cancelled checkout redirect                   |
| `setQuantity()`   | Sets product quantity                         |
| `setCurrency()`   | Sets payment currency                         |
| `setUnitAmount()` | Sets product amount in smallest currency unit |
| `setName()`       | Sets product name                             |
| `create()`        | Creates Checkout Session                      |
| `getUrl()`        | Returns Stripe Checkout URL                   |

## Step 8. Create PaymentController

File: `controller/PaymentController.java`

Now we need a controller to receive requests from the frontend.

```
@RestController
@RequestMapping("/api/payments")
public class PaymentController {

    private final PaymentService paymentService;

    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @PostMapping("/checkout")
    public Map<String, String> createCheckout(
            @RequestBody CheckoutRequest request)
            throws StripeException {

        String checkoutUrl =
                paymentService.createCheckoutSession(
                        request.getProductId()
                );

        // Sends Checkout URL to frontend
        return Map.of("url", checkoutUrl);
    }
}
```

### Understand this controller

**1. `@RestController`**

Tells Spring Boot that this class handles HTTP requests and returns response data.

**2. `@RequestMapping("/api/payments")`**

Defines the common URL for all endpoints inside this controller.

**3. `@PostMapping("/checkout")`**

Defines a POST endpoint.

Complete API:

```
POST /api/payments/checkout
```

**4. `@RequestBody CheckoutRequest request`**

This is very important.

It tells Spring Boot to:

- Receive JSON data from the frontend.
- Convert that JSON into a CheckoutRequest object.
- Store the converted object in the `request` variable.

**5. `request.getProductId()`**

Gets the product ID sent by the frontend.

**6. Call PaymentService**

```
paymentService.createCheckoutSession(
    request.getProductId()
);
```

Passes the selected product ID to PaymentService.

**7. Return Checkout URL**

```
return Map.of("url", checkoutUrl);
```

Sends the Checkout URL back to the frontend in JSON format.

### What exactly will the frontend send?

Suppose the customer selects Mouse.

Frontend sends this request:

```
POST /api/payments/checkout
Content-Type: application/json
```

Request body:

```
{
    "productId": 102
}
```

### What happens next?

1. `@RequestBody` receives the JSON.
2. Spring Boot converts it into a CheckoutRequest object.
3. `request.getProductId()` returns 102.
4. Controller passes 102 to PaymentService.
5. PaymentService fetches Mouse from the database.
6. PaymentService gets its actual name and price.
7. PaymentService creates a Checkout Session through Stripe.
8. Stripe returns the Checkout URL.
9. Controller sends that URL to the frontend.

### What response will the frontend receive?

```
{
    "url": "https://checkout.stripe.com/..."
}
```

The frontend can now use this URL to redirect the customer to Stripe.

## Step 9. Redirect Customer to Stripe

Now the frontend receives the Checkout URL.

Example JavaScript:

```
async function makePayment(selectedProductId) {

    const response = await fetch(
        "http://localhost:8080/api/payments/checkout",
        {
            method: "POST",

            headers: {
                "Content-Type": "application/json"
            },

            // Sending selected product ID
            body: JSON.stringify({
                productId: selectedProductId
            })
        }
    );

    const data = await response.json();

    // Redirect customer to Stripe
    window.location.href = data.url;
}
```

Suppose the user clicks Pay Now for Mouse.

The frontend calls:

```
makePayment(102);
```

The request body becomes:

```
{
    "productId": 102
}
```

The backend creates a Checkout Session for Mouse and returns its Checkout URL.

Then:

```
window.location.href = data.url;
```

Redirects the customer to Stripe's hosted payment page.

**Remember the responsibilities:**

- Frontend → Sends the selected product ID.
- Controller → Receives the request.
- Service → Fetches product and creates Checkout Session.
- Stripe → Provides the payment page.
- Frontend → Redirects the customer to Stripe.

## Step 10. Success and Cancel Endpoints

Stripe redirects the customer according to the URLs we configured.

Example:

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

These endpoints can be handled by your frontend application or backend, depending on your application design.

**Important:** Reaching the success URL does not prove that payment succeeded.

A customer might reach that URL without your backend having verified the payment.
 
## Step 11 . Always verify the payment through the webhook. - NEXT PART 


## Step 12. Implement Webhook
# LEARNING TAKEAWAY

Stripe integration mainly involves three important things:

**1. Create Checkout Session**

Backend creates a Checkout Session using product information and Stripe returns a Checkout URL.

**2. Redirect to Stripe**

Frontend redirects the customer to Stripe's hosted payment page using the Checkout URL.

**3. Handle Webhook**

Stripe informs our backend about payment events. Backend verifies the event and updates the order status.

These three concepts form the foundation of Stripe payment integration in Spring Boot.

**The most important thing to remember about dynamic checkout:**

Frontend sends the product ID → Backend fetches product details → Backend creates Stripe Checkout Session → Frontend redirects the customer to Stripe.

## **A few things to remember while practising**

1. The code is a learning implementation. A production application should also create and save a `PENDING` order before creating the Checkout Session.
2. The webhook should update that existing order, using its stored Stripe Session ID.
3. Handle webhook retries and duplicate events safely.
4. The example uses quantity `1`. A shopping cart with multiple products will require multiple line items.
5. The success and cancel URLs should point to actual frontend routes in your application.
