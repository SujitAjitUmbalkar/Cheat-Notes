# JPA provides two main types of interfaces

# 1️⃣  CORE JPA INTERFACES (Standard JPA API) (jakarta.persistence)
These are defined by the **JPA Specification** and implemented by providers like Hibernate.
(now under Jakarta EE, previously Java EE)


```
EntityManagerFactory
        |
        v
EntityManager
        |
        +---- getTransaction() → EntityTransaction
        |
        +---- createQuery() → Query
        |                        ↑
        |                        |
        |                  TypedQuery<T>
        |
        +---- getCriteriaBuilder() → CriteriaBuilder
                                       |
                                       v
                                  CriteriaQuery<T>
```


---

# 🔷 1️⃣ EntityManagerFactory

## ✅ Responsibility

* Creates `EntityManager` objects
* Heavy-weight object
* Created once per application

---

## 🔷 Methods of EntityManagerFactory

| Method                         | Return Type   | Responsibility                  |
| ------------------------------ | ------------- | ------------------------------- |
| createEntityManager()          | EntityManager | Creates new persistence context |
| createEntityManager(Map props) | EntityManager | Create with custom properties   |
| close()                        | void          | Close factory                   |
| isOpen()                       | boolean       | Check if open                   |
| getProperties()                | Map           | Get configuration               |

---

# 🔷 2️⃣ EntityManager

## ✅ Responsibility

* Core interface of JPA
* Manages persistence context (1st level cache)
* Performs CRUD operations
* Executes queries
* Controls entity lifecycle

---

## 🔷 Methods of EntityManager

### 🔹 Lifecycle Methods

| Method           | Use                                    |
| ---------------- | -------------------------------------- |
| persist(entity)  | Make entity managed (INSERT on flush)  |
| merge(entity)    | Copy detached state to managed entity  |
| remove(entity)   | Mark entity for deletion               |
| detach(entity)   | Remove entity from persistence context |
| clear()          | Remove all managed entities            |
| contains(entity) | Check if entity is managed             |

---

### 🔹 Fetch Methods

| Method                  | Use                      |
| ----------------------- | ------------------------ |
| find(Class, id)         | Fetch entity immediately |
| getReference(Class, id) | Return lazy proxy        |

---

### 🔹 Query Creation

| Method                     | Use          |
| -------------------------- | ------------ |
| createQuery(String)        | JPQL query   |
| createNamedQuery(String)   | Named JPQL   |
| createNativeQuery(String)  | Native SQL   |
| createQuery(CriteriaQuery) | Criteria API |

---

### 🔹 Synchronization & Transaction

| Method            | Use                            |
| ----------------- | ------------------------------ |
| flush()           | Synchronize with database      |
| getTransaction()  | Get transaction (Java SE only) |
| joinTransaction() | Join existing transaction      |
| close()           | Close EntityManager            |
| isOpen()          | Check if open                  |

---

# 🔷 3️⃣ EntityTransaction

## ✅ Responsibility

* Used in Java SE applications
* Controls transaction manually
* Not commonly used in Spring Boot

---

## 🔷 Methods of EntityTransaction

| Method            | Use                  |
| ----------------- | -------------------- |
| begin()           | Start transaction    |
| commit()          | Commit transaction   |
| rollback()        | Rollback transaction |
| isActive()        | Check if active      |
| setRollbackOnly() | Mark rollback        |
| getRollbackOnly() | Check rollback flag  |

---

# 🔷 4️⃣ Query

## ✅ Responsibility

* Represents JPQL or native query
* Used to fetch or modify data

---

## 🔷 Methods of Query

| Method            | Use                       |
| ----------------- | ------------------------- |
| getResultList()   | Get multiple results      |
| getSingleResult() | Get one result            |
| executeUpdate()   | For update/delete queries |
| setParameter()    | Bind parameters           |
| setMaxResults()   | Limit records             |
| setFirstResult()  | Pagination offset         |
| getMaxResults()   | Get limit                 |
| getFirstResult()  | Get offset                |

---

# 🔷 5️⃣ TypedQuery<T>

## ✅ Responsibility

* Type-safe version of Query
* Returns specific entity type
* No casting required

---

## 🔷 Inheritance

```
Query
   ↑
TypedQuery<T>
```

---

## 🔷 Methods of TypedQuery

(Same as Query but returns type-safe result)

| Method            | Use             |
| ----------------- | --------------- |
| getResultList()   | Returns List<T> |
| getSingleResult() | Returns T       |
| setParameter()    | Bind parameters |
| setMaxResults()   | Limit results   |
| setFirstResult()  | Offset          |

---

# 🔷 6️⃣ CriteriaBuilder

## ✅ Responsibility

* Used to build dynamic type-safe queries
* Part of Criteria API

---

## 🔷 Important Methods

| Method             | Use                  |
| ------------------ | -------------------- |
| createQuery(Class) | Create CriteriaQuery |
| equal()            | Equality condition   |
| like()             | LIKE condition       |
| greaterThan()      | Greater condition    |
| and()              | AND condition        |
| or()               | OR condition         |

---

# 🔷 7️⃣ CriteriaQuery<T>

## ✅ Responsibility

* Represents criteria query object

---

## 🔷 Methods

| Method    | Use                |
| --------- | ------------------ |
| select()  | Select entity      |
| where()   | Add conditions     |
| from()    | Define root entity |
| orderBy() | Sorting            |

---

# 🔷 8️⃣ Metamodel Interfaces (Advanced)

| Interface         | Responsibility         |
| ----------------- | ---------------------- |
| Metamodel         | Access entity metadata |
| EntityType        | Represents entity type |
| SingularAttribute | Represents attribute   |

(Used in advanced criteria queries)

---

# 🟢 SUMMARY VIEW

| Interface            | Main Responsibility         |
| -------------------- | --------------------------- |
| EntityManagerFactory | Creates EntityManager       |
| EntityManager        | Core JPA operations         |
| EntityTransaction    | Manual transaction control  |
| Query                | Execute JPQL/native queries |
| TypedQuery           | Type-safe query             |
| CriteriaBuilder      | Build dynamic queries       |
| CriteriaQuery        | Represent criteria query    |
| Metamodel            | Access entity metadata      |

---

# 🎯 MOST IMPORTANT (Must Remember)

If interviewer asks “Core JPA Interfaces” → say:

✔ EntityManagerFactory
✔ EntityManager
✔ EntityTransaction
✔ Query
✔ TypedQuery
✔ CriteriaBuilder
✔ CriteriaQuery

---

Perfect 👌
Now we’ll do the same structured breakdown for **SPRING DATA JPA interfaces**.

You’ll get:

1️⃣ List of Spring Data JPA interfaces
2️⃣ Their responsibility (use)
3️⃣ Inheritance diagram
4️⃣ Separate method table for each interface

We are focusing on **org.springframework.data.jpa.repository** and related repository hierarchy.

---

# 2️⃣ 🟢 SPRING DATA JPA — CORE INTERFACES

Spring Data JPA provides abstraction over JPA (EntityManager).

---

# 🔷 MAIN INHERITANCE DIAGRAM

# 🟢 COMPLETE RELATIONSHIP DIAGRAM

```id="q0v3xk"
Repository<T, ID>
        ↑
CrudRepository<T, ID>
        ↑
PagingAndSortingRepository<T, ID>
        ↑
JpaRepository<T, ID>

JpaSpecificationExecutor<T>  (Parallel Extension)

JpaRepository  +  JpaSpecificationExecutor
            ↓
      YourRepository

⚠ Note: `JpaSpecificationExecutor` does NOT extend JpaRepository directly.
It is implemented separately along with JpaRepository.

🟢 Simple Rule to Remember

If it extends Repository, it is part of hierarchy.
If it does NOT extend Repository, it is a support interface.

---
```

# 🔷 1️⃣ Repository<T, ID>

## ✅ Responsibility

• Enables Spring to create a proxy implementation at runtime.
• Acts as a marker interface in Spring Data.
• Defines domain type and primary key type using generics.
• Does not provide CRUD methods directly.
• Serves as the root of the repository hierarchy.

## 🔷 Methods

❌ No methods
Just:

```java
public interface Repository<T, ID>
```

---

# 🔷 2️⃣ CrudRepository<T, ID>

## ✅ Responsibility

• Provides basic Create, Read, Update, Delete operations.
• Defines methods like save(), findById(), findAll(), and delete().
• Works with any entity and primary key type.
• Returns Optional for single entity fetch operations.
• Suitable for simple applications requiring basic database operations.

---

## 🔷 Methods of CrudRepository

| Method                    | Return Type | Use                   |
| ------------------------- | ----------- | --------------------- |
| save(S entity)            | S           | Insert or update      |
| saveAll(Iterable<S>)      | Iterable<S> | Bulk save             |
| findById(ID id)           | Optional<T> | Find by primary key   |
| existsById(ID id)         | boolean     | Check existence       |
| findAll()                 | Iterable<T> | Fetch all             |
| findAllById(Iterable<ID>) | Iterable<T> | Fetch multiple IDs    |
| count()                   | long        | Count records         |
| delete(T entity)          | void        | Delete entity         |
| deleteById(ID id)         | void        | Delete by ID          |
| deleteAll()               | void        | Delete all            |
| deleteAll(Iterable<T>)    | void        | Delete given entities |

---

# 🔷 3️⃣ PagingAndSortingRepository<T, ID>

## ✅ Responsibility

• Extends CrudRepository.
• Adds pagination support using Pageable.
• Adds sorting support using Sort.
• Allows fetching data in smaller pages instead of full lists.
• Useful when working with large datasets.

Extends: `CrudRepository`
---

## 🔷 Methods of PagingAndSortingRepository

| Method                     | Return Type | Use        |
| -------------------------- | ----------- | ---------- |
| findAll(Sort sort)         | Iterable<T> | Sorting    |
| findAll(Pageable pageable) | Page<T>     | Pagination |

---

# 🔷 4️⃣ JpaRepository<T, ID>

## ✅ Responsibility
• Extends PagingAndSortingRepository.
• Adds JPA-specific features like flush() and batch operations.
• Returns List instead of Iterable for better usability.
• Integrates directly with JPA EntityManager.
• Most commonly used repository in Spring Boot projects.

Extends: `PagingAndSortingRepository`

---

## 🔷 Methods of JpaRepository

### 🔹 Flush Control

| Method               | Use                      |
| -------------------- | ------------------------ |
| flush()              | Force sync with DB       |
| saveAndFlush(entity) | Save + flush immediately |

---

### 🔹 Batch Operations

| Method                    | Use                |
| ------------------------- | ------------------ |
| deleteAllInBatch()        | Bulk delete        |
| deleteAllByIdInBatch(ids) | Bulk delete by IDs |
| deleteInBatch(entities)   | Bulk delete        |

---

### 🔹 Lazy Reference

| Method                  | Use                  |
| ----------------------- | -------------------- |
| getReferenceById(ID id) | Lazy proxy reference |

---

### 🔹 Query by Example

| Method              | Use                      |
| ------------------- | ------------------------ |
| findAll(Example<S>) | Query by example         |
| findOne(Example<S>) | Single result by example |

---

# 🔷 5️⃣ JpaSpecificationExecutor<T>

## ✅ Responsibility

• Supports dynamic query building using Specification.
• Allows combining multiple conditions at runtime.
• Works well for complex filtering scenarios.
• Supports pagination and sorting with specifications.
• Used mainly in advanced search features.

Does NOT extend JpaRepository
It is implemented alongside it.

Example:

```java
public interface UserRepository 
      extends JpaRepository<User, Long>, 
              JpaSpecificationExecutor<User>
```

---

## 🔷 Methods of JpaSpecificationExecutor

| Method                              | Return Type | Use                 |
| ----------------------------------- | ----------- | ------------------- |
| findAll(Specification<T>)           | List<T>     | Dynamic filtering   |
| findAll(Specification<T>, Pageable) | Page<T>     | Filter + pagination |
| findOne(Specification<T>)           | Optional<T> | Single result       |
| count(Specification<T>)             | long        | Count with filter   |
| exists(Specification<T>)            | boolean     | Check existence     |

---

# 🔷 6️⃣ Supporting Interfaces (Used by Repositories)

--- (we will learn this in paging and sorting chapter ) 

## 🔹 Pageable

### Responsibility

• Represents pagination configuration.
• Stores page number and page size.
• Can include sorting information.
• Passed as a parameter to repository methods.
• Converts internally to SQL LIMIT and OFFSET.

### Important Methods

| Method          | Use          |
| --------------- | ------------ |
| getPageNumber() | Page index   |
| getPageSize()   | Page size    |
| getSort()       | Sorting info |
| next()          | Next page    |

---

## 🔹 Page<T>

### Responsibility

• Represents paginated result data.
• Contains both content and pagination metadata.
• Provides methods like getContent() and getTotalPages().
• Helps check next/previous page availability.
• Returned by repository methods using pagination.

### Methods

| Method             | Use              |
| ------------------ | ---------------- |
| getContent()       | List of records  |
| getTotalElements() | Total count      |
| getTotalPages()    | Total pages      |
| getNumber()        | Current page     |
| hasNext()          | Next page exists |

---

## 🔹 Sort

### Responsibility

• Represents sorting configuration.
• Supports ascending and descending order.
• Can sort by one or multiple fields.
• Can be used independently or inside Pageable.
• Converts internally to SQL ORDER BY.

### Methods

| Method           | Use         |
| ---------------- | ----------- |
| by(String field) | Create sort |
| ascending()      | ASC order   |
| descending()     | DESC order  |

```

---

# 🟢 INTERNAL WORKING FLOW

When you call:

```java
userRepository.save(user);
```

Internally:

```
JpaRepository
   ↓
Uses EntityManager
   ↓
If ID null → persist()
If ID not null → merge()
   ↓
Flush at commit
```

---

# 🟢 SUMMARY TABLE

| Interface                  | Responsibility         |
| -------------------------- | ---------------------- |
| Repository                 | Marker interface       |
| CrudRepository             | Basic CRUD             |
| PagingAndSortingRepository | Pagination & sorting   |
| JpaRepository              | JPA specific features  |
| JpaSpecificationExecutor   | Dynamic filtering      |
| Pageable                   | Pagination abstraction |
| Page                       | Paginated result       |
| Sort                       | Sorting abstraction    |

---

# 🎯 WHAT INTERVIEWER EXPECTS YOU TO KNOW

If asked “Spring Data JPA Interfaces”:

✔ Repository hierarchy
✔ Difference between CrudRepository & JpaRepository
✔ What JpaRepository adds
✔ How save() works internally
✔ Role of JpaSpecificationExecutor

---
| Interface                  | Type                | Extends                    | Purpose                               | Key Methods                              | Pagination Support | Sorting Support | JPA Specific Features | Dynamic Query Support | Used In                | Example Usage                                      |
| -------------------------- | ------------------- | -------------------------- | ------------------------------------- | ---------------------------------------- | ------------------ | --------------- | --------------------- | --------------------- | ---------------------- | -------------------------------------------------- |
| Repository                 | Marker              | —                          | Enable Spring proxy creation          | —                                        | ❌                  | ❌               | ❌                     | ❌                     | Parent interface       | `interface UserRepo extends Repository<User,Long>` |
| CrudRepository             | Core Repository     | Repository                 | Basic CRUD operations                 | save(), findById(), delete(), findAll()  | ❌                  | ❌               | ❌                     | ❌                     | Simple CRUD apps       | `extends CrudRepository<User,Long>`                |
| PagingAndSortingRepository | Extended Repository | CrudRepository             | Adds pagination & sorting             | findAll(Pageable), findAll(Sort)         | ✅                  | ✅               | ❌                     | ❌                     | When pagination needed | `extends PagingAndSortingRepository<User,Long>`    |
| JpaRepository              | Advanced Repository | PagingAndSortingRepository | JPA specific features + full CRUD     | flush(), saveAndFlush(), deleteInBatch() | ✅                  | ✅               | ✅                     | ❌                     | Most production apps   | `extends JpaRepository<User,Long>`                 |
| JpaSpecificationExecutor   | Parallel Extension  | —                          | Dynamic filtering using Specification | findAll(Specification), count()          | ✅                  | ✅               | ❌                     | ✅                     | Complex filters        | `extends JpaSpecificationExecutor<User>`           |
| Pageable                   | Utility Interface   | —                          | Pagination configuration              | getPageNumber(), getPageSize()           | ✅                  | ✅ (with Sort)   | ❌                     | ❌                     | Method parameter       | `PageRequest.of(0,5)`                              |
| Page                       | Result Wrapper      | —                          | Paginated result container            | getContent(), getTotalPages()            | —                  | —               | ❌                     | ❌                     | Return type            | `Page<User>`                                       |
| Sort                       | Utility Interface   | —                          | Sorting configuration                 | by(), ascending(), descending()          | ❌                  | ✅               | ❌                     | ❌                     | Parameter in query     | `Sort.by("name")`                                  |


