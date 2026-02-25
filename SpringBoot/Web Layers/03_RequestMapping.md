# @RequestMapping –

## 1️⃣ What is @RequestMapping?

@RequestMapping maps HTTP requests to controller methods.

It matches based on:
- URL
- HTTP Method
- Query Parameters
- Headers
- Request Content Type (consumes)
- Response Type (produces)

Used at:
- Class level
- Method level  (Not good practice) 

````md id="rqm-revision-01"
------------------------------------------------------------

## 2️⃣ Basic Syntax

@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping("/{id}")
    public String getEmployee(@PathVariable Long id) {
        return "Employee " + id;
    }

    @PostMapping
    public String createEmployee() {
        return "Created";
    }
}
````

Final Endpoint:
GET /employees/1

---

## 3️⃣ Important Parameters

### ✔ value / path

Defines URL mapping.

```java
@RequestMapping("/employees")
```

---

### ✔ method

Specifies HTTP method.

```java
@RequestMapping(method=RequestMethod.POST)
This means all your methods come under this class should be postmapping , so avoid this in class level , only urls in requestmapping
best practice 
✔ Keep HTTP method at method level
✔ Keep only base URL at class level
```

Shortcut annotations:

* @GetMapping
* @PostMapping
* @PutMapping
* @DeleteMapping
* @PatchMapping

---

### ✔ params

Matches request only if query parameter exists.

```java
@RequestMapping(value="/employees", params="id")
```

Matches:
`/employees?id=10`

---

### ✔ headers

Matches request only if specific header is present.

```java
@RequestMapping(headers="X-API-VERSION=1")
```

## 4️⃣ What Are Headers?

Headers are metadata sent by client.

Example:

POST /employees
Content-Type: application/json
Accept: application/json
Authorization: Bearer token

Client sends headers (Browser / Postman / Frontend).

Common headers:

* Content-Type
* Accept
* Authorization
* Custom headers (X-API-VERSION)

---

## 5️⃣ consumes

Defines what type of request body is accepted.

```java
@RequestMapping(
    method=RequestMethod.POST,
    consumes="application/json"
)
```

Common values:

* application/json
* application/xml
* text/plain
* multipart/form-data

Wrong type → 415 Unsupported Media Type

---

## 6️⃣ produces

Defines response format.

```java
@RequestMapping(produces="application/json")
```

Common values:

* application/json
* application/xml
* text/plain
* application/pdf

Mismatch with Accept header → 406 Not Acceptable

---

## 7️⃣ How Spring Matches Request

Spring checks:

✔ URL
✔ HTTP Method
✔ Query Params
✔ Headers
✔ Content-Type (consumes)
✔ Accept header vs produces

If all match → method executes.

---

## 8️⃣ Final Summary 

* @RequestMapping maps HTTP requests.
* Can filter by URL, method, params, headers.
* consumes = request body type.
* produces = response body type.
* Headers are metadata sent by client.
* Shortcut annotations are preferred in real projects.
* Works with DispatcherServlet internally.

```

