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

### Steps 
1. Create config class
2. Define SecurityFilterChain bean
3. Set authorization rules
4. Configure login method
5. build()

---

---

# Important Classes & Methods for `SecurityFilterChain`

| Class                                 | Important Methods         | Purpose                                               | Example                             |
| ------------------------------------- | ------------------------- | ----------------------------------------------------- | ----------------------------------- |
| **`SecurityFilterChain`**             | `build()`                 | Builds the final filter chain used by Spring Security | `return http.build();`              |
| **`HttpSecurity`**                    | `authorizeHttpRequests()` | Configures authorization rules for endpoints          | `http.authorizeHttpRequests()`      |
|                                       | `formLogin()`             | Enables form-based login page                         | `http.formLogin()`                  |
|                                       | `httpBasic()`             | Enables HTTP Basic authentication                     | `http.httpBasic()`                  |
|                                       | `csrf()`                  | Enables/disables CSRF protection                      | `http.csrf().disable()`             |
|                                       | `sessionManagement()`     | Configures session handling                           | `http.sessionManagement()`          |
|                                       | `logout()`                | Configures logout behavior                            | `http.logout()`                     |
|                                       | `addFilter()`             | Adds custom filter to security chain                  | `http.addFilter()`                  |
|                                       | `addFilterBefore()`       | Adds filter before another filter                     | `http.addFilterBefore()`            |
|                                       | `addFilterAfter()`        | Adds filter after another filter                      | `http.addFilterAfter()`             |
|                                       | `exceptionHandling()`     | Handles authentication/authorization errors           | `http.exceptionHandling()`          |
|                                       | `headers()`               | Configure HTTP security headers                       | `http.headers()`                    |
| **`AuthorizeHttpRequestsConfigurer`** | `requestMatchers()`       | Matches specific URLs                                 | `.requestMatchers("/public/**")`    |
|                                       | `permitAll()`             | Allows access without login                           | `.permitAll()`                      |
|                                       | `authenticated()`         | Requires authentication                               | `.authenticated()`                  |
|                                       | `hasRole()`               | Allows access to specific role                        | `.hasRole("ADMIN")`                 |
|                                       | `hasAuthority()`          | Allows based on authority                             | `.hasAuthority("READ")`             |
| **`FormLoginConfigurer`**             | `loginPage()`             | Custom login page URL                                 | `.loginPage("/login")`              |
|                                       | `defaultSuccessUrl()`     | Redirect after login success                          | `.defaultSuccessUrl("/home")`       |
|                                       | `failureUrl()`            | Redirect after login failure                          | `.failureUrl("/login?error")`       |
| **`LogoutConfigurer`**                | `logoutUrl()`             | URL used for logout                                   | `.logoutUrl("/logout")`             |
|                                       | `logoutSuccessUrl()`      | Redirect after logout                                 | `.logoutSuccessUrl("/login")`       |
| **`SessionManagementConfigurer`**     | `sessionCreationPolicy()` | Controls session creation                             | `.sessionCreationPolicy(STATELESS)` |
| **`CsrfConfigurer`**                  | `disable()`               | Disables CSRF protection                              | `.csrf(csrf -> csrf.disable())`     |

---

# Example Configuration Using These Methods

```java

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http

        // Authorization configuration
        .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/read/**").hasAuthority("READ")
                .anyRequest().authenticated()
        )

        // Form login configuration
        .formLogin(login -> login
                .loginPage("/login")
                .defaultSuccessUrl("/home")
                .failureUrl("/login?error")
        )

        // HTTP Basic authentication
        .httpBasic()

        // Logout configuration
        .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login")
        )

        // CSRF configuration
        .csrf(csrf -> csrf.disable())

        // Session management
        .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        )

        // Exception handling
        .exceptionHandling(exception ->
                exception.accessDeniedPage("/access-denied")
        )

        // Security headers
        .headers(headers ->
                headers.frameOptions(frame -> frame.sameOrigin())
        )

        // Custom filters
        .addFilter(new CustomFilter())
        .addFilterBefore(new JwtFilter(), UsernamePasswordAuthenticationFilter.class)
        .addFilterAfter(new LoggingFilter(), UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

---

# Simplified Architecture (for remembering)

```
HttpSecurity
     ↓
configure rules
     ↓
build()
     ↓
SecurityFilterChain
     ↓
Filters execute during requests
```

---

✅ **Most important methods to remember (for interviews)**
| Method                    | Short Note                                                               |
| ------------------------- | ------------------------------------------------------------------------ |
| `build()`                 | Creates the final `SecurityFilterChain` used by Spring Security filters. |
| `authorizeHttpRequests()` | Starts configuration of authorization rules for HTTP requests.           |
| `requestMatchers()`       | Matches specific URL patterns for applying security rules.               |
| `permitAll()`             | Allows public access without authentication.                             |
| `authenticated()`         | Requires user authentication for access.                                 |
| `hasRole()`               | Allows access only to users with a specific role.                        |
| `hasAuthority()`          | Allows access based on a specific permission/authority.                  |
| `formLogin()`             | Enables form-based login authentication.                                 |
| `loginPage()`             | Specifies custom login page URL.                                         |
| `defaultSuccessUrl()`     | Redirects user after successful login.                                   |
| `failureUrl()`            | Redirects user when authentication fails.                                |
| `httpBasic()`             | Enables HTTP Basic authentication using headers.                         |
| `csrf()`                  | Configures Cross-Site Request Forgery protection.                        |
| `disable()`               | Disables CSRF protection (commonly for REST APIs).                       |
| `sessionManagement()`     | Configures how sessions are managed.                                     |
| `sessionCreationPolicy()` | Defines session policy (STATELESS, ALWAYS, etc.).                        |
| `logout()`                | Enables logout configuration.                                            |
| `logoutUrl()`             | URL used to trigger logout.                                              |
| `logoutSuccessUrl()`      | Redirect URL after successful logout.                                    |
| `exceptionHandling()`     | Handles security exceptions like access denied.                          |
| `headers()`               | Configures HTTP security headers.                                        |
| `addFilter()`             | Adds a custom filter into the filter chain.                              |
| `addFilterBefore()`       | Adds a filter before a specified filter.                                 |
| `addFilterAfter()`        | Adds a filter after a specified filter.                                  |

---

### When we write implementation class of UserDetailSerice 

* Default properties spring.security.user.* are ignored and not get printed 
* Authentication uses users loaded from database.
* Password must be encoded using PasswordEncoder.
* Username entered in login must match the field used in loadUserByUsername().

If user is not found, Spring shows Invalid username or password for security.

**Why Default Password Is Not Printed (Spring Security)**

1. When Spring Security is added, Spring Boot automatically creates a **default user** (`username = user`) and prints a **random password** in the console.
2. This happens only when **no authentication-related beans** are defined in the project.
3. If a custom `UserDetailsService` bean is created (like `UserService implements UserDetailsService`), Spring Boot assumes the developer will **handle authentication manually**.
4. Because of this, Spring Boot **disables the default user and password generation**.
5. Therefore **no password is printed in the console** when the application starts.
6. Authentication will then use **database users loaded through `UserDetailsService` instead of the default in-memory user**.

