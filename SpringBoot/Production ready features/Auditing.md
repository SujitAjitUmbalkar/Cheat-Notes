
# 📌 Auditing in Spring Boot (JPA Auditing)

## 🔎 1. What is Auditing?

Auditing means automatically tracking:

* `createdAt` → When record was created
* `updatedAt` → When record was modified
* `createdBy` → Who created
* `updatedBy` → Who modified
---

# 🧭 3. Steps to Implement Auditing (Production Level)

---

# ✅ Step 1: Enable JPA Auditing

### 📍 Why?

Spring does NOT enable auditing automatically.

### 📍 How?

```java
@SpringBootApplication
@EnableJpaAuditing
public class Application {
}
```

### 🔹 What it does?

Activates:

| Annotation          | Purpose            |
| ------------------- | ------------------ |
| `@CreatedDate`      | Set only at INSERT |
| `@LastModifiedDate` | Updated at UPDATE  |
| `@CreatedBy`        | Set creator        |
| `@LastModifiedBy`   | Updated modifier   |

For CreatedBy and LastModifiedBy, you must implement:

`AuditorAware <T> `
Spring does NOT know the current user automatically — you must tell it how to get the logged-in user (usually from Spring Security).
Without this → auditing will not work.

---

# ✅ Step 2: Create Base Entity Add Auditing Fields

Real developers DO NOT repeat fields in every entity.

They create a **Base Class**.

---

## 🏗 Create BaseEntity

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Auditing @Getter @Setter
public abstract class BaseEntity
{
    @CreatedDate
    @Column(updatable = false, nullable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime updatedDate;

    @CreatedBy
    private String createdBy;

    @LastModifiedBy
    private String updatedBy;
}
```

### 🔹 Why `@MappedSuperclass`?

* Fields become part of child entity table
* No separate table created

### 🔹 `WhyEnable Entity Listener`     

This listener:

* Listens to JPA lifecycle events
* Before Persist
* Before Update
* Automatically fills fields

Without this → fields remain null.


# ✅ Step 3 : Use BaseEntity in Entities

```java
@Entity
public class Patient extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
}
```
All fields will be added in child class 

Now auditing works automatically. except createdBy and Updatedby , to do this , we have to implement AuditorAware

---

# ✅ Step 4: Implement AuditorAware (VERY IMPORTANT)  .. for CreatedBy and UpdatedBy

Spring does not know:

> Who is current user?

So we provide logic.

---

## 🏗 Create AuditorAware Implementation

```java
@Component
public class AuditorAwareImpl implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        return Optional.of("SYSTEM_USER");
    }
}
```
### 🔎 Why important?

Whenever:

* Entity is saved by whom 
* Entity is updated by whom

Spring calls:

```java
getCurrentAuditor()
```

Return value goes into:

* createdBy
* updatedBy

---

## 🔥 Real Developer Version (With Spring Security)

```java
@Override
public Optional<String> getCurrentAuditor() {
    Authentication auth = SecurityContextHolder
            .getContext()
            .getAuthentication();

    if (auth == null || !auth.isAuthenticated()) {
        return Optional.of("SYSTEM");
    }

    return Optional.of(auth.getName());
}
```

This fetches logged-in username.

---


# 🧠 How It Works Internally

1. Entity goes into Persistence Context
2. Before INSERT → `@CreatedDate` & `@CreatedBy` filled
3. Before UPDATE → `@LastModifiedDate` & `@LastModifiedBy` filled
4. Hibernate performs dirty checking
5. SQL executed

---

# ⚙️ Important Properties

In `application.properties`:

```properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

## 👉 Hibernate Envers

It creates:

* Separate audit tables
* Version history of each row

Example:

```
patient
patient_AUD
```

Used in large enterprise systems.

---

# 📌 Final Summary

| Step                   | Why                   |
| ---------------------- | --------------------- |
| Enable Auditing        | Activates feature     |
| Create BaseEntity      | Clean reusable design |
| Add Audit Annotations  | Auto-populate fields  |
| Add EntityListener     | Connect lifecycle     |
| Implement AuditorAware | Provide current user  |
| Use in Entities        | Apply auditing        |

---

# 🎓 Interview Points

* Difference between `@CreatedDate` and `@LastModifiedDate`
* Why use `@MappedSuperclass`
* How AuditorAware works
* How auditing connects to Persistence Context
* Difference between simple auditing & Envers
