# RestClient Notes (Spring Boot)

## 1. What is RestClient?

**Definition:**

> `RestClient` is Spring Boot's modern synchronous HTTP client used to call another REST API (another microservice or third-party API).

Example:

```text
Order Service
      │
      │ HTTP Request
      ▼
Inventory Service
```

---

# 2. Why do we need RestClient?

One Spring Boot application cannot directly call another application.

RestClient allows one backend to send HTTP requests to another backend.

Example:

```text
Order Service
      │
      ▼
GET /products
      │
      ▼
Inventory Service
```

---

# 3. Common Uses

* Microservice → Microservice communication
* Third-party API calls
* Internal REST API calls

Example:

```text
Payment Service
      │
      ▼
Bank API

Weather Service
      │
      ▼
Weather API
```

---

# 4. Requirements to use RestClient

### Mandatory

* RestClient Bean
* HTTP Method
* URI
* Response Type

### Optional

* Request Body
* Headers
* Query Parameters
* Path Variables

---

# 5. Steps to use RestClient

### Step 1

Create Bean

```java
@Bean
RestClient restClient() {
    return RestClient.create();
}
```

---

### Step 2

Inject it

```java
private final RestClient restClient;
```

---

### Step 3

Choose HTTP Method

```java
.get()
.post()
.put()
.delete()
.patch()
```

---

### Step 4

Provide URI

```java
.uri("http://localhost:8081/products")
```

or

```java
.uri(service.getUri()+"/products")
```

---

### Step 5

(Optional)

Send Body

```java
.body(productDTO)
```

---

### Step 6

(Optional)

Add Headers

```java
.header("Authorization","Bearer TOKEN")
```

---

### Step 7

Send Request

```java
.retrieve()
```

---

### Step 8

Read Response

```java
.body(ProductDTO.class)
```

or

```java
.body(String.class)
```

---

# 6. RestClient Workflow

```text
RestClient

↓

HTTP Method

↓

URI

↓

(Optional)
Headers

↓

(Optional)
Body

↓

retrieve()

↓

body(ResponseType.class)

↓

Java Object
```

---

# 7. Important Methods

### GET

```java
restClient.get()
```

Fetch data.

---

### POST

```java
restClient.post()
```

Create data.

---

### PUT

```java
restClient.put()
```

Update complete object.

---

### PATCH

```java
restClient.patch()
```

Update partial object.

---

### DELETE

```java
restClient.delete()
```

Delete data.

---

# 8. What is URI?

URI tells RestClient

> **Which API should I call?**

Example

```java
.uri("http://localhost:8081/products")
```

means

```text
Call

http://localhost:8081/products
```

---

# 9. Two ways of writing URI

## A) Without Discovery Service

```java
.uri("http://localhost:8081/products")
```

### Characteristics

* Hardcoded
* Small projects
* Local development
* Port changes require code/config update

Flow

```text
Order Service

↓

localhost:8081

↓

Inventory Service
```

---

## B) With Discovery Service (Eureka)

```java
ServiceInstance service =
discoveryClient.getInstances("inventory-service").getFirst();

.uri(service.getUri()+"/products")
```

### Characteristics

* No hardcoded port
* Uses service name
* Eureka returns current address

Flow

```text
Order Service

↓

DiscoveryClient

↓

Eureka

↓

Current Inventory Service URL

↓

Inventory Service
```

---

# 10. Difference

Without Discovery

```java
.uri("http://localhost:8081/products")
```

You already know the address.

---

With Discovery

```java
.uri(service.getUri()+"/products")
```

You only know the **service name**.

Eureka tells you the address.

---

# 11. What if Discovery Service is NOT used?

Options

### Option 1

Hardcode URL

```java
.uri("http://localhost:8081/products")
```

---

### Option 2

Store URL in application.yml

```yaml
inventory:
  url: http://localhost:8081
```

Then

```java
.uri(inventoryUrl+"/products")
```

---

### Option 3

Use Kubernetes/DNS

```text
http://inventory-service/products
```

No Eureka required.

---

# 12. Request Flow

```text
Controller

↓

Service

↓

RestClient

↓

HTTP Request

↓

Another Service

↓

HTTP Response

↓

RestClient

↓

Service

↓

Controller

↓

Client
```

---

# 13. Where should RestClient be used?

✅ Service Layer

```text
Controller

↓

Service

↓

RestClient
```

❌ Avoid placing communication logic directly inside the Controller.

---

# 14. Response Conversion

If API returns

```json
{
  "id":1,
  "name":"Laptop"
}
```

RestClient automatically converts it into

```java
ProductDTO
```

using

```java
.body(ProductDTO.class)
```

Similarly, when sending a Java object with `.body(dto)`, Spring automatically converts it to JSON.

---

# 15. Advantages

* Modern replacement for `RestTemplate`
* Clean and fluent API
* Automatic JSON ↔ Java object conversion
* Supports all HTTP methods
* Easy integration with Spring Boot

---

# 16. RestClient vs OpenFeign

| RestClient                                | OpenFeign                                                |
| ----------------------------------------- | -------------------------------------------------------- |
| Manual HTTP calls                         | Interface-based HTTP calls                               |
| You write the request code                | Spring generates the implementation                      |
| More control                              | Less boilerplate                                         |
| Good for external APIs or custom requests | Excellent for microservice-to-microservice communication |

---

# Interview Summary

**What is RestClient?**

> RestClient is Spring Boot's modern synchronous HTTP client used to communicate with REST APIs. It sends HTTP requests (GET, POST, PUT, DELETE, PATCH), receives responses, and automatically converts Java objects to JSON and JSON responses back into Java objects.

---

# Easy Memory Trick

```text
Need another API?

        │

        ▼

RestClient

        │

Choose HTTP Method

        │

Provide URI

        │

(Optional)
Headers

        │

(Optional)
Body

        │

retrieve()

        │

body(ResponseType.class)

        │

---

## RestClient Methods Cheat Sheet

| Method               | Purpose                                         | Example                                    |
| -------------------- | ----------------------------------------------- | ------------------------------------------ |
| `get()`              | Fetch data from API                             | `restClient.get()`                         |
| `post()`             | Create new resource                             | `restClient.post()`                        |
| `put()`              | Update/Replace entire resource                  | `restClient.put()`                         |
| `patch()`            | Partially update resource                       | `restClient.patch()`                       |
| `delete()`           | Delete resource                                 | `restClient.delete()`                      |
| `uri()`              | Specify API URL/Endpoint                        | `.uri("http://localhost:8081/products")`   |
| `header()`           | Add a single HTTP header                        | `.header("Authorization", "Bearer token")` |
| `headers()`          | Add multiple headers                            | `.headers(h -> h.setBearerAuth(token))`    |
| `body()`             | Send request body (POST/PUT/PATCH)              | `.body(productDTO)`                        |
| `contentType()`      | Set request Content-Type                        | `.contentType(MediaType.APPLICATION_JSON)` |
| `accept()`           | Specify expected response type                  | `.accept(MediaType.APPLICATION_JSON)`      |
| `retrieve()`         | Send request and prepare to read response       | `.retrieve()`                              |
| `body(Class<T>)`     | Convert response into an object                 | `.body(ProductDTO.class)`                  |
| `body(String.class)` | Read response as String                         | `.body(String.class)`                      |
| `toEntity()`         | Get complete response (status + headers + body) | `.toEntity(ProductDTO.class)`              |
| `toBodilessEntity()` | Get response without body                       | `.toBodilessEntity()`                      |

---

## Typical GET Request

```java
String response = restClient.get()
        .uri("http://localhost:8081/products")
        .retrieve()
        .body(String.class);
```

---

## Typical POST Request

```java
ProductDTO product = restClient.post()
        .uri("http://localhost:8081/products")
        .body(productDTO)
        .retrieve()
        .body(ProductDTO.class);
```

---

## Typical PUT Request

```java
ProductDTO updated = restClient.put()
        .uri("http://localhost:8081/products/1")
        .body(productDTO)
        .retrieve()
        .body(ProductDTO.class);
```

---

## Typical DELETE Request

```java
restClient.delete()
        .uri("http://localhost:8081/products/1")
        .retrieve()
        .toBodilessEntity();
```

---

## Order of Methods (Easy to Remember)

| Step | Method                                                | Purpose                          |
| ---- | ----------------------------------------------------- | -------------------------------- |
| 1    | `get()` / `post()` / `put()` / `patch()` / `delete()` | Choose HTTP method               |
| 2    | `uri()`                                               | Specify API endpoint             |
| 3    | `header()` / `headers()` *(Optional)*                 | Add request headers              |
| 4    | `contentType()` *(Optional)*                          | Set request content type         |
| 5    | `accept()` *(Optional)*                               | Set expected response media type |
| 6    | `body()` *(POST/PUT/PATCH only)*                      | Send request data                |
| 7    | `retrieve()`                                          | Execute the request              |
| 8    | `body()` / `toEntity()` / `toBodilessEntity()`        | Read the response                |

### Memory Formula

```text
HTTP Method
      ↓
URI
      ↓
Headers (Optional)
      ↓
Content-Type (Optional)
      ↓
Body (Optional)
      ↓
retrieve()
      ↓
Read Response
```

---


Receive Java Object

This sequence is enough to remember the complete `RestClient` workflow during coding or interviews.


| Scenario                  | RestClient URI                                 | Who finds the service? | Gateway Used? |
| ------------------------- | ---------------------------------------------- | ---------------------- | ------------- |
| No Gateway + Discovery    | `orderService.getUri()+"/api/v1/orders/hello"` | `DiscoveryClient`      | ❌             |
| Gateway + Discovery       | `http://localhost:8080/api/v1/orders/hello`    | Gateway (via Eureka)   | ✅             |
| No Gateway + No Discovery | `http://localhost:8081/api/v1/orders/hello`    | No one (hardcoded URL) | ❌             |


Interview Tip: In most microservice architectures, external clients (browser, mobile apps, Postman) call the API Gateway, while internal microservice-to-microservice communication usually bypasses the Gateway and uses service discovery + load balancing directly. This reduces latency and avoids making the Gateway a bottleneck for internal traffic.
