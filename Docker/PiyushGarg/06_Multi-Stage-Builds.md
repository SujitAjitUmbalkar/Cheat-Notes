# Multi-Stage Builds (Docker) — Complete Notes

A **multi-stage build** allows you to use **multiple `FROM` statements** in one Dockerfile.

Each stage has its own purpose.

For a Spring Boot project:

* **Stage 1:** Build the JAR using Maven.
* **Stage 2:** Copy only the JAR into a lightweight runtime image.

This makes the final image much smaller, cleaner, and more secure.

---

# Why do we need Multi-Stage Builds?

Without multi-stage builds, if you build inside Docker, the final image contains:

* Maven
* Source code (`src`)
* `.git`
* Build cache
* `pom.xml`
* Compilers
* JDK tools
* Final JAR

```text
Final Image
│
├── Maven
├── Source Code
├── pom.xml
├── Target
├── JDK
└── app.jar
```

Image becomes large.

---

With Multi-Stage Build

```text
Builder Stage
│
├── Maven
├── Source Code
├── JDK
└── Builds app.jar
        │
        ▼
Runtime Stage
│
├── JRE
└── app.jar
```

Only the JAR is copied.

Everything else is discarded.

---

# Advantages

* Smaller image
* Faster download
* Faster deployment
* Better security
* No source code inside image
* No Maven inside image
* No unnecessary build tools
* Industry standard

---

# How it Works

## Stage 1

```dockerfile
FROM maven:3.9.11-eclipse-temurin-21 AS builder
```

Creates a temporary container.

Purpose:

Build the project.

---

## Stage 2

```dockerfile
FROM eclipse-temurin:21-jre
```

Creates another fresh image.

Purpose:

Run the application.

---

Docker copies only the required files.

---

# Workflow

```text
Spring Boot Project
│
├── src
├── pom.xml
└── Dockerfile
        │
        ▼
Stage 1
(Maven Image)
        │
        ▼
mvn clean package
        │
Creates
target/app.jar
        │
        ▼
Stage 2
(JRE Image)
        │
Copies only
app.jar
        │
        ▼
Final Docker Image
```

---

# Complete Dockerfile

```dockerfile
# ----------------------------
# Stage 1 : Build the Project
# ----------------------------

# Maven image with JDK 21
FROM maven:3.9.11-eclipse-temurin-21 AS builder

# Working directory
WORKDIR /app

# Copy entire project
COPY . .

# Build the project and create JAR
RUN mvn clean package -DskipTests

# ----------------------------
# Stage 2 : Run the Project
# ----------------------------

# Lightweight Java Runtime
FROM eclipse-temurin:21-jre

# Working directory
WORKDIR /app

# Copy only the generated JAR from Stage 1
COPY --from=builder /app/target/DockerApp-0.0.1-SNAPSHOT.jar app.jar

# Application listens on port 8080
EXPOSE 8080

# Start Spring Boot
ENTRYPOINT ["java","-jar","app.jar"]
```

---

# Understanding Every Command

## Stage 1

```dockerfile
FROM maven:3.9.11-eclipse-temurin-21 AS builder
```

Uses Maven + JDK.

Names this stage **builder**.

---

```dockerfile
WORKDIR /app
```

Creates

```
/app
```

and moves into it.

---

```dockerfile
COPY . .
```

Copies entire Spring Boot project.

```
Local Project
│
├── src
├── pom.xml
├── mvnw
└── target
```

↓

```
Container
/app
```

---

```dockerfile
RUN mvn clean package -DskipTests
```

Builds

```
target/app.jar
```

inside the builder container.

---

## Stage 2

```dockerfile
FROM eclipse-temurin:21-jre
```

Fresh image.

Nothing from Stage 1 exists automatically.

---

```dockerfile
WORKDIR /app
```

Creates

```
/app
```

---

```dockerfile
COPY --from=builder /app/target/DockerApp-0.0.1-SNAPSHOT.jar app.jar
```

This is the most important command.

Meaning:

```
Copy

FROM

Builder Stage

↓

/app/target/app.jar

↓

Current Stage

↓

/app/app.jar
```

Only one file is copied.

Everything else is discarded.

---

```dockerfile
EXPOSE 8080
```

Documents the application's container port.

---

```dockerfile
ENTRYPOINT ["java","-jar","app.jar"]
```

Runs Spring Boot.

---

# Visual Diagram

```text
                Stage 1
 ┌────────────────────────────────┐
 │ Maven + JDK                    │
 │                                │
 │ Source Code                    │
 │ pom.xml                        │
 │ mvn clean package              │
 │                                │
 │ target/app.jar                 │
 └──────────────┬─────────────────┘
                │
                │ COPY --from=builder
                ▼
 ┌────────────────────────────────┐
 │ Stage 2                        │
 │                                │
 │ JRE                            │
 │ app.jar                        │
 │                                │
 │ java -jar app.jar              │
 └────────────────────────────────┘
```

---

# Image Size Comparison

| Dockerfile               | Final Image |
| ------------------------ | ----------: |
| JDK + JAR                |       Large |
| JRE + JAR                |      Medium |
| Multi-stage + JRE        |       Small |
| Multi-stage + Distroless |    Smallest |

---

# When Should You Use It?

| Situation              | Multi-Stage? |
| ---------------------- | ------------ |
| Learning Docker basics | Optional     |
| Spring Boot production | ✅ Yes        |
| CI/CD pipelines        | ✅ Yes        |
| Docker Hub publishing  | ✅ Yes        |
| Microservices          | ✅ Yes        |

---

# Golden Rules

* Use **multiple `FROM`** instructions for separate build and runtime stages.
* Build the application in the **builder stage** (Maven + JDK).
* Run the application in a **runtime stage** (JRE only).
* Copy **only the final JAR** using `COPY --from=<stage>`.
* Do **not** include source code, Maven, or build tools in the final image.
* Prefer **JRE** over **JDK** for runtime.
* Use **`-DskipTests`** during image builds if tests are already executed in CI/CD.
* Name build stages (e.g., `AS builder`) for readability and maintainability.
* Keep the runtime image as small as possible for faster pulls and better security.

## Memory Trick

```text
Stage 1 → Build
      │
      ▼
Creates JAR
      │
COPY --from=builder
      ▼
Stage 2 → Run
      │
      ▼
Small, Clean, Secure Image
```

**One-line summary:**
**Build everything in Stage 1, copy only what you need into Stage 2.**
