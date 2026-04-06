# Entity vs DTO (Detailed Comparison)

This document provides a **detailed, exam-ready comparison** between **Entity** and **DTO (Data Transfer Object)**.  
It covers **definitions, layers, annotations, usage, responsibilities, data flow, and best practices**.

---

## 📊 Entity vs DTO (Detailed Comparison Table)

| Aspect | Entity | DTO (Data Transfer Object) |
|------|--------|----------------------------|
| What it is | Java class that represents a database table | Java class used to transfer data between layers |
| Primary Purpose | Persist data in database | Carry data safely between layers (API ↔ Service) |
| Architecture Type | Persistence / Domain object | Transfer / API object |
| Layer Used In | Persistence layer | Presentation & Service layers |
| Mapped To | Database table | Client request / response |
| Who uses it | JPA / Hibernate / Repository | Controller / Service |
| Database Mapping | Yes | No |
| JPA Managed | Yes | No |
| Lifecycle | Long-lived (persistent) | Short-lived (request/response scoped) |
| Exposure to Client | No (should not be exposed) | Yes (designed to be exposed) |
| Security | May contain sensitive fields | Contains only required safe fields |
| Coupling | Tightly coupled to DB schema | Loosely coupled to DB |
| Change Impact | DB change affects Entity | DB change usually does not affect DTO |
| Validation | Rarely used | Commonly used |
| Serialization (JSON) | Not recommended directly | Designed for JSON / XML |
| Business Logic | Should not contain | Should not contain |
| Conversion Needed | Used directly by JPA | Converted ↔ Entity in Service layer |

---

### 🟩 DTO – Validation / JSON Annotations (Not compulsory , but you can use these annotations in DTO )

| Annotation | Use / Purpose | Important Parameters |
|------------|--------------|----------------------|
| @NotNull | Field must not be null | message |
| @NotEmpty | Must not be null and size > 0 (String, Collection, Map, Array) | message |
| @NotBlank | String must not be null and must contain non-whitespace characters | message |
| @Size | Validates size/length range | min, max, message |
| @Min | Number must be ≥ given value | value, message |
| @Max | Number must be ≤ given value | value, message |
| @Email | Valid email format | regexp, message |
| @Pattern | Must match given regex | regexp, message |
| @Positive | Number must be > 0 | message |
| @PositiveOrZero | Number must be ≥ 0 | message |
| @Negative | Number must be < 0 | message |
| @NegativeOrZero | Number must be ≤ 0 | message |
| @Past | Date must be in the past | message |
| @PastOrPresent | Date must be past or present | message |
| @Future | Date must be in the future | message |
| @FutureOrPresent | Date must be future or present | message |
| @Digits | Controls number of integer & fraction digits | integer, fraction, message |
| @DecimalMin | Decimal number must be ≥ value | value, inclusive, message |
| @DecimalMax | Decimal number must be ≤ value | value, inclusive, message |
| @AssertTrue | Boolean must be true | message |
| @AssertFalse | Boolean must be false | message |
| @Valid | Triggers validation on nested objects | (no parameters) |

### Example: DTO
``` 
public class UserDTO {

    @NotNull(message = "ID cannot be null")
    private Long id;

    @NotBlank(message = "Name cannot be blank")
    @Size(min = 3, max = 50, message = "Name must be between 3 to 50 characters")
    private String name;

    @Email(message = "Invalid email format")
    @NotBlank(message = "Email is required")
    private String email;

    @Pattern(regexp = "^[0-9]{10}$", message = "Phone number must be 10 digits")
    private String phone;

    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 60, message = "Age must not exceed 60")
    private Integer age;

    @Positive(message = "Salary must be positive")
    private Double salary;

    @PositiveOrZero(message = "Bonus cannot be negative")
    private Double bonus;

    @Negative(message = "Debt should be negative")
    private Double debt;

    @NegativeOrZero(message = "Balance must be zero or negative")
    private Double balance;

    @Past(message = "Date of birth must be in the past")
    private LocalDate dob;

    @PastOrPresent(message = "Joining date cannot be in future")
    private LocalDate joiningDate;

    @Future(message = "Expiry date must be in future")
    private LocalDate expiryDate;

    @FutureOrPresent(message = "Event date must be today or future")
    private LocalDate eventDate;

    @Digits(integer = 5, fraction = 2, message = "Invalid amount format")
    private Double amount;

    @DecimalMin(value = "1000.50", inclusive = true, message = "Minimum amount is 1000.50")
    private Double minAmount;

    @DecimalMax(value = "9999.99", inclusive = true, message = "Maximum amount is 9999.99")
    private Double maxAmount;

    @AssertTrue(message = "Terms must be accepted")
    private Boolean termsAccepted;

    @AssertFalse(message = "User must not be banned")
    private Boolean isBanned;

    @NotEmpty(message = "Roles cannot be empty")
    private List<String> roles;

    // Nested Object Validation
    @Valid
    private AddressDTO address;

    // getters & setters
}
```

## 🚫 What Happens If You Skip DTO?

| Problem          | Explanation                      |
| ---------------- | -------------------------------- |
| Security risk    | Sensitive fields get exposed     |
| Tight coupling   | DB changes break API             |
| Poor API design  | Unnecessary data sent            |
| Hard to maintain | Entity changes affect all layers |


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

### 🟦 Functions Used in @Table (it applies on whole table ) 

| Belongs To | Annotation / Function | Brief Use |
|------------|----------------------|------------|
| — | @Table | Defines table-level configuration for an entity |
| @Table | name() | Sets the database table name |
| @Table | schema() | Sets the schema in which the table exists |
| @Table | catalog() | Sets the database/catalog name , catalog(schema(table))) |
| @Table | uniqueConstraints() | Holds one or more @UniqueConstraint annotations |
| @Table | indexes() | Holds one or more @Index annotations |
| @Table.uniqueConstraints() | @UniqueConstraint | Defines a composite unique constraint |
| @UniqueConstraint | name() | Name of the unique constraint |
| @UniqueConstraint | columnNames() | Columns that must be unique together |
| @Table.indexes() | @Index | Defines an index for performance |
| @Index | name() | Name of the index |
| @Index | columnList() | Column(s) included in the index |
| @Index | unique() | Whether the index enforces uniqueness |

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


### Relation Between them 

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
