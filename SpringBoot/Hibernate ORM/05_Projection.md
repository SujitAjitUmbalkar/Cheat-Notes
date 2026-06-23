# Projection - Spring Data JPA Notes

## Definition

```text
Projection is a technique used to fetch only required columns from the database instead of loading the complete entity.
```

Example:

Entity:

```java
Patient
{
    id,
    name,
    email,
    bloodGroup,
    gender,
    birthDate
}
```

Need only:

```text
id
name
email
```

Use Projection.

---

# Why Use Projection?

### 1. Better Performance

```text
Fetches fewer columns.
Reduces network traffic.
Uses less memory.
```

### 2. Faster Queries

```text
SELECT name,email

instead of

SELECT *
```

---

### 3. Hide Sensitive Data

```text
Do not expose:
password
salary
address

Return only required fields.
```

---

# Types of Projection

| Type                 | Uses @Query?  | Most Used |
| -------------------- | ------------- | --------- |
| Interface Projection | No (usually)  | ⭐⭐⭐       |
| DTO/Class Projection | Yes (usually) | ⭐⭐⭐       |
| Dynamic Projection   | Optional      | ⭐⭐        |

---

# 1. Interface Projection

## Interface

```java
public interface IPatientInfo {

    Long getId();

    String getName();

    String getEmail();
}
```

---

## Repository

### Derived Query

```java
List<IPatientInfo> findByBloodGroup(
        BloodGroupType bloodGroup
);
```

No `@Query` required.

---

### Custom Query

```java
@Query("""
SELECT p.id as id,
       p.name as name,
       p.email as email
FROM Patient p
""")
List<IPatientInfo> getAllPatientsInfo();
```

---

## Important Rule

Getter names must match entity fields.

```java
getName()  -> name

getEmail() -> email
```

---

# Interface Projection Internally

Spring creates a proxy object.

```text
Patient Entity
      ↓
Projection Proxy
      ↓
getId()
getName()
getEmail()
```

---

### Printing Issue

Avoid:

```java
System.out.println(patientInfo);
```

Use:

```java
System.out.println(
        patientInfo.getName()
);
```

Reason:

```text
Projection interfaces are proxy objects.
Direct printing may show proxy details.
```

---

# 2. DTO/Class Projection

Used when result does not match entity structure.

---

## DTO

```java
public record PatientDto(
        Long id,
        String name,
        String email
) {}
```

---

## Repository

```java
@Query("""
SELECT new com.example.dto.PatientDto(
       p.id,
       p.name,
       p.email
)
FROM Patient p
""")
List<PatientDto> getPatients();
```

---

## Why DTO Projection?

Need custom object.

Example:

```text
Patient Name
Appointment Count
```

No such entity exists.

Create DTO.

---

# 3. Dynamic Projection

Repository:

```java
<T> List<T> findByBloodGroup(
        BloodGroupType bloodGroup,
        Class<T> type
);
```

---

Usage:

```java
patientRepository.findByBloodGroup(
        O_POSITIVE,
        IPatientInfo.class
);
```

or

```java
patientRepository.findByBloodGroup(
        O_POSITIVE,
        PatientDto.class
);
```

---

# Aggregation Projection

Used when fetching calculated values.

Example:

```text
O_POSITIVE -> 5

A_POSITIVE -> 3

B_NEGATIVE -> 1
```

---

## DTO

```java
public record BloodGroupCountDto(
        BloodGroupType bloodGroup,
        Long count
) {}
```

---

## Repository

```java
@Query("""
SELECT new BloodGroupCountDto(
       p.bloodGroup,
       COUNT(p)
)
FROM Patient p
GROUP BY p.bloodGroup
""")
List<BloodGroupCountDto>
getBloodGroupCounts();
```

---

## Concepts Used

```text
COUNT()
GROUP BY
DTO Projection
Constructor Expression
```

---

# Interface vs DTO Projection

| Feature              | Interface Projection | DTO Projection |
| -------------------- | -------------------- | -------------- |
| Partial Columns      | ✅                    | ✅              |
| Custom Object        | ❌                    | ✅              |
| Aggregation Result   | ❌                    | ✅              |
| Usually Needs @Query | ❌                    | ✅              |
| Spring Creates Proxy | ✅                    | ❌              |
| Most Common          | ✅                    | ✅              |

---

# When to Use What?

### Use Interface Projection

```text
Need only existing entity fields.
Simple read operations.
```

Example:

```text
id
name
email
```

---

### Use DTO Projection

```text
Need custom response.
Need COUNT, SUM, AVG.
Need calculated values.
```

Example:

```text
BloodGroup + Count
Department + DoctorCount
Doctor + AppointmentCount
```

---

# Interview Questions

### What is Projection?

```text
Fetching only required fields instead of the complete entity.
```

### Why use Projection?

```text
Performance improvement.
Less memory usage.
Hide unnecessary fields.
```

### Why can't we print Interface Projection directly?

```text
Because Spring creates proxy objects for interface projections.
Use getter methods to access values.
```

### When is DTO Projection preferred?

```text
When the result does not match entity structure or contains calculated fields.
```

---

# Memory Trick

```text
Existing Entity Fields
        ↓
Interface Projection

Custom Fields / COUNT / GROUP BY
        ↓
DTO Projection + @Query
```

### Formula

```text
Need Few Entity Columns?
→ Interface Projection

Need Custom Result?
→ DTO Projection

Need Runtime Choice?
→ Dynamic Projection
```
