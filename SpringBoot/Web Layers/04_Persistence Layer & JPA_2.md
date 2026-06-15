
# Persistence Layer & JPA – Detailed Notes

Persistence Layer is responsible for:

• Saving data into database  
• Fetching data from database  
• Updating records  
• Deleting records  

It acts as a bridge between:
Service Layer → Database

In Spring Boot architecture:

<img width="1458" height="297" alt="image" src="https://github.com/user-attachments/assets/3cfda026-e88e-40ab-b5c5-859a8ae99906" />

Controller → Service → Repository → Database

Entity classes represent database tables.

------------------------------------------------------------

2️⃣ Entity in JPA
============================================================

## What is an Entity?

An Entity is a normal Java class that represents a database table.

Entity class → represents a table
Entity object (instance) → represents one row
Fields of entity → represent columns

Entity follows ORM (Object Relational Mapping):
1. Object → Table  
2. Fields → Columns  

------------------------------------------------------------

## How to Write an Entity

Basic Structure:

```java
import jakarta.persistence.*;

@Entity
@Table(name = "employee")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    private Double salary;
}
````

## Rules for Entity

1. Must be annotated with @Entity
2. Must have one and only one @Id
3. Must have default constructor
4. Should follow Java Bean conventions
5. Class should not be final

---

# 3️⃣ Important JPA Annotations

---

## @Entity

Marks class as database table.

Parameters:
• name → Optional entity name

Example:
@Entity(name = "emp")

---

## @Table  (more info in Annotation - List )

Specifies table details.

Parameters:
• name → Table name
• schema → Schema name
• catalog → Database name
• uniqueConstraints → Composite unique constraints
• indexes → Database indexes

Example:
@Table(name = "employees")

---

## @Id

Marks primary key field.

Every entity must have exactly one @Id.

---

## @GeneratedValue

Auto-generates ID value.

Parameter:
• strategy → GenerationType

Generation Strategies:

1. IDENTITY
   Uses database auto-increment
   Best for MySQL

2. SEQUENCE
   Uses database sequence
   Best for PostgreSQL / Oracle

3. AUTO
   Hibernate decides automatically

---

## @Column

Defines column properties.

Parameters:
• name → Column name
• nullable → true/false
• unique → true/false
• length → String length
• updatable → true/false
• insertable → true/false
• columnDefinition → Custom SQL definition

Example:
@Column(nullable = false, unique = true)

---

# 4️⃣ H2 Database

H2 is:

• Lightweight
• In-memory database
• Used for development/testing
• Starts automatically with Spring Boot

Data is lost when application stops (in-memory mode).

---

## H2 Dependency

In pom.xml:

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

---

# 5️⃣ H2 Configuration (application.properties)

Basic Configuration:

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

Important Properties Explained:

spring.datasource.url
→ Defines database URL

spring.h2.console.enabled
→ Enables browser console

spring.jpa.hibernate.ddl-auto
Options:
• create → Creates tables every time
• create-drop → Creates and drops on shutdown
• update → Updates schema without deleting data
• validate → Validates schema only
• none → No action

spring.jpa.show-sql
→ Shows SQL in console

---

# 6️⃣ How to Access H2 Console

Steps:

1. Enable console property
2. Run application
3. Open browser
4. Visit:

[http://localhost:8080/h2-console](http://localhost:8080/h2-console)

5. Enter:
   JDBC URL: jdbc:h2:mem:testdb
   Username: sa
   Password: (empty)

Then click Connect.

---

# 7️⃣ JPA Repository Interfaces

Spring Data JPA provides repository interfaces.

Most commonly used:

JpaRepository<T, ID>

Example:

```java
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {
}
```

T → Entity class
ID → Type of primary key

---

# 8️⃣ Important JpaRepository Methods

1. save() → Insert or update
2. findById() → Fetch by ID
3. findAll() → Fetch all records
4. deleteById() → Delete record
5. existsById() → Check existence
6 count() → Count records

No implementation required.
Spring generates implementation at runtime.

---

# 9️⃣ Custom Query Methods

Spring can generate queries based on method names:

Example:

findByName(String name)

findBySalaryGreaterThan(Double salary)

Spring converts this to SQL automatically.

---

# 🔟 Transaction Management

Database operations should run inside transactions.

Use:
@Transactional

Ensures:
All operations succeed OR none succeed.

---

# 11️⃣ Entity Lifecycle (Important)

States:

1. Transient → New object, not saved
2. Persistent → Managed by Hibernate
3. Detached → Not managed anymore
4. Removed → Marked for deletion

Hibernate manages state automatically.

---

# 12️⃣ Why Use JPA Instead of JDBC?

JDBC:
• Manual SQL writing
• Manual connection handling
• More boilerplate code

JPA:
• No manual SQL (in most cases)
• Object-based programming
• Cleaner architecture
• Faster development

---

# Final Summary

Entity → Represents table
@Repository → Handles DB operations
JpaRepository → Provides CRUD
H2 → Development database
@GeneratedValue → Auto ID
ddl-auto → Controls schema behavior

Persistence Layer cleanly manages database interaction using ORM principles.
