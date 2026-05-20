# Testcontainers

Testcontainers is a Java testing library that runs real services like:

* MySQL
* PostgreSQL
* MongoDB
* Redis
* Kafka

inside Docker containers during testing.

It is mainly used in Spring Boot integration testing to provide a real production-like environment.

---

# Why Use Testcontainers?

Normally developers use:

```text id="81m4zt"
H2 in-memory database
```

for testing.

But H2 may behave differently from real MySQL/PostgreSQL.

Testcontainers solves this problem by starting an actual database inside Docker.

Benefits:

* real database behavior
* production-like testing
* isolated clean environment
* automatic container removal
* reliable integration testing

---

# What Testcontainers Requires

## 1. Java Project

Usually used with:

* Spring Boot
* JUnit

---

## 2. Docker Installation

Testcontainers works using Docker containers.

Install:

[Docker Desktop](https://www.docker.com/products/docker-desktop/?utm_source=chatgpt.com)

After installation, start Docker Desktop.

Check Docker:

```bash id="j2xv61"
docker --version
```

---

## 3. Internet Connection (First Time)

Docker downloads database images first time.

Example:

```text id="n1dfwy"
mysql:8.0
postgres:16
```

After download, images stay cached locally.

---

# Testcontainers Setup

# Step 1: Add Dependencies

## Maven Dependencies

```xml id="9bsl2o"
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>mysql</artifactId>
    <scope>test</scope>
</dependency>
```

---

# Step 2: Create Test Class

```java id="trm9m0"
@SpringBootTest
@Testcontainers
class UserRepositoryTest {
}
```

---

# Step 3: Create Database Container

```java id="e7k0xo"
@Container
static MySQLContainer<?> mysql =
        new MySQLContainer<>("mysql:8.0")
                .withDatabaseName("testdb")
                .withUsername("root")
                .withPassword("root");
```

This starts a real MySQL database container.

---

# Step 4: Connect Spring Boot to Container

```java id="a9b8yq"
@DynamicPropertySource
static void configureProperties(
        DynamicPropertyRegistry registry)
{
    registry.add(
            "spring.datasource.url",
            mysql::getJdbcUrl
    );

    registry.add(
            "spring.datasource.username",
            mysql::getUsername
    );

    registry.add(
            "spring.datasource.password",
            mysql::getPassword
    );
}
```

This dynamically gives Spring Boot the database connection values.

---

# Step 5: Write Test Cases

```java id="b0kn9z"
@Test
void testSaveUser()
{
    User user = new User();
    user.setName("Sujeet");

    userRepository.save(user);

    assertThat(userRepository.findAll())
            .hasSize(1);
}
```

---
---

# Important Annotations

| Annotation               | Purpose                                |
| ------------------------ | -------------------------------------- |
| `@Testcontainers`        | Enables Testcontainers support         |
| `@Container`             | Marks Docker container                 |
| `@SpringBootTest`        | Loads full Spring Boot application     |
| `@DynamicPropertySource` | Dynamically sets datasource properties |
| `@Test`                  | Defines test method                    |

---

# How It Works Internally

```text id="6a5mp3"
Test starts
    ↓
Docker container starts
    ↓
MySQL starts inside container
    ↓
Spring Boot connects to it
    ↓
Tests execute
    ↓
Container stops automatically
```

---

# Advantages

| Advantage              | Meaning                 |
| ---------------------- | ----------------------- |
| Real database          | Actual MySQL/PostgreSQL |
| Fresh DB every test    | Clean environment       |
| Automatic cleanup      | No manual deletion      |
| Reliable testing       | Closer to production    |
| Supports many services | MySQL, Redis, Kafka etc |

---

# Disadvantages

| Disadvantage    | Meaning                |
| --------------- | ---------------------- |
| Docker required | Must install Docker    |
| Slower than H2  | Container startup time |
| More RAM usage  | Docker consumes memory |



# Simple Understanding

## H2

```text id="jlwm0x"
Database simulation
```

## Testcontainers

```text id="jlwm0y"
Real database testing
```
