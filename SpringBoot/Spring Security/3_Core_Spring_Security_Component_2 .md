
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

# STEP 3 — ProviderManager & AuthenticationProvider

# 1️⃣ ProviderManager

`ProviderManager` is a **class**.

It **implements** the interface:

```
AuthenticationManager
```

So when this runs:

```java
authenticationManager.authenticate(authentication);
```

Actually Spring executes:

```java
providerManager.authenticate(authentication);
```

So **ProviderManager receives the authentication request**.

---

# 2️⃣ What ProviderManager Does

`ProviderManager` **does NOT authenticate the user itself**.

Instead it **delegates authentication to AuthenticationProviders**.

Inside the class there is a list:

```java
List<AuthenticationProvider> providers;
```

Structure:

```
ProviderManager
      │
      └── List<AuthenticationProvider>
```

Example internally:

```
ProviderManager
      │
      └── List<AuthenticationProvider>
                │
                ├── DaoAuthenticationProvider
                ├── LdapAuthenticationProvider
                ├── JwtAuthenticationProvider
                └── CustomAuthenticationProvider
```
Each of these implements AuthenticationProvider 
---

# 3️⃣ AuthenticationProvider

`AuthenticationProvider` is an **interface**.

Its responsibility:

```
Perform actual authentication
```

Important methods:

```java
Authentication authenticate(Authentication authentication)
```

Purpose:

```
Verify credentials
```

Second method:

```java
boolean supports(Class<?> authentication)
```

Purpose:

```
Check if this provider can handle this authentication type
```

---

# 4️⃣ Implementations of AuthenticationProvider

Different classes **implement this interface**.

Structure:

```
AuthenticationProvider (interface)
        ▲
        │
        ├── DaoAuthenticationProvider
        ├── LdapAuthenticationProvider
        ├── JwtAuthenticationProvider
        └── CustomAuthenticationProvider
```

Meaning:

* `DaoAuthenticationProvider` → database authentication
* `LdapAuthenticationProvider` → LDAP authentication
* `JwtAuthenticationProvider` → JWT authentication
* `CustomAuthenticationProvider` → developer-defined authentication

In **most Spring Boot applications**, the provider used is:

```
DaoAuthenticationProvider
```

---

# 5️⃣ How ProviderManager Chooses Provider

ProviderManager loops through the providers list.

Process:

```
ProviderManager
      ↓
Check provider.supports(authentication)
```

Example:

```
Authentication object =
UsernamePasswordAuthenticationToken
```

ProviderManager checks:

```
DaoAuthenticationProvider.supports()? → true
```

Then it calls:

```
DaoAuthenticationProvider.authenticate()
```

So **only the matching provider performs authentication**.

---

# 6️⃣ Complete Structure

Working flow:

```
ProviderManager
      ↓
Find provider using supports()
      ↓
Call authenticate()
```
### short Story 

---

ProviderManager has a **list of AuthenticationProvider references**, but the objects inside the list are **implementations of AuthenticationProvider** (like `DaoAuthenticationProvider`, `LdapAuthenticationProvider`, etc).

Each of these **implements the AuthenticationProvider interface**.

`AuthenticationProvider` has two important methods:

* `supports()`
* `authenticate()`

Using these methods, **ProviderManager checks each AuthenticationProvider implementation**.

First it calls:

```
supports(authentication)
```

to see **which provider can handle the authentication type**.

If it returns **true**, then ProviderManager calls:

```
authenticate(authentication)
```
and **that provider performs the actual authentication**.

---
---

# STEP 4 — DaoAuthenticationProvider

# 1️⃣ What is DaoAuthenticationProvider

`DaoAuthenticationProvider` is a **class** that **implements AuthenticationProvider**.

Relation:

```
AuthenticationProvider (interface)
        ▲
        │
DaoAuthenticationProvider (class)
```

Purpose:

```
Authenticate user using database
```

It is the **most commonly used AuthenticationProvider** in Spring Boot applications.

---

# 2️⃣ What DaoAuthenticationProvider Does

It performs **two main tasks**:

```
1. Load user from database
2. Verify password
```

But it **does not access the database directly**.

Instead it uses two components.

```
UserDetailsService
PasswordEncoder
```

Structure:

```
DaoAuthenticationProvider
        │
        ├── UserDetailsService
        └── PasswordEncoder
```

---

#  What Happens Internally

When `ProviderManager` selects this provider:

```
ProviderManager
       ↓
DaoAuthenticationProvider.authenticate()
```

Then the process begins.

### Step 1 — Receive Authentication Object

```
UsernamePasswordAuthenticationToken
```

Contains:

```
username
password
```

---

### Step 2 — Call UserDetailsService

DaoAuthenticationProvider calls:

```java
loadUserByUsername(username)
```

Component used:

```
UserDetailsService
```

Purpose:

```
Fetch user from database
```

---

### Step 3 — UserDetails Returned

`UserDetailsService` returns:

```
UserDetails
```

This object represents the **user data**.

Contains:

```
username
password
roles
authorities
```

---

### Step 4 — Password Verification

Now password is checked using:

```
PasswordEncoder
```

Example:

```
BCryptPasswordEncoder
```

Verification:

```java
passwordEncoder.matches(rawPassword, encodedPassword)
```

If password matches → authentication success.

---

# Current Flow

```
Client
 ↓
UsernamePasswordAuthenticationFilter
 ↓
AuthenticationManager
 ↓
ProviderManager
 ↓
DaoAuthenticationProvider
 ↓
UserDetailsService.loadUserByUsername()
 ↓
UserDetails returned
 ↓
PasswordEncoder.matches()
```
```
UserDetailsService
```

---

# STEP 5 — UserDetailsService
---

# 1️⃣ What is UserDetailsService

`UserDetailsService` is an **interface**.

---

# 2️⃣ Main Method

`UserDetailsService` has one important method:

```java
UserDetails loadUserByUsername(String username)
```

Meaning:

```
Take username
Fetch user from database
Return user details
```

Example call from `DaoAuthenticationProvider`:

```java
userDetailsService.loadUserByUsername(username);
```

---

# 3️⃣ Developer Implements It

Spring only provides the interface.

You create the implementation.

Example:

```
MyUserDetailsServiceImpl
```

Structure:

```
UserDetailsService (interface)
        ▲
        │
MyUserDetailsServiceImpl (your class)
```

---

# 4️⃣ What Happens Inside Your Implementation

Inside your service you usually call the repository.

Example:

```java
User user = userRepository.findByUsername(username);
```
---

# 5️⃣ What This Method Returns

`loadUserByUsername()` must return:

```
UserDetails
```

Not the raw entity.

So your code converts the user to `UserDetails`.

Now Spring receives a **UserDetails object**.

---

# Current Flow

```
Client
 ↓
UsernamePasswordAuthenticationFilter
 ↓
AuthenticationManager
 ↓
ProviderManager
 ↓
DaoAuthenticationProvider
 ↓
UserDetailsService -> MyUserDetailesServiceImpl 
 ↓
Database
```
Now the method `loadUserByUsername()` returns something important.

---

# STEP 6 — UserDetails
---

# 1️⃣ What is UserDetails

`UserDetails` is an **interface**.

Purpose:

```
Represent the authenticated user
```

It is basically an **object that holds user information** required by Spring Security.

---

# 2️⃣ What Information UserDetails Contains

`UserDetails` contains security-related data like:

Example structure:

```
UserDetails
   ├── username
   ├── password
   ├── authorities
   ├── accountNonExpired
   ├── accountNonLocked
   ├── credentialsNonExpired
   └── enabled
```

Spring Security uses this object for **authentication and authorization**.

---

# 3️⃣ Who Creates UserDetails

The `UserDetailsService` implementation creates and returns i
So flow becomes:

```
UserDetailsService
        ↓
Fetch user from database
        ↓
Create UserDetails object
        ↓
Return to DaoAuthenticationProvider
```

---

# 4️⃣ How DaoAuthenticationProvider Uses It

After receiving `UserDetails`, `DaoAuthenticationProvider` extracts:

```
storedPassword
```

Then it compares with the **entered password** using:

PasswordEncoder 
---
