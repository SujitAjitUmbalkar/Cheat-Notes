### Why configure `SecurityFilterChain`?

In **Spring Security**, `SecurityFilterChain` is used to **define security rules for HTTP requests**.

Without configuration, **Spring Boot** applies **default security** (all endpoints require login with a default user).

We configure `SecurityFilterChain` when we want to:

* Allow some APIs without login (`permitAll`)
* Protect APIs with authentication (`authenticated`)
* Apply role-based access (`hasRole`)
* Choose login type (form login, basic auth, JWT)

So it **customizes how security works for requests**.

---

### How it works (short flow)

```
Client Request
      ↓
SecurityFilterChain
      ↓
Apply security rules
      ↓
Authenticate user (if required)
      ↓
Request reaches Controller
```

---

### Example

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth
            .anyRequest().authenticated()
        )
        .formLogin();

    return http.build();
}
```

Meaning:

```
All requests → require authentication
Login → use form login page
```

---

✅ **One-line summary**

`SecurityFilterChain` is configured to **customize request security rules in Spring Security** (who can access which endpoints and how authentication should happen).

--- 
