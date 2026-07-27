
## 1. What is a Gateway Filter?

A **Gateway Filter** is a component that **intercepts an HTTP request or response** for a **specific route** and performs an operation **before forwarding the request (Pre Filter)** or **after receiving the response (Post Filter)**.

### Uses

* Authentication
* Logging
* Header Manipulation
* URL Rewriting
* Retry
* Circuit Breaker
* Rate Limiting
* Response Modification

---

# 2. Filter Execution Flow

```text
                  REQUEST

Client
   │
   ▼
Pre Gateway Filters
   │
   ▼
Route Matching (Predicates)
   │
   ▼
Forward to Target Service
   │
   ▼
Target Service
   │
   ▼
Post Gateway Filters
   │
   ▼
Client
```

---

# 3. Types of Gateway Filters

Spring provides **Built-in GatewayFilter Factories**.

They can be grouped into:

```text
Gateway Filters
│
├── Header Filters
├── Path Filters
├── Request Filters
├── Response Filters
├── Resilience Filters
├── Redirect Filters
└── Security Filters
```

---

# 4. Header Filters

Purpose: **Modify HTTP Headers**

| Filter                   | Purpose                                      | Example               |
| ------------------------ | -------------------------------------------- | --------------------- |
| **AddRequestHeader**     | Adds a header before forwarding the request. | Add `X-App: Gateway`. |
| **RemoveRequestHeader**  | Removes a request header.                    | Remove `Cookie`.      |
| **AddResponseHeader**    | Adds a header to the response.               | Add `X-Version: v1`.  |
| **RemoveResponseHeader** | Removes a response header.                   | Remove `Server`.      |

---

## Example

```yaml
filters:
  - AddRequestHeader=X-App, Gateway
```

---

# 5. Path Filters

Purpose: **Modify Request URL**

| Filter          | Purpose                        | Example                 |
| --------------- | ------------------------------ | ----------------------- |
| **PrefixPath**  | Adds a prefix to the URL.      | `/orders → /api/orders` |
| **StripPrefix** | Removes leading path segments. | `/api/orders → /orders` |
| **RewritePath** | Rewrites URL using regex.      | `/v1/orders → /orders`  |
| **SetPath**     | Replaces the entire path.      | `/old → /new`           |

---

## Example

```yaml
filters:
  - StripPrefix=1
```

```
Before
/api/orders/1

After
/orders/1
```

---

# 6. Request Filters

Purpose: **Modify or Validate Incoming Request**

| Filter                | Purpose                                     | Example                          |
| --------------------- | ------------------------------------------- | -------------------------------- |
| **RequestSize**       | Rejects large request bodies.               | Reject uploads larger than 5 MB. |
| **ModifyRequestBody** | Changes the request body before forwarding. | Convert XML to JSON.             |

---

## Example

```yaml
filters:
  - RequestSize=5MB
```

---

# 7. Response Filters

Purpose: **Modify Outgoing Response**

| Filter                 | Purpose                       | Example                           |
| ---------------------- | ----------------------------- | --------------------------------- |
| **ModifyResponseBody** | Changes the response body.    | Hide sensitive fields.            |
| **SetStatus**          | Returns a custom HTTP status. | Return `503 Service Unavailable`. |

---

## Example

```yaml
filters:
  - SetStatus=404
```

---

# 8. Resilience Filters

Purpose: **Improve Fault Tolerance**

| Filter                 | Purpose                                   | Example                                         |
| ---------------------- | ----------------------------------------- | ----------------------------------------------- |
| **Retry**              | Retries failed requests automatically.    | Retry service call 3 times.                     |
| **CircuitBreaker**     | Stops repeated calls to failing services. | Return fallback when Inventory Service is down. |
| **RequestRateLimiter** | Limits request rate.                      | Allow only 100 requests/minute.                 |

---

## Retry

```yaml
filters:
  - Retry=3
```

```
Attempt 1 ❌
Attempt 2 ❌
Attempt 3 ✅
```

---

## CircuitBreaker

```yaml
filters:
  - name: CircuitBreaker
    args:
      name: orderCircuitBreaker
```

```
Failure
   │
Circuit Opens
   │
Fallback Response
```

---

## RequestRateLimiter

```yaml
filters:
  - name: RequestRateLimiter
```

```
100 Requests
     │
101st Request
     │
Rejected
```

---

# 9. Redirect Filters

Purpose: **Redirect Client**

| Filter         | Purpose                              | Example                    |
| -------------- | ------------------------------------ | -------------------------- |
| **RedirectTo** | Redirects the client to another URL. | Redirect `/old` to `/new`. |

---

## Example

```yaml
filters:
  - RedirectTo=302, https://example.com
```

---

# 10. Security Filters

Purpose: **Forward Authentication**

| Filter         | Purpose                                           | Example                                 |
| -------------- | ------------------------------------------------- | --------------------------------------- |
| **TokenRelay** | Forwards OAuth2/JWT token to downstream services. | Pass JWT from Gateway to Order Service. |

---

## Example

```yaml
filters:
  - TokenRelay=
```

---

# 11. Gateway Filter Categories

| Category       | Filters                                                                        |
| -------------- | ------------------------------------------------------------------------------ |
| **Header**     | AddRequestHeader, RemoveRequestHeader, AddResponseHeader, RemoveResponseHeader |
| **Path**       | PrefixPath, StripPrefix, RewritePath, SetPath                                  |
| **Request**    | RequestSize, ModifyRequestBody                                                 |
| **Response**   | ModifyResponseBody, SetStatus                                                  |
| **Resilience** | Retry, CircuitBreaker, RequestRateLimiter                                      |
| **Redirect**   | RedirectTo                                                                     |
| **Security**   | TokenRelay                                                                     |

---

# 12. Route-Level Configuration

Gateway Filters are configured **inside a route**.

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service

          uri: lb://ORDER-SERVICE

          predicates:
            - Path=/orders/**

          filters:
            - StripPrefix=1
            - AddRequestHeader=X-App, Gateway
            - Retry=3
```

Only `/orders/**` uses these filters.

---

# 13. Applying Built-in Filters to Every Route

Instead of repeating filters for each route, use **default-filters**.

```yaml
spring:
  cloud:
    gateway:

      default-filters:
        - AddResponseHeader=X-Gateway, SpringCloudGateway
        - Retry=3
```

Now every route gets these filters.

---

# 14. GatewayFilter vs GlobalFilter

| Feature          | GatewayFilter                      | GlobalFilter                     |
| ---------------- | ---------------------------------- | -------------------------------- |
| Scope            | Specific Route                     | Every Route                      |
| Configuration    | YAML Route or Custom GatewayFilter | Java (`GlobalFilter`)            |
| Built-in Support | ✅ Yes                              | ❌ No (implement yourself)        |
| Use Case         | Retry, StripPrefix, Headers        | Authentication, Logging, Metrics |

---

# 15. Complete Filter Flow

```text
Client
   │
   ▼
Gateway
   │
   ▼
──────────────────────────────
PRE FILTERS
──────────────────────────────
✓ AddRequestHeader
✓ StripPrefix
✓ RewritePath
✓ Retry
✓ CircuitBreaker
✓ RateLimiter
✓ RequestSize
✓ TokenRelay
──────────────────────────────
          │
          ▼
Route Matching
(Predicates)
          │
          ▼
Target Service
          │
          ▼
──────────────────────────────
POST FILTERS
──────────────────────────────
✓ AddResponseHeader
✓ RemoveResponseHeader
✓ ModifyResponseBody
✓ SetStatus
──────────────────────────────
          │
          ▼
Client
```

---

# 16. Quick Revision Table (Interview)

| Filter                   | One-Line Purpose               | Example                    |
| ------------------------ | ------------------------------ | -------------------------- |
| **AddRequestHeader**     | Adds a request header.         | `X-App: Gateway`           |
| **RemoveRequestHeader**  | Removes a request header.      | Remove `Cookie`.           |
| **AddResponseHeader**    | Adds a response header.        | `X-Version: v1`.           |
| **RemoveResponseHeader** | Removes a response header.     | Remove `Server`.           |
| **PrefixPath**           | Adds a path prefix.            | `/orders → /api/orders`    |
| **StripPrefix**          | Removes leading path segments. | `/api/orders → /orders`    |
| **RewritePath**          | Rewrites URL using regex.      | `/v1/orders → /orders`     |
| **SetPath**              | Replaces the URL path.         | `/old → /new`              |
| **RequestSize**          | Limits request body size.      | Reject payload > 5 MB.     |
| **ModifyRequestBody**    | Changes request body.          | XML → JSON.                |
| **ModifyResponseBody**   | Changes response body.         | Remove password field.     |
| **SetStatus**            | Returns custom HTTP status.    | `503 Service Unavailable`. |
| **Retry**                | Retries failed requests.       | Retry 3 times.             |
| **CircuitBreaker**       | Stops repeated failures.       | Return fallback response.  |
| **RequestRateLimiter**   | Limits request rate.           | 100 requests/minute.       |
| **RedirectTo**           | Redirects the client.          | Redirect to login page.    |
| **TokenRelay**           | Forwards OAuth2/JWT token.     | Gateway → Order Service.   |

---

## ⭐ Memory Trick

```text
Gateway Filters
│
├── Header      → Headers
├── Path        → URL
├── Request     → Incoming Request
├── Response    → Outgoing Response
├── Resilience  → Retry • CircuitBreaker • RateLimiter
├── Redirect    → Redirect Client
└── Security    → TokenRelay
```

**Remember:** These are **built-in GatewayFilter factories**, so by default they apply to a **specific route**. If you configure them under **`default-filters`**, the same built-in filters apply to **all routes**. `GlobalFilter` is a separate interface used for custom logic that runs for every request.
