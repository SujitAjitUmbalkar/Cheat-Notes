## Spring Data JPA Query Method Keywords

Assume:

```java
@Entity
public class Employee {
    private String name;
    private Integer age;
    private String email;
    private BigDecimal salary;
    private String department;
    private Boolean active;
    private LocalDateTime createdAt;
}
```

---

### 1. Retrieval Keywords

| Keyword    | Repository Method           | Service Layer Usage                     |
| ---------- | --------------------------- | --------------------------------------- |
| `findBy`   | `findByName(String name)`   | `employeeRepository.findByName(name)`   |
| `readBy`   | `readByName(String name)`   | `employeeRepository.readByName(name)`   |
| `getBy`    | `getByName(String name)`    | `employeeRepository.getByName(name)`    |
| `queryBy`  | `queryByName(String name)`  | `employeeRepository.queryByName(name)`  |
| `searchBy` | `searchByName(String name)` | `employeeRepository.searchByName(name)` |
| `streamBy` | `streamByName(String name)` | `employeeRepository.streamByName(name)` |

---

### 2. Count / Exists / Delete Keywords

| Keyword    | Repository Method                 | Service Layer Usage                           |
| ---------- | --------------------------------- | --------------------------------------------- |
| `countBy`  | `countByDepartment(String dept)`  | `employeeRepository.countByDepartment(dept)`  |
| `existsBy` | `existsByEmail(String email)`     | `employeeRepository.existsByEmail(email)`     |
| `deleteBy` | `deleteById(Long id)`             | `employeeRepository.deleteById(id)`           |
| `removeBy` | `removeByDepartment(String dept)` | `employeeRepository.removeByDepartment(dept)` |

---

### 3. Comparison Keywords

| Keyword            | Repository Method                                     | Service Layer Usage                                                        |
| ------------------ | ----------------------------------------------------- | -------------------------------------------------------------------------- |
| `Is`               | `findByNameIs(String name)`                           | `employeeRepository.findByNameIs(name)`                                    |
| `Equals`           | `findByNameEquals(String name)`                       | `employeeRepository.findByNameEquals(name)`                                |
| `Not`              | `findByNameNot(String name)`                          | `employeeRepository.findByNameNot(name)`                                   |
| `LessThan`         | `findByAgeLessThan(Integer age)`                      | `employeeRepository.findByAgeLessThan(25)`                                 |
| `LessThanEqual`    | `findByAgeLessThanEqual(Integer age)`                 | `employeeRepository.findByAgeLessThanEqual(25)`                            |
| `GreaterThan`      | `findBySalaryGreaterThan(BigDecimal salary)`          | `employeeRepository.findBySalaryGreaterThan(new BigDecimal("50000"))`      |
| `GreaterThanEqual` | `findBySalaryGreaterThanEqual(BigDecimal salary)`     | `employeeRepository.findBySalaryGreaterThanEqual(new BigDecimal("50000"))` |
| `Between`          | `findBySalaryBetween(BigDecimal min, BigDecimal max)` | `employeeRepository.findBySalaryBetween(min,max)`                          |

---

### 4. String Matching Keywords

| Keyword         | Repository Method                       | Service Layer Usage                                 |
| --------------- | --------------------------------------- | --------------------------------------------------- |
| `Like`          | `findByNameLike(String pattern)`        | `employeeRepository.findByNameLike("%Raj%")`        |
| `NotLike`       | `findByNameNotLike(String pattern)`     | `employeeRepository.findByNameNotLike("%Raj%")`     |
| `Containing`    | `findByNameContaining(String text)`     | `employeeRepository.findByNameContaining("Raj")`    |
| `NotContaining` | `findByNameNotContaining(String text)`  | `employeeRepository.findByNameNotContaining("Raj")` |
| `StartingWith`  | `findByNameStartingWith(String prefix)` | `employeeRepository.findByNameStartingWith("Ra")`   |
| `EndingWith`    | `findByNameEndingWith(String suffix)`   | `employeeRepository.findByNameEndingWith("esh")`    |
| `IgnoreCase`    | `findByNameIgnoreCase(String name)`     | `employeeRepository.findByNameIgnoreCase("RAJ")`    |

---

### 5. Collection Keywords

| Keyword | Repository Method                           | Service Layer Usage                               |
| ------- | ------------------------------------------- | ------------------------------------------------- |
| `In`    | `findByDepartmentIn(List<String> depts)`    | `employeeRepository.findByDepartmentIn(depts)`    |
| `NotIn` | `findByDepartmentNotIn(List<String> depts)` | `employeeRepository.findByDepartmentNotIn(depts)` |

---

### 6. Null Check Keywords

| Keyword     | Repository Method        | Service Layer Usage                         |
| ----------- | ------------------------ | ------------------------------------------- |
| `IsNull`    | `findByEmailIsNull()`    | `employeeRepository.findByEmailIsNull()`    |
| `Null`      | `findByEmailNull()`      | `employeeRepository.findByEmailNull()`      |
| `IsNotNull` | `findByEmailIsNotNull()` | `employeeRepository.findByEmailIsNotNull()` |
| `NotNull`   | `findByEmailNotNull()`   | `employeeRepository.findByEmailNotNull()`   |

---

### 7. Boolean Keywords

| Keyword | Repository Method     | Service Layer Usage                      |
| ------- | --------------------- | ---------------------------------------- |
| `True`  | `findByActiveTrue()`  | `employeeRepository.findByActiveTrue()`  |
| `False` | `findByActiveFalse()` | `employeeRepository.findByActiveFalse()` |

---

### 8. Date & Time Keywords

| Keyword  | Repository Method                           | Service Layer Usage                              |
| -------- | ------------------------------------------- | ------------------------------------------------ |
| `Before` | `findByCreatedAtBefore(LocalDateTime date)` | `employeeRepository.findByCreatedAtBefore(date)` |
| `After`  | `findByCreatedAtAfter(LocalDateTime date)`  | `employeeRepository.findByCreatedAtAfter(date)`  |

---

### 9. Logical Operator Keywords

| Keyword | Repository Method                             | Service Layer Usage                                |
| ------- | --------------------------------------------- | -------------------------------------------------- |
| `And`   | `findByNameAndAge(String name,Integer age)`   | `employeeRepository.findByNameAndAge(name,age)`    |
| `Or`    | `findByNameOrEmail(String name,String email)` | `employeeRepository.findByNameOrEmail(name,email)` |

---

### 10. Sorting Keywords

| Keyword   | Repository Method                                | Service Layer Usage                                          |
| --------- | ------------------------------------------------ | ------------------------------------------------------------ |
| `OrderBy` | `findByDepartmentOrderBySalaryDesc(String dept)` | `employeeRepository.findByDepartmentOrderBySalaryDesc(dept)` |
| `Asc`     | `findByDepartmentOrderByNameAsc(String dept)`    | `employeeRepository.findByDepartmentOrderByNameAsc(dept)`    |
| `Desc`    | `findByDepartmentOrderByNameDesc(String dept)`   | `employeeRepository.findByDepartmentOrderByNameDesc(dept)`   |

---

### 11. Limiting Keywords

| Keyword | Repository Method                | Service Layer Usage                                 |
| ------- | -------------------------------- | --------------------------------------------------- |
| `Top`   | `findTop5ByOrderBySalaryDesc()`  | `employeeRepository.findTop5ByOrderBySalaryDesc()`  |
| `First` | `findFirstByOrderBySalaryDesc()` | `employeeRepository.findFirstByOrderBySalaryDesc()` |

---

### 12. Distinct Keywords

| Keyword    | Repository Method                       | Service Layer Usage                                 |
| ---------- | --------------------------------------- | --------------------------------------------------- |
| `Distinct` | `findDistinctByDepartment(String dept)` | `employeeRepository.findDistinctByDepartment(dept)` |

---

## Most Important Interview Examples

| Requirement               | Repository Method                        | Service Usage                                                          |
| ------------------------- | ---------------------------------------- | ---------------------------------------------------------------------- |
| Find by name              | `findByName()`                           | `employeeRepository.findByName(name)`                                  |
| Find age > 25             | `findByAgeGreaterThan()`                 | `employeeRepository.findByAgeGreaterThan(25)`                          |
| Find salary between range | `findBySalaryBetween()`                  | `employeeRepository.findBySalaryBetween(min,max)`                      |
| Find active employees     | `findByActiveTrue()`                     | `employeeRepository.findByActiveTrue()`                                |
| Check email exists        | `existsByEmail()`                        | `employeeRepository.existsByEmail(email)`                              |
| Count employees in dept   | `countByDepartment()`                    | `employeeRepository.countByDepartment(dept)`                           |
| Find top 10 salaries      | `findTop10ByOrderBySalaryDesc()`         | `employeeRepository.findTop10ByOrderBySalaryDesc()`                    |
| Search name contains text | `findByNameContaining()`                 | `employeeRepository.findByNameContaining("raj")`                       |
| Find created after date   | `findByCreatedAtAfter()`                 | `employeeRepository.findByCreatedAtAfter(date)`                        |
| Find by dept and salary   | `findByDepartmentAndSalaryGreaterThan()` | `employeeRepository.findByDepartmentAndSalaryGreaterThan(dept,salary)` |

**Memorize these keywords in order:**

`findBy → And/Or → GreaterThan/LessThan → Between → Containing → In → IsNull → True/False → OrderBy → Top/First → Distinct → CountBy → ExistsBy → DeleteBy`

These cover almost all query methods used in real projects.

## Spring Data JPA Query Method Rules

### Rule 1: Start with a valid prefix

Use prefixes like `findBy`, `countBy`, `existsBy`, `deleteBy`.

```java
findByName(String name)
```

---

### Rule 2: Use exact entity field names

Method field names must exactly match entity field names.

```java
private String email;

findByEmail(String email)
```

---

### Rule 3: Follow Java Camel Case

Convert entity fields to Camel Case in method names.

```java
private LocalDateTime createdAt;

findByCreatedAtAfter(date)
```

---

### Rule 4: One condition = One parameter

Each field condition generally requires one parameter.

```java
findByNameAndAge(String name, Integer age)
```

---

### Rule 5: Combine conditions using And / Or

Use `And` and `Or` to join multiple conditions.

```java
findByDepartmentAndSalaryGreaterThan(dept, salary)
```

---

### Rule 6: Place comparison keyword after field

Keywords like `GreaterThan`, `LessThan`, `Between` come after field name.

```java
findBySalaryGreaterThan(amount)
```

---

### Rule 7: Place IgnoreCase after field

`IgnoreCase` always follows the field it applies to.

```java
findByEmailIgnoreCase(email)
```

---

### Rule 8: Use Top / First before By

`Top` and `First` are placed before `By`.

```java
findTop5ByOrderBySalaryDesc()
```

---

### Rule 9: Sorting comes at the end

Use `OrderBy<Field>Asc/Desc` after conditions.

```java
findByDepartmentOrderByNameAsc(dept)
```

---

### Rule 10: Use In / NotIn with collections

Pass a collection parameter when using `In` or `NotIn`.

```java
findByDepartmentIn(List<String> departments)
```

---

### Rule 11: Null checks need no parameter

`IsNull` and `IsNotNull` methods don't require arguments.

```java
findByEmailIsNull()
```

---

### Rule 12: Boolean checks need no parameter

`True` and `False` methods don't require arguments.

```java
findByActiveTrue()
```

---

### Rule 13: Nested fields use property traversal

Traverse relationships by chaining field names.

```java
findByDepartmentName(String name)
```

---

### Rule 14: Distinct comes before By

Place `Distinct` before `By`.

```java
findDistinctByDepartment(dept)
```

---

### Rule 15: Method name is the query

Spring Data JPA generates SQL from the method name.

```java
findByNameContainingIgnoreCase(name)
```

---

## Universal Formula

```java
<Prefix>
[Distinct]
[Top/First]
By
<Field>
[Keyword]
[And/Or]
<Field>
[Keyword]
OrderBy<Field>Asc/Desc
```

### Example

```java
findDistinctTop10ByDepartmentAndSalaryGreaterThanOrderByNameAsc(
        String department,
        BigDecimal salary
)
```

If you remember only one thing, remember:

```text
Prefix + Field + Keyword + And/Or + Field + Keyword + OrderBy
```

Everything in Spring Data JPA query methods follows this pattern.
