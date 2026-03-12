
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

# STEP 2 — AuthenticationManager

After the login filter creates the **Authentication object**, it sends it to:

```
AuthenticationManager
```

Diagram flow:

```
UsernamePasswordAuthenticationFilter
        ↓
AuthenticationManager
```

---

## What is AuthenticationManager?

`AuthenticationManager` is an **interface** in Spring Security.

Its **main job** is:

```
Authenticate the user
```

It contains one important method:

```java
Authentication authenticate(Authentication authentication)
```

Meaning:

```
Take authentication request
Verify user
Return authenticated object
```

---

## Example Flow

The filter sends this object:

```
UsernamePasswordAuthenticationToken
(
 username = sujeet
 password = 1234
 authenticated = false
)
```

to:

```
AuthenticationManager.authenticate()
```

Example internally:

```java
authenticationManager.authenticate(authenticationToken);
```

---

## Important Concept ⚠️

`AuthenticationManager` **does NOT actually authenticate the user itself**.

Instead it **delegates the work** to another class.

That class is:

```
ProviderManager
```

So internally:

```
AuthenticationManager
      ↓ implemented by
ProviderManager
```
Flow:

```
UsernamePasswordAuthenticationFilter
        ↓
AuthenticationManager
        ↓
ProviderManager
```
Your notes are already good 👍 Sujeet. The only thing missing is the **clear placement of `AuthenticationProvider` between `ProviderManager` and `DaoAuthenticationProvider`**.

I'll correct and slightly improve your notes without changing your teaching style.

---

# STEP 3 — ProviderManager

From the previous step:

```
UsernamePasswordAuthenticationFilter
        ↓
AuthenticationManager
        ↓
ProviderManager
```

`ProviderManager` is the **actual implementation of AuthenticationManager**.

So when this runs:

```java
authenticationManager.authenticate(authentication);
```

Internally it becomes:

```java
providerManager.authenticate(authentication);
```

---

# What is ProviderManager?

`ProviderManager` is a class whose job is:

```
Manage multiple AuthenticationProviders
```

Internally it contains a **list of AuthenticationProvider objects**.

Example internally:

```java
List<AuthenticationProvider> providers;
```

So the structure looks like:

```
ProviderManager
     │
     ├── AuthenticationProvider
     ├── AuthenticationProvider
     ├── AuthenticationProvider
```

---

# What is AuthenticationProvider?

`AuthenticationProvider` is an **interface in Spring Security**.

Its job is:

```
Perform the actual authentication logic
```

It defines two important methods:

```java
Authentication authenticate(Authentication authentication);

boolean supports(Class<?> authentication);
```

Meaning:

| Method         | Purpose                                                   |
| -------------- | --------------------------------------------------------- |
| authenticate() | performs authentication                                   |
| supports()     | checks if this provider supports this authentication type |

So **AuthenticationProvider = authentication contract**.

---

# Implementations of AuthenticationProvider

Different classes implement this interface.

Example:

| Provider                     | Purpose                 |
| ---------------------------- | ----------------------- |
| DaoAuthenticationProvider    | database authentication |
| LdapAuthenticationProvider   | LDAP authentication     |
| JwtAuthenticationProvider    | JWT authentication      |
| CustomAuthenticationProvider | custom authentication   |

So the structure becomes:

```
ProviderManager
     │
     ├── DaoAuthenticationProvider
     ├── LdapAuthenticationProvider
     ├── JwtAuthenticationProvider
```

All of these **implement AuthenticationProvider**.

---

# Why Multiple AuthenticationProviders?

Because applications may support **multiple authentication types**.

Example:

| Authentication Type | Provider                     |
| ------------------- | ---------------------------- |
| Database login      | DaoAuthenticationProvider    |
| LDAP login          | LdapAuthenticationProvider   |
| JWT login           | JwtAuthenticationProvider    |
| Custom auth         | CustomAuthenticationProvider |

Spring Security selects the **correct provider automatically**.

---

# How ProviderManager Works

`ProviderManager` **loops through all providers** and checks:

```
Which provider supports this Authentication object?
```

Each provider implements:

```java
boolean supports(Class<?> authentication)
```

Example authentication object:

```
UsernamePasswordAuthenticationToken
```

So only the **matching provider performs authentication**.

---

# In Most Applications

The most commonly used provider is:

```
DaoAuthenticationProvider
```
---

# Updated Flow (Correct)

```
Client
  ↓
Security Filters
  ↓
UsernamePasswordAuthenticationFilter
  ↓
AuthenticationManager
  ↓
ProviderManager
  ↓
AuthenticationProvider
  ↓
DaoAuthenticationProvider
```
