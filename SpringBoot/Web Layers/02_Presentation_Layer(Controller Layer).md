# Presentation Layer (Controller Layer) – Spring Boot Notes 
Covered Notes
(Annotated Controllers , Request Mapping , Shortcut Annotations (Recommended) , Dynamic URL Handling ,  @RequestBody , Response Handling , Exception Handling , Data Validation

## 1️⃣ What is Presentation Layer?

The Presentation Layer (also called Controller Layer) is the top layer of a Spring Boot application.

It acts as the **entry point** of the application.

It handles:
- HTTP requests from client
- Sends HTTP responses back to client
- Converts request data to Java objects
- Converts Java objects to JSON/XML

Client → Controller → Service → Repository → Database


------------------------------------------------------------

## 2️⃣ Spring Boot Web Project Structure

Typical layered architecture:

controller → Handles HTTP requests  
service → Business logic  
repository → Database interaction  
entity → Database mapped classes  
dto → Data transfer objects  
database → Actual database  
client → Frontend / Postman / Browser  


------------------------------------------------------------

## 3️⃣ Annotated Controllers

Spring MVC provides annotation-based programming model.

### @Controller
Used for MVC applications (returns views like JSP/HTML).

### @RestController
Combination of:
@Controller + @ResponseBody

Meaning:
- Every method returns data directly in JSON/XML format.
- No view resolution.
- Mostly used in REST APIs.

Example:

```java
@RestController
public class EmployeeController {
}
````

---

## 4️⃣ Request Mapping
                          (we will learn this deeply in another file ) 
@RequestMapping is used to map HTTP requests to controller methods.

It can match:

* URL
* HTTP Method
* Parameters
* Headers
* Media Types

Example:

```java
@RequestMapping(value="/employees", method=RequestMethod.GET)
```

### Shortcut Annotations (Recommended)

* @GetMapping
* @PostMapping
* @PutMapping
* @DeleteMapping
* @PatchMapping

Example:

```java
@GetMapping("/employees")
```

---

## 5️⃣ Dynamic URL Handling

Here are **short, clean Markdown-friendly notes** on `@PathVariable` and `@RequestParam` with their parameters 👇

---

# 🟢 @PathVariable

## ✅ What It Does

• Used to extract values from the **URL path**.
• Binds URI template variables to method parameters.
• Commonly used in REST APIs.
• Value is part of the endpoint structure.
• Mostly used with `GET`, `PUT`, `DELETE`.

---

## ✅ Syntax Example

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.getUser(id);
}
```

URL:

```
/users/5
```

Here → `5` is mapped to `id`.

---

## ✅ Parameters of @PathVariable

```java
@PathVariable(value = "id", required = true)
```

• `value` → Name of the path variable in URL.
• `name` → Same as value (alias).
• `required` → Whether the variable is mandatory (default = true).

---

## ✅ When To Use

• When value is mandatory.
• When value represents resource identity (like ID).
• When designing RESTful APIs.

---

# 🟢 @RequestParam

## ✅ What It Does

• Used to extract values from **query parameters**.
• Binds URL query parameters to method arguments.
• Used for filtering, searching, pagination.
• Value is optional or additional data.
• Works with GET and POST.

---

## ✅ Syntax Example

```java
@GetMapping("/users")
public List<User> getUsers(@RequestParam String city) {
    return userService.getUsersByCity(city);
}
```

URL:

```
/users?city=Mumbai
```

Here → `Mumbai` is mapped to `city`.

---

## ✅ Parameters of @RequestParam

```java
@RequestParam(
    value = "city",
    required = true,
    defaultValue = "Pune"
)
```

• `value` → Name of query parameter.
• `name` → Same as value (alias).
• `required` → Whether parameter is mandatory (default = true).
• `defaultValue` → Value used if parameter is missing.

---

# 🟢 Key Differences

| Feature              | @PathVariable | @RequestParam             |
| -------------------- | ------------- | ------------------------- |
| Source               | URL path      | Query parameter           |
| Mandatory by default | Yes           | Yes                       |
| Used for             | Resource ID   | Filtering / optional data |
| Example              | /users/5      | /users?city=Mumbai        |

---

# 🟢 Quick Rule to Remember

• If value is part of URL structure → use `@PathVariable`
• If value is extra/filter/search parameter → use `@RequestParam`

---

If you want, I can also write notes on:

• @RequestBody
• @ModelAttribute
• @RequestHeader
• @RestController vs @Controller

Tell me next 👌


---

## 6️⃣ @RequestBody

Used to bind HTTP request body to Java object.

When client sends JSON:

```json
{
  "name": "Ram",
  "age": 25
}
```

Spring converts JSON → Java Object using message converters (Jackson).

Used in:

* POST
* PUT
* PATCH

Example:

```java
@PostMapping("/employees")
public Employee createEmployee(@RequestBody Employee employee) {
}
```

---

## 7️⃣ Responsibilities of Presentation Layer

The Controller Layer should:

✔ Accept HTTP requests
✔ Validate request input
✔ Convert request data to DTO
✔ Call Service layer
✔ Return proper HTTP response
✔ Handle exceptions
✔ Return correct status codes

The Controller should NOT:

❌ Contain business logic
❌ Directly access database
❌ Contain complex processing

---

## 8️⃣ What Presentation Layer Contains

* Controllers
* DTO classes
* Validation annotations
* Exception handlers
* ResponseEntity
* API documentation (Swagger/OpenAPI)
* Filters / Interceptors (sometimes)

---

## 9️⃣ Response Handling

### Returning Object Directly

```java
@GetMapping("/employees/{id}")
public Employee getEmployee(@PathVariable Long id) {
    return employeeService.getEmployee(id);
}
```

Spring automatically converts it to JSON.

### Using ResponseEntity

Used to control:

* Status code
* Headers
* Body

Example:

```java
return ResponseEntity.status(HttpStatus.CREATED).body(employee);
```

---

## 🔟 HTTP Status Codes (Important)

* 200 OK → Success
* 201 CREATED → Resource created
* 400 BAD REQUEST → Validation error
* 404 NOT FOUND → Resource not found
* 500 INTERNAL SERVER ERROR → Server error

---

## 1️⃣1️⃣ Exception Handling

Use:

### @ExceptionHandler

Handles specific exception inside controller.

### @ControllerAdvice

Global exception handling.

Example:

```java
@ControllerAdvice
public class GlobalExceptionHandler {
}
```

---

## 1️⃣2️⃣ DTO (Data Transfer Object)

Used to:

* Hide internal entity structure
* Control what data is exposed
* Avoid exposing sensitive fields

Controller → Accepts DTO
Service → Converts DTO to Entity

---

## 1️⃣3️⃣ Validation in Presentation Layer

Use annotations:

* @NotNull
* @NotBlank
* @Size
* @Email
* etc.

And add @Valid in controller:

```java
public ResponseEntity create(@Valid @RequestBody EmployeeDto dto)
```

Validation happens before service layer.

---

## 1️⃣4️⃣ Flow of Request

1. Client sends HTTP request
2. DispatcherServlet receives request
3. Finds matching Controller
4. Executes Controller method
5. Calls Service layer
6. Service calls Repository
7. Data returned back
8. Controller sends response

---

## 1️⃣5️⃣ Key Principles

* Keep controller thin
* Move business logic to service
* Use DTO instead of Entity
* Handle exceptions properly
* Use proper HTTP status codes
* Follow REST principles

---

## Final Summary

Presentation Layer is responsible for:

→ Handling HTTP communication
→ Data conversion
→ Input validation
→ Calling business logic
→ Sending structured response

It acts as a bridge between client and service layer.

It should be clean, lightweight, and focused only on request-response handling.
