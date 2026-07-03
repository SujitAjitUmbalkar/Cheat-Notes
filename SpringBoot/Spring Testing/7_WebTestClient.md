## WebTestClient - Detailed Method Reference

| **Category**          | **Method**                  | **Purpose**                                                                                                                           | **Example**                                                            |
| --------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **HTTP**              | `get()`                     | Sends an HTTP GET request.                                                                                                            | `webTestClient.get()`                                                  |
|                       | `post()`                    | Sends an HTTP POST request.                                                                                                           | `webTestClient.post()`                                                 |
|                       | `put()`                     | Sends an HTTP PUT request.                                                                                                            | `webTestClient.put()`                                                  |
|                       | `delete()`                  | Sends an HTTP DELETE request.                                                                                                         | `webTestClient.delete()`                                               |
|                       | `patch()`                   | Sends an HTTP PATCH request.                                                                                                          | `webTestClient.patch()`                                                |
| **URI**               | `uri(String)`               | Sends request to a fixed URI.                                                                                                         | `.uri("/employees")`                                                   |
|                       | `uri(String, Object...)`    | Replaces path variables.                                                                                                              | `.uri("/employees/{id}", 1)`                                           |
| **Headers**           | `header(String, String...)` | Adds custom request headers.                                                                                                          | `.header("Authorization", "Bearer token")`                             |
|                       | `contentType(MediaType)`    | Sets the request's Content-Type header. Mostly used with POST, PUT, PATCH.                                                            | `.contentType(MediaType.APPLICATION_JSON)`                             |
|                       | `accept(MediaType...)`      | Specifies which response content types are acceptable.                                                                                | `.accept(MediaType.APPLICATION_JSON)`                                  |
| **Body**              | `bodyValue(Object)`         | Sends a Java object as the request body. Spring converts it to JSON using Jackson.                                                    | `.bodyValue(employeeDto)`                                              |
| **Execute**           | `exchange()`                | Executes the HTTP request and waits for the response. Every request must end with this method before assertions.                      | `.exchange()`                                                          |
| **Status**            | `expectStatus()`            | Starts HTTP status assertions.                                                                                                        | `.expectStatus()`                                                      |
|                       | `isOk()`                    | Verifies response status is **200 OK**.                                                                                               | `.isOk()`                                                              |
|                       | `isCreated()`               | Verifies response status is **201 Created**.                                                                                          | `.isCreated()`                                                         |
|                       | `isNoContent()`             | Verifies response status is **204 No Content**.                                                                                       | `.isNoContent()`                                                       |
|                       | `isNotFound()`              | Verifies response status is **404 Not Found**.                                                                                        | `.isNotFound()`                                                        |
|                       | `isBadRequest()`            | Verifies response status is **400 Bad Request**.                                                                                      | `.isBadRequest()`                                                      |
|                       | `is5xxServerError()`        | Verifies response belongs to **500–599** status codes.                                                                                | `.is5xxServerError()`                                                  |
| **Body**              | `expectBody()`              | Reads the response body as raw JSON/XML/bytes without converting it into a Java object. Usually followed by `jsonPath()` or `json()`. | `.expectBody()`                                                        |
|                       | `expectBody(Class<T>)`      | Converts the response body into the specified Java class using Jackson.                                                               | `.expectBody(EmployeeDto.class)`                                       |
|                       | `expectBodyList(Class<T>)`  | Converts a JSON array into a `List<T>`.                                                                                               | `.expectBodyList(EmployeeDto.class)`                                   |
| **JSON**              | `jsonPath(String)`          | Reads a value from the JSON response using JSONPath expression.                                                                       | `.jsonPath("$.email")`                                                 |
|                       | `json(String)`              | Compares the complete JSON response with the expected JSON string.                                                                    | `.json(expectedJson)`                                                  |
| **Object Assertions** | `isEqualTo(Object)`         | Compares the deserialized object with the expected object. Requires proper `equals()` and `hashCode()`.                               | `.isEqualTo(expectedDto)`                                              |
|                       | `value(Consumer<T>)`        | Gives access to the deserialized object for custom assertions.                                                                        | `.value(dto -> assertThat(dto.getEmail()).isEqualTo("abc@gmail.com"))` |
| **Response Result**   | `returnResult()`            | Returns the complete response object (`EntityExchangeResult`) for advanced assertions or manual processing.                           | `.returnResult()`                                                      |
|                       | `consumeWith(Consumer)`     | Gives access to the entire response (status, headers, body) for custom verification.                                                  | `.consumeWith(result -> {...})`                                        |

---

# Typical Method Chain

```java
webTestClient
        .post()                          // HTTP Method
        .uri("/employees")               // URI
        .contentType(MediaType.APPLICATION_JSON) // Request Header
        .accept(MediaType.APPLICATION_JSON)      // Accept Header
        .bodyValue(employeeDto)          // Request Body
        .exchange()                      // Execute Request
        .expectStatus().isCreated()      // Status Assertion
        .expectBody(EmployeeDto.class)   // Deserialize Response
        .isEqualTo(expectedDto);         // Compare Objects
```

---

# Alternative JSON Assertion Flow

```java
webTestClient
        .get()
        .uri("/employees/{id}", 1)
        .exchange()
        .expectStatus().isOk()
        .expectBody()
        .jsonPath("$.id").isEqualTo(1)
        .jsonPath("$.name").isEqualTo("Anuj")
        .jsonPath("$.email").isEqualTo("anuj@gmail.com");
```

---

# When to Use Which `expectBody`

| Method                  | Use When                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------- |
| `expectBody()`          | You want to verify specific JSON fields using `jsonPath()` or compare raw JSON.       |
| `expectBody(Class)`     | The endpoint returns a single object and you want it deserialized into a Java object. |
| `expectBodyList(Class)` | The endpoint returns a JSON array (list of objects).                                  |

---

# Object Assertion Methods

| Method           | Description                                                  | Requires `equals()`? |
| ---------------- | ------------------------------------------------------------ | -------------------- |
| `isEqualTo()`    | Compares the entire object with the expected object.         | ✅ Yes                |
| `value()`        | Allows custom assertions on selected fields.                 | ❌ No                 |
| `consumeWith()`  | Gives access to the entire response for advanced assertions. | ❌ No                 |
| `returnResult()` | Returns the response for later use.                          | ❌ No                 |


