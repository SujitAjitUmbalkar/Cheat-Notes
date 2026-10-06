
# Granular Authorization 

## 1. What is Granular Authorization?

**Granular Authorization** means controlling access using **specific permissions/authorities** instead of checking only broad roles.

### Role-based

```text
CREATOR → can access POST APIs
```

But this is broad. What if a `CREATOR` should be allowed to create/update posts but **not delete** them?

### Granular authorization

```text
CREATOR
   ↓
POST_VIEW
POST_CREATE
POST_UPDATE
```

Now each API can check exactly what permission is required:

```java
.hasAuthority("POST_CREATE")
.hasAuthority("POST_UPDATE")
.hasAuthority("POST_DELETE")
```

### Mental model

```text
User
 ↓
Role
 ↓
PermissionMapping
 ↓
Permissions
 ↓
GrantedAuthority
 ↓
Spring Security
 ↓
hasAuthority(...)
 ↓
ALLOW / 403
```

---

# 2. Define Permissions in an Enum

First, create a central list of all permissions available in the application.

```java
public enum Permission {

    POST_VIEW,
    POST_CREATE,
    POST_UPDATE,
    POST_DELETE,

    USER_VIEW,
    USER_CREATE,
    USER_UPDATE,
    USER_DELETE
}
```

### Why enum?

Instead of writing permission strings everywhere:

```java
"POST_CREATE"
"POST_DELETE"
"USER_UPDATE"
```

we use:

```java
Permission.POST_CREATE
Permission.POST_DELETE
Permission.USER_UPDATE
```

This gives us a **central and type-safe list of permissions**.

---

# 3. Decide Which Role Gets Which Permissions

Now we need to define the relationship:

```text
Role → Permissions
```

For example:

```text
USER
 ├── USER_VIEW
 └── POST_VIEW

CREATOR
 ├── USER_VIEW
 ├── POST_VIEW
 ├── POST_CREATE
 └── POST_UPDATE

ADMIN
 ├── USER_VIEW
 ├── USER_CREATE
 ├── USER_UPDATE
 ├── USER_DELETE
 ├── POST_VIEW
 ├── POST_CREATE
 ├── POST_UPDATE
 └── POST_DELETE
```

We keep this mapping in `PermissionMapping`.

```java
public class PermissionMapping {

    private static final Map<Role, Set<Permission>> map = Map.of(

        Role.USER, Set.of(
            Permission.USER_VIEW,
            Permission.POST_VIEW
        ),

        Role.CREATOR, Set.of(
            Permission.USER_VIEW,
            Permission.POST_VIEW,
            Permission.POST_CREATE,
            Permission.POST_UPDATE
        ),

        Role.ADMIN, Set.of(
            Permission.USER_VIEW,
            Permission.USER_CREATE,
            Permission.USER_UPDATE,
            Permission.USER_DELETE,

            Permission.POST_VIEW,
            Permission.POST_CREATE,
            Permission.POST_UPDATE,
            Permission.POST_DELETE
        )
    );
}
```

### Important

This is the **business authorization configuration**.

It answers:

> "If a user has this role, what permissions should that user receive?"

---

# 4. Convert Permissions into Spring Security Authorities

Spring Security does not directly work with our `Permission` enum.

It works with:

```java
GrantedAuthority
```

So we create a function that converts:

```text
Permission
        ↓
SimpleGrantedAuthority
```

The function accepts a `Role`.

```java
public static Set<SimpleGrantedAuthority> authorities(Role role) {

    return map.get(role)
            .stream()
            .map(permission ->
                    new SimpleGrantedAuthority(permission.name())
            )
            .collect(Collectors.toSet());
}
```

## Understand this function

Suppose:

```java
authorities(Role.CREATOR)
```

### Step 1

Find `CREATOR` in the map:

```text
CREATOR
 ↓
USER_VIEW
POST_VIEW
POST_CREATE
POST_UPDATE
```

### Step 2

Convert each permission:

```java
Permission.POST_CREATE
```

into:

```java
new SimpleGrantedAuthority("POST_CREATE")
```

So the result becomes:

```text
SimpleGrantedAuthority("USER_VIEW")
SimpleGrantedAuthority("POST_VIEW")
SimpleGrantedAuthority("POST_CREATE")
SimpleGrantedAuthority("POST_UPDATE")
```

### Function's job

> **Role → configured permissions → Spring Security `GrantedAuthority` objects**

---

# 5. Use This Function in `UserEntity`

Now Spring Security needs to know:

> "What authorities does this particular user have?"

Our user has roles stored in the database.

For example:

```text
User
 └── CREATOR
```

So inside `UserEntity` we override:

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
```

Then create a `HashSet`:

```java
Set<GrantedAuthority> authorities = new HashSet<>();
```

Why `HashSet`?

Because a user can have multiple roles, and we want to collect all authorities without duplicates.

---

## Complete function

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {

    Set<GrantedAuthority> authorities = new HashSet<>();

    for (Role role : roles) {

        authorities.addAll(
            PermissionMapping.authorities(role)
        );

        authorities.add(
            new SimpleGrantedAuthority("ROLE_" + role.name())
        );
    }

    return authorities;
}
```

---

# 6. Understand `getAuthorities()` Carefully

Suppose database contains:

```text
User
Role = CREATOR
```

Spring calls:

```java
getAuthorities()
```

### Step 1 — create empty set

```java
Set<GrantedAuthority> authorities = new HashSet<>();
```

Initially:

```text
{}
```

### Step 2 — loop through user's roles

```java
for (Role role : roles)
```

Role is:

```text
CREATOR
```

### Step 3 — call our mapping function

```java
PermissionMapping.authorities(role)
```

This gives:

```text
USER_VIEW
POST_VIEW
POST_CREATE
POST_UPDATE
```

### Step 4 — add those authorities

```java
authorities.addAll(
    PermissionMapping.authorities(role)
);
```

Now:

```text
USER_VIEW
POST_VIEW
POST_CREATE
POST_UPDATE
```

### Step 5 — also add the role authority

```java
authorities.add(
    new SimpleGrantedAuthority("ROLE_" + role.name())
);
```

So:

```text
ROLE_CREATOR
```

is also added.

### Final result

```text
ROLE_CREATOR
USER_VIEW
POST_VIEW
POST_CREATE
POST_UPDATE
```

So **the role is still available**, but now we also have fine-grained permissions.

---

# 7. Why Keep `ROLE_CREATOR`?

Because granular authorization does **not necessarily mean removing roles**.

You can use both:

### Role-based

```java
.hasRole("ADMIN")
```

and:

### Permission-based

```java
.hasAuthority("POST_DELETE")
```

For example:

```java
.requestMatchers(HttpMethod.POST, "/posts/**")
    .hasAuthority(Permission.POST_CREATE.name())

.requestMatchers(HttpMethod.GET, "/posts/**")
    .hasAuthority(Permission.POST_VIEW.name())

.requestMatchers(HttpMethod.PUT, "/posts/**")
    .hasAuthority(Permission.POST_UPDATE.name())

.requestMatchers(HttpMethod.DELETE, "/posts/**")
    .hasAuthority(Permission.POST_DELETE.name())
```

This gives much more precise control.

---

# 8. Where Does `getAuthorities()` Get Used?

This is an important part of the flow.

Spring Security authentication needs an authenticated user with authorities.

During normal username/password authentication, the authentication process eventually obtains the user's `UserDetails`.

Because `UserEntity` implements `UserDetails`, Spring Security calls:

```java
userEntity.getAuthorities()
```

and gets:

```text
ROLE_CREATOR
POST_VIEW
POST_CREATE
POST_UPDATE
...
```

These authorities become part of the authenticated `Authentication` object.

Conceptually:

```text
Authentication
 ├── Principal
 ├── Credentials
 └── Authorities
       ├── ROLE_CREATOR
       ├── POST_VIEW
       ├── POST_CREATE
       └── POST_UPDATE
```

---

# 9. But We Are Using JWT — What Changes?

This is the important part in your project.

After login, you issue a JWT:

```text
Login
 ↓
AuthenticationManager
 ↓
UserDetails
 ↓
getAuthorities()
 ↓
Authentication successful
 ↓
JWT generated
```

On later requests, the user sends the JWT instead of username/password.

Example:

```text
GET /posts/10
Authorization: Bearer <JWT>
```

Your `JwtAuthFilter` intercepts this request.

So we need to authenticate the request **from the JWT**.

---

# 10. JWT Filter Must Also Put Authorities into Authentication

The JWT filter extracts the username/user information from the token.

Then it loads the user:

```java
UserDetails userDetails =
        userDetailsService.loadUserByUsername(username);
```

Because your returned object is `UserEntity`, you can get its authorities:

```java
userDetails.getAuthorities()
```

And create the authenticated object:

```java
UsernamePasswordAuthenticationToken authentication =
        new UsernamePasswordAuthenticationToken(
                userDetails,
                null,
                userDetails.getAuthorities()
        );
```

Then put it into the `SecurityContext`:

```java
SecurityContextHolder.getContext()
        .setAuthentication(authentication);
```

---

# 11. JWT Filter Complete Important Part

Conceptually your filter should do:

```java
String username = jwtService.extractUsername(token);

UserDetails userDetails =
        userDetailsService.loadUserByUsername(username);

UsernamePasswordAuthenticationToken authentication =
        new UsernamePasswordAuthenticationToken(
                userDetails,
                null,
                userDetails.getAuthorities()
        );

SecurityContextHolder.getContext()
        .setAuthentication(authentication);
```

The important line for granular authorization is:

```java
userDetails.getAuthorities()
```

Because this eventually comes from:

```java
UserEntity.getAuthorities()
```

which internally calls:

```java
PermissionMapping.authorities(role)
```

---

# 12. Complete JWT Request Flow

This is the **most important flow to remember**.

Suppose the user is:

```text
User = Sujeet
Role = CREATOR
```

### During authentication/login

```text
User enters username + password
             ↓
     AuthenticationManager
             ↓
       UserDetailsService
             ↓
          UserEntity
             ↓
    getAuthorities()
             ↓
   PermissionMapping.authorities(CREATOR)
             ↓
       Permissions
             ↓
  SimpleGrantedAuthority
             ↓
Authentication contains:
ROLE_CREATOR
POST_VIEW
POST_CREATE
POST_UPDATE
             ↓
          JWT created
```

---

# 13. Later API Request

User requests:

```http
POST /posts/10
Authorization: Bearer <JWT>
```

Now:

```text
HTTP Request
     ↓
JwtAuthFilter
     ↓
Validate JWT
     ↓
Extract username
     ↓
Load UserEntity
     ↓
userDetails.getAuthorities()
     ↓
UserEntity.getAuthorities()
     ↓
PermissionMapping.authorities(CREATOR)
     ↓
POST_CREATE
POST_UPDATE
POST_VIEW
     ↓
Create Authentication
     ↓
SecurityContext
```

Then Spring Security checks:

```java
.hasAuthority("POST_CREATE")
```

The user has:

```text
POST_CREATE ✅
```

Therefore:

```text
Request allowed
```

---

# 14. What If CREATOR Tries DELETE?

Request:

```http
DELETE /posts/10
```

Security configuration:

```java
.requestMatchers(HttpMethod.DELETE, "/posts/**")
    .hasAuthority(Permission.POST_DELETE.name())
```

But `CREATOR` has:

```text
POST_VIEW
POST_CREATE
POST_UPDATE
```

and does **not** have:

```text
POST_DELETE
```

Therefore:

```text
POST_DELETE ❌
       ↓
     403
Forbidden
```

Even though:

```text
ROLE_CREATOR ✅
```

the request is denied because the **required authority** is missing.

---

# 15. Complete Architecture

```text
                 DATABASE
                    │
                    │
              User → Role
                    │
                    ▼
          PermissionMapping
                    │
          Role → Permissions
                    │
                    ▼
        Permission.POST_CREATE
                    │
                    ▼
      SimpleGrantedAuthority
                    │
                    ▼
        UserEntity.getAuthorities()
                    │
                    ▼
             Authentication
                    │
                    ▼
                  JWT
                    │
                    │
          Later API Request
                    │
                    ▼
              JwtAuthFilter
                    │
                    ▼
        Load UserDetails/UserEntity
                    │
                    ▼
        getAuthorities()
                    │
                    ▼
          SecurityContext
                    │
                    ▼
          Spring Security
                    │
                    ▼
        hasAuthority(...)
                    │
             ┌──────┴──────┐
             ▼             ▼
           ALLOW          403
```

---

# 16. The 4 Most Important Pieces

### 1. `Permission`

Defines **what actions exist**.

```java
POST_CREATE
POST_UPDATE
POST_DELETE
POST_VIEW
```

---

### 2. `PermissionMapping`

Defines **which role gets which permissions**.

```java
CREATOR → POST_VIEW, POST_CREATE, POST_UPDATE
ADMIN   → POST_VIEW, POST_CREATE, POST_UPDATE, POST_DELETE
```

---

### 3. `authorities(Role role)`

Converts:

```text
Role
 ↓
Permissions
 ↓
SimpleGrantedAuthority
```

```java
public static Set<SimpleGrantedAuthority> authorities(Role role) {

    return map.get(role)
            .stream()
            .map(permission ->
                new SimpleGrantedAuthority(permission.name())
            )
            .collect(Collectors.toSet());
}
```

---

### 4. `UserEntity.getAuthorities()`

Converts the **current user's roles** into the complete set of Spring Security authorities.

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {

    Set<GrantedAuthority> authorities = new HashSet<>();

    for (Role role : roles) {

        authorities.addAll(
            PermissionMapping.authorities(role)
        );

        authorities.add(
            new SimpleGrantedAuthority("ROLE_" + role.name())
        );
    }

    return authorities;
}
```

Then the JWT filter uses:

```java
userDetails.getAuthorities()
```

when creating:

```java
UsernamePasswordAuthenticationToken
```

---

# 17. Final Cheat Sheet

```text
Permission Enum
       ↓
Define available permissions
       ↓
PermissionMapping
       ↓
Define Role → Permission relationship
       ↓
authorities(Role)
       ↓
Convert Permission → SimpleGrantedAuthority
       ↓
UserEntity.getAuthorities()
       ↓
Collect authorities for user's roles
       ↓
Authentication
       ↓
JWT
       ↓
JwtAuthFilter
       ↓
Load UserDetails
       ↓
getAuthorities()
       ↓
SecurityContext
       ↓
API Authorization
       ↓
hasAuthority("POST_CREATE")
       ↓
ALLOW / 403
```

### One-line memory trick

> **Role tells us what group the user belongs to; Permission tells us what the user is allowed to do; `GrantedAuthority` is the Spring Security representation of that permission.**
