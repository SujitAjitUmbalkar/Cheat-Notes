Below is the **ready-to-use `.md` (Markdown) file content**.
You can **copy–paste this into a file like `Entity_vs_DTO.md`** and upload it to **GitHub / notes / docs**.

---

````md
# Entity vs DTO (Detailed Comparison)

This document provides a **detailed, exam-ready comparison** between **Entity** and **DTO (Data Transfer Object)**.  
It covers **definitions, layers, annotations, usage, responsibilities, data flow, and best practices**.

---

## 📊 Entity vs DTO (Detailed Comparison Table)

| Aspect | **Entity** | **DTO (Data Transfer Object)** |
|------|------------|--------------------------------|
| **What it is** | Java class that represents a **database table** | Java class used to **transfer data between layers** |
| **Primary Purpose** | Persist data in database | Carry data safely between layers (especially API ↔ Service) |
| **Architecture Type** | Persistence / Domain object | Transfer / API object |
| **Layer Used In** | Persistence layer | Presentation & Service layers |
| **Mapped To** | Database table | Client request / response |
| **Who uses it** | JPA / Hibernate / Repository | Controller / Service |
| **Database Mapping** | ✔ Yes | ❌ No |
| **JPA Managed** | ✔ Yes | ❌ No |
| **Lifecycle** | Long-lived (persistent) | Short-lived (request/response scoped) |
| **Exposure to Client** | ❌ Should NOT be exposed | ✔ Designed to be exposed |
| **Security** | May contain sensitive fields (passwords, keys) | Contains only required safe fields |
| **Coupling** | Tightly coupled to DB schema | Loosely coupled to DB |
| **Change Impact** | DB change affects Entity | DB change usually does NOT affect DTO |
| **Validation** | Rarely used | Commonly used |
| **Serialization (JSON)** | Not recommended directly | Designed for JSON / XML |
| **Business Logic** | ❌ Should NOT contain | ❌ Should NOT contain |
| **Conversion Needed** | Used directly by JPA | Converted ↔ Entity in Service layer |

---

## 🧱 Annotations Used (Very Important)

### 🟦 Entity – JPA Annotations

| Annotation | Purpose |
|----------|--------|
| `@Entity` | Marks class as JPA entity |
| `@Table` | Maps class to DB table |
| `@Id` | Primary key |
| `@GeneratedValue` | Auto-generate primary key |
| `@Column` | Column mapping |
| `@OneToMany` | One-to-many relationship |
| `@ManyToOne` | Many-to-one relationship |
| `@JoinColumn` | Foreign key mapping |

### Example: Entity
```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue
    private Long id;
}
````

---

### 🟩 DTO – Validation / JSON Annotations

| Annotation      | Purpose                 |
| --------------- | ----------------------- |
| `@NotNull`      | Field must not be null  |
| `@NotBlank`     | Field must not be empty |
| `@Email`        | Email validation        |
| `@Size`         | Field length constraint |
| `@JsonProperty` | JSON field mapping      |
| `@JsonIgnore`   | Exclude field from JSON |

### Example: DTO

```java
public class UserDTO {

    @NotBlank
    private String name;

    @Email
    private String email;
}
```

---

## 🔁 Layer-wise Usage

| Layer      | Uses Entity    | Uses DTO |
| ---------- | -------------- | -------- |
| Controller | ❌              | ✔        |
| Service    | ✔ (conversion) | ✔        |
| Repository | ✔              | ❌        |
| Database   | ✔              | ❌        |

---

## 🔄 Data Flow

| Flow Stage           | Object Used |
| -------------------- | ----------- |
| Client → Controller  | DTO         |
| Controller → Service | DTO         |
| Service → Repository | Entity      |
| Repository → Service | Entity      |
| Service → Controller | DTO         |
| Controller → Client  | DTO         |

---

## 🚫 What Happens If You Skip DTO?

| Problem          | Explanation                      |
| ---------------- | -------------------------------- |
| Security risk    | Sensitive fields get exposed     |
| Tight coupling   | DB changes break API             |
| Poor API design  | Unnecessary data sent            |
| Hard to maintain | Entity changes affect all layers |

---
