## Purpose of `JwtAuthFilter`

### Why do we need this filter?

After login, the user receives a **JWT**.

For every subsequent protected API request, the client sends that JWT:

```text
Authorization: Bearer <JWT>
```

Spring Security needs to know:

> **Who is making this request?**

The `JwtAuthFilter` reads the JWT, identifies the user, and tells Spring Security that the user is authenticated.

---

### Main purpose

> **`JwtAuthFilter` authenticates a user for each request using the JWT sent in the `Authorization` header.**

It connects:

```text
JWT from request
      ↓
Identify user
      ↓
Create Authentication
      ↓
SecurityContext
      ↓
Spring Security knows current user
```

---

### Why is it needed?

Without this filter:

```text
Client
  ↓
Authorization: Bearer JWT
  ↓
Spring Security
  ↓
❓ Who is this user?
  ↓
Not authenticated
```

With the filter:

```text
Client
  ↓
Bearer JWT
  ↓
JwtAuthFilter
  ↓
Validate/read JWT
  ↓
Get user ID
  ↓
Find User
  ↓
Create Authentication
  ↓
SecurityContext
  ↓
Spring Security knows the user ✅
```

---

### What does the function do?

```java
doFilterInternal(...)
```

Its job is:

1. Get the `Authorization` header.
2. Check whether it contains `Bearer <JWT>`.
3. Extract the JWT.
4. Read/validate the JWT.
5. Extract the user ID.
6. Find that user.
7. Create an `Authentication` object.
8. Put it into `SecurityContextHolder`.
9. Continue the filter chain.

---

### Most important line

```java
SecurityContextHolder.getContext()
        .setAuthentication(authenticationToken);
```

**Purpose:**

> Tell Spring Security: **"This request belongs to this authenticated user."**

---

### Remember the difference

**Login:**

```text
Email + Password
       ↓
AuthenticationManager
       ↓
Check credentials
       ↓
Generate JWT
```

**Later API requests:**

```text
JWT
 ↓
JwtAuthFilter
 ↓
Identify user
 ↓
Set Authentication
 ↓
Access protected API
```

### One-line exam/interview answer

> **JwtAuthFilter is used to authenticate incoming requests using the JWT token and store the authenticated user's information in Spring Security's SecurityContext.**
