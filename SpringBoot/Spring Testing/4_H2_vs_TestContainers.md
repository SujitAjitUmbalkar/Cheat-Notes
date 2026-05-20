When testing a Spring Boot application, database choice matters because different testing approaches give different levels of accuracy and speed.

# 1. Why configure a separate test database?

We usually do not use the real production database for tests because:

* tests may delete or modify real data
* tests should run independently
* repeated test execution should not affect actual application data
* developers need isolated environments

Example:

```text
Production DB  -> real users data
Test DB        -> temporary testing data
```

So developers create:

```properties
application-test.properties
```

with separate DB configuration.

---

# 2. Why not always use H2?

H2 Database Engine is an in-memory database mostly used for fast testing.

Example:

```properties
spring.datasource.url=jdbc:h2:mem:testdb
```

Advantages:

* very fast
* lightweight
* no installation needed
* good for unit/integration testing

But H2 is NOT exactly same as real databases like:

* MySQL
* PostgreSQL

Problems with H2:

| Problem                      | Example                              |
| ---------------------------- | ------------------------------------ |
| SQL syntax differences       | Query works in H2 but fails in MySQL |
| Data type differences        | JSON, ENUM handling differs          |
| Dialect differences          | Native queries may fail              |
| Transaction behavior differs | Locking/isolation changes            |
| Case sensitivity differences | Table names behave differently       |

So sometimes:

```text
Tests pass in H2
But application fails in production
```

---

# 3. What is Testcontainers?

Testcontainers is a library that starts real databases inside Docker containers during tests.

Example:

```text
JUnit Test
   ↓
Testcontainers starts MySQL container
   ↓
Spring Boot connects to real MySQL
   ↓
Tests run
   ↓
Container removed automatically
```

---

# 4. Why use Testcontainers?

Because it gives a REAL database environment.

Example:

Instead of fake/in-memory H2:

```text
H2 simulation
```

you get:

```text
Actual MySQL/PostgreSQL running in Docker
```

Benefits:

| Benefit                    | Meaning                           |
| -------------------------- | --------------------------------- |
| Production-like testing    | Same DB as production             |
| Real SQL behavior          | Native queries validated          |
| Automatic cleanup          | Fresh DB every test               |
| Isolation                  | No shared test pollution          |
| CI/CD friendly             | Works in pipelines                |
| Reliable integration tests | Less “works on my machine” issues |

---

# 5. Example Comparison

## Using H2

```text
Fast
Simple
Less accurate
```

## Using Testcontainers

```text
Slightly slower
More setup
Highly accurate
Production-like
```

---

# 6. Simple Real-world Analogy

## H2

Like practicing car driving in a simulator.

## Testcontainers

Like driving the actual car on a real road.

Both are useful, but real-road testing catches real problems.

---

# 7. Example Testcontainers Code

```java id="3wr8k"
@Testcontainers
@SpringBootTest
class UserRepositoryTest {

    @Container
    static MySQLContainer<?> mysql =
            new MySQLContainer<>("mysql:8.0");

}
```

This automatically starts a real MySQL database in Docker for testing.

---

# 8. When to use what?

| Situation                         | Recommended    |
| --------------------------------- | -------------- |
| Simple repository tests           | H2             |
| Fast local testing                | H2             |
| Real DB compatibility testing     | Testcontainers |
| Native SQL queries                | Testcontainers |
| Production-like integration tests | Testcontainers |
| CI/CD reliable testing            | Testcontainers |

Most professional projects use:

```text
Unit Tests -> Mock/H2
Integration Tests -> Testcontainers
```
