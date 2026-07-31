## 1. What is API Gateway?

* **Single Entry Point** for all client requests in a Microservices architecture.
* Receives client requests and forwards them to the correct microservice.

**Flow**

```text
Client
   |
API Gateway
   |
Route → Predicate → Filter → URI
   |
Microservice
```

---

# 2. Why API Gateway?

✅ Single Entry Point

✅ Authentication & Authorization

✅ Load Balancing

✅ Routing

✅ Logging

✅ Monitoring

✅ Rate Limiting

✅ Request/Response Modification

---

# 3. Steps to Apply Spring Cloud Gateway

### Step 1

Create a Spring Boot project.

### Step 2

Add dependencies

* Spring Cloud Gateway
* Eureka Client (optional)

### Step 3

Configure routes in `application.yml`

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/v1/orders/**
```

### Step 4

Run Eureka (if using Service Discovery).

### Step 5

Start all microservices.

### Step 6

Start API Gateway.

### Step 7

Send request

```text
Client
   ↓
Gateway
   ↓
Matching Service
```

---

# 4. Spring Cloud Gateway Building Blocks

## 1. Route

A **Route** is a complete rule that tells the Gateway:

* Which request?
* Which service?
* Which filters?

Structure

```text
Route
│
├── id
├── uri
├── predicates
└── filters
```

Example

```yaml
- id: order-service
  uri: lb://ORDER-SERVICE
  predicates:
    - Path=/api/v1/orders/**
  filters:
    - AddRequestHeader=X-User, Anuj
```

---

## Route Properties

### id

Unique name of the route.

Example

```yaml
id: order-service
```

---

### uri

Destination service.

Example

```yaml
uri: lb://ORDER-SERVICE
```

`lb://` = Load Balancer + Service Discovery.

---

# 2. Predicate

**Definition**

A **Predicate** is a condition that decides **whether a route should be selected**.

✔ Match → Execute Filters → Forward Request

❌ No Match → Check Next Route

Flow

```text
Request
   |
Predicate
   |
Match?
 /   \
Yes   No
 |      |
Filter  Next Route
 |
URI
```

### Predicate Types (with examples)

| Predicate      | Purpose                  | Example                       |
| -------------- | ------------------------ | ----------------------------- |
| **Path**       | Match URL path           | `Path=/api/orders/**`         |
| **Method**     | Match HTTP method        | `Method=GET`                  |
| **Header**     | Match request header     | `Header=Authorization`        |
| **Cookie**     | Match cookie             | `Cookie=user,admin`           |
| **Query**      | Match query parameter    | `Query=name`                  |
| **Host**       | Match hostname           | `Host=*.example.com`          |
| **After**      | Allow after a date/time  | `After=2026-08-01T10:00:00Z`  |
| **Before**     | Allow before a date/time | `Before=2026-12-31T23:59:59Z` |
| **Between**    | Allow between two times  | `Between=start,end`           |
| **RemoteAddr** | Match client IP          | `RemoteAddr=192.168.1.0/24`   |

---

# 3. Filter

Filters perform actions **before** or **after** forwarding the request.

Examples

* Add Header
* Remove Header
* Authentication
* Authorization
* Logging
* Redirect
* Rate Limiting

Example

```yaml
filters:
  - AddRequestHeader=X-User, Anuj
```

Adds

```text
X-User: Anuj
```

before forwarding.

Another Example

```yaml
filters:
  - RedirectTo=302, https://example.com
```

Returns

```text
302 Redirect
```

instead of forwarding.

---

# 5. Gateway Execution Flow

```text
Client
   |
API Gateway
   |
Check Route
   |
Predicate
   |
Match?
 /   \
Yes   No
 |      |
Execute  Check Next Route
Filters
 |
Forward to URI
 |
Microservice
 |
Response
 |
Client
```

---

# 6. Your Doubt

### Why do we see many elements if Gateway has only 3 building blocks?

Only these are **Building Blocks**:

* ✅ Route
* ✅ Predicate
* ✅ Filter

These are **Route Properties**:

* ❌ id
* ❌ uri

```text
Gateway
   |
 Route
 ├── id          (Property)
 ├── uri         (Property)
 ├── Predicate   (Building Block)
 └── Filter      (Building Block)
```

---

# 7. Quick Revision

* **API Gateway** → Single entry point.
* **Route** → Complete routing rule.
* **id** → Route name.
* **uri** → Destination microservice.
* **Predicate** → Decides if the route should be used.
* **Filter** → Performs actions before/after forwarding.
* **No Predicate Match** → Check next route.
* **Predicate Match** → Execute Filters → Forward to URI.

### Memory Trick

```text
Request
   ↓
Route
   ↓
Predicate (Should I use this Route?)
   ↓
Filter (What should I do?)
   ↓
URI (Where should I send it?)
   ↓
Microservice
```

**Mnemonic:** **RPFU** → **Route → Predicate → Filter → URI**. This is the order followed by Spring Cloud Gateway for every incoming request.
