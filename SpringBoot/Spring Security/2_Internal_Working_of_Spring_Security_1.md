
# Step 1 — Adding Spring Security

In a Spring Boot project, we just add **one dependency**.

```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
After adding this dependency:

```text
Spring Boot automatically configures security
```

This is called **Auto Configuration**. 

Spring Boot internally uses configuration classes to set up security automatically.

You **do not need to write code initially**.

---

## What Happens After Adding It

When you start the application:

1️⃣ Spring Security becomes **active automatically**
2️⃣ All APIs become **protected**
3️⃣ Login is required to access endpoints

Example:

```
http://localhost:8080/posts
```

Instead of response you will see:

```
Login Page
```

This is the **default Spring Security login form**.

---

## Simple Way to Remember

Just remember this:

```
Add dependency
      ↓
Spring Security auto-configures
      ↓
All APIs become secured
```

---

## Small Practical Point (Developer Experience)

When beginners add this dependency, they often see:

```
401 Unauthorized
```

They think something is wrong.

But actually:

```
Spring Security is protecting your APIs
```

---

✅ **Key Point to Remember**

```
spring-boot-starter-security
→ automatically enables security
```
# Step 2 — Authentication vs Authorization

These are the **two main concepts of security**.

* **Authentication** → verifying the identity of a user
* **Authorization** → checking what the user is allowed to do 

# 1️⃣ Authentication (Who are you?)

Authentication means:

```text
Checking if the user is real or not
```

Example login:

```text
username: sujit
password: 1234
```

Spring Security checks:

```text
Is this really Sujit?
```

If correct:

```text
User is authenticated
```

After successful authentication, Spring Security creates something called:

```
SecurityContext
```

This stores:

```
user
roles
permissions
```

# 2️⃣ Authorization (What can you do?)

After login, the system decides:

```text
What actions this user is allowed to perform
```

Example roles:

```
ADMIN
USER
```

Permissions example:

| Role  | Permission   |
| ----- | ------------ |
| USER  | view posts   |
| ADMIN | delete posts |

---

# Simple Way to Remember

```text
Authentication → Who are you?
Authorization → What can you do?
```

✅ **Key Points to Remember**

```
Authentication → verify user identity
Authorization → check user permissions
```

# Step 3 — INTERNAL WORKING OF SPRING SECURITY PART 1

<img width="1961" height="945" alt="2_Internal_Working_of_Spring_Security_1" src="https://github.com/user-attachments/assets/64e7c16b-7b87-48cc-8e7b-d4cf54bf35c1" />


### Spring Boot starts

When you add dependency:

```text
spring-boot-starter-security
```

Spring Boot automatically runs:

```text
SecurityFilterAutoConfiguration
```

Its job is to **register a filter** in the servlet container.

That filter is:

```text
DelegatingFilterProxy
```
---

### 2️⃣ DelegatingFilterProxy is registered

Now every request goes through:

```text
Client Request
      ↓
DelegatingFilterProxy
```

But remember:

```text
DelegatingFilterProxy does NOT contain security logic
```

It only **delegates work to a Spring bean**.

Tomcat cannot directly call Spring beans.

So we need a bridge between Tomcat and Spring Security.

---

### 3️⃣ Spring Security creates a bean

Spring Security creates a bean named:

```text
springSecurityFilterChain
```

Important:

```text
Bean Name → springSecurityFilterChain
Class → FilterChainProxy
```
### 4️⃣ This bean contains the security filters

Inside this bean there is a **list of filters** like:

```text
CsrfFilter
UsernamePasswordAuthenticationFilter
BasicAuthenticationFilter
LogoutFilter
```

This list is called:

```text
SecurityFilterChain
```

---

### 5️⃣ DelegatingFilterProxy delegates the request

Now the flow becomes:

```text
SecurityFilterAutoConfiguration
      ↓
Registers DelegatingFilterProxy
      ↓
DelegatingFilterProxy receives request and spring security creates bean of class FilterChainProxy , name of bean springSecurityFilterChain 
      ↓
Calls bean "springSecurityFilterChain" bean of FilterChianProxy
      ↓
filterchainproxy runs security filters
      ↓
Request reaches controller
```

Spring Security creates the bean springSecurityFilterChain.
DelegatingFilterProxy only calls this bean.
That bean contains list of filters , 
and that filters are called by FilterChainproxy
---

# Step 4 — Default Behaviour of Spring Security

When we add this dependency:

```xml
spring-boot-starter-security
```

Spring Security **automatically enables some default security features**. 

You do **not write any code**, but security starts working.

---

# 1️⃣ Creates `springSecurityFilterChain`

Spring Security automatically creates a bean:

```text
springSecurityFilterChain
```

This bean is the **main security filter chain** that checks every request.

Simple meaning:

```text
All requests pass through security filters
```

---

# 2️⃣ Enables HTTP Basic Authentication

Spring Security enables **HTTP Basic Authentication** by default.

Example request:

```
Authorization: Basic username:password
```

This is commonly used for:

```text
APIs
Web services
Testing with Postman
```

---

# 3️⃣ Generates Default Login Form

If you open your application in browser:

```
http://localhost:8080
```

You will see a **default login page** automatically created by Spring Security.

You did **not create this page**.

Spring Security generated it.

---

# 4️⃣ Creates Default User

Spring Security automatically creates a user:

```text
username : user
```

Password is **generated automatically**.

You will see it in the console when the application starts.

Example:

```text
Using generated security password: 7d5a3c1e-9a24
```

So login becomes:

```text
username: user
password: (generated password)
```

---

# 5️⃣ Password Stored Using BCrypt ,  BCrypt is a password hashing algorithm used to securely store passwords.

Spring Security protects passwords using:

```text
BCrypt
```

Meaning:

```text
passwords are stored in encrypted form
```

Example:

```
$2a$10$9Qf...
```

Not plain text.

---

# 6️⃣ Logout Feature Enabled

Spring Security automatically provides:

```
/logout
```

So when user logs out:

```text
Session is cleared
User becomes unauthenticated
```

---

# 7️⃣ CSRF Protection Enabled

Spring Security automatically enables protection against:

```text
CSRF attack
```

This protects forms and state-changing requests like:

```
POST
PUT
DELETE
```

---

# Simple Way to Remember

When Spring Security starts, it automatically gives:

```text
Login page
Default user
Generated password
Password encryption
Logout support
CSRF protection
```

---

✅ **Key Idea**

```text
Add spring-boot-starter-security
→ Spring Boot automatically enables basic security
```
