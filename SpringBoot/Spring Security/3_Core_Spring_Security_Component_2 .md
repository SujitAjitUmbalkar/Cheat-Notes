
We are learning **internal authentication flow of** Spring Security.
<img width="1063" height="511" alt="3_Core_Spring_Security_Component_2" src="https://github.com/user-attachments/assets/915dab17-8963-496d-b225-936988708cc0" />
---

# STEP 1 — Client Sends Login Request

Look at the **top-left of the diagram**.

```
Client /login
```

This means a **user is trying to login** to the application.

Example request:

```
POST /login
```

Body:

```
username = sujeet
password = 1234
```

This request is sent from:

* Browser
* Postman
* Mobile app
* Frontend (React/Angular)

Example:

```http
POST /login
Content-Type: application/x-www-form-urlencoded

username=sujeet
password=1234
```

Now this request **enters the Spring Boot application**.

But before it reaches the controller, something important happens.

It passes through the **Security Filter Chain** of Spring Security.

```
Client
  ↓
Security Filters
```

These filters check things like:

* Authentication
* Authorization
* CSRF
* Session
* etc.

For **login requests**, a special filter handles it.

```
UsernamePasswordAuthenticationFilter
```

This filter:

1️⃣ Extracts username
2️⃣ Extracts password
3️⃣ Creates an **Authentication object**

Example object created:

```
UsernamePasswordAuthenticationToken
```

Example representation:

```
UsernamePasswordAuthenticationToken
(
  username = sujeet
  password = 1234
  authenticated = false
)
```

This object is then sent to:

```
AuthenticationManager
```

Flow till now:

```
Client
   ↓
Security Filter Chain
   ↓
UsernamePasswordAuthenticationFilter
   ↓
Create Authentication Object
   ↓
AuthenticationManager
```
---
