# Docker Multi-Stage Builds (Spring Boot)

### Definition

> **Take the complete Spring Boot project, compile/package it, generate the JAR, then create a lightweight runtime image containing only the JAR (and Java Runtime).**

---

# 1. Why Multi-Stage?

* Docker builds the JAR automatically.
* No need to run `mvn clean package`.
* Final image contains only **JRE + app.jar**.
* No Maven, source code or build tools.
* Cleaner, more secure and production-ready.

---

# 2. One-Stage vs Multi-Stage

## One-Stage

```text
You
 │
mvn clean package
 │
app.jar
 │
docker build
 │
Image
 │
docker run
 │
Container
```

Commands

```bash
mvn clean package
docker build -t my-app .
docker run -p 8080:8080 my-app
```

---

## Multi-Stage

```text
You
 │
docker build
 │
Docker
 │
mvn clean package
 │
app.jar
 │
Runtime Image
 │
docker run
 │
Container
```

Commands

```bash
docker build -t my-app .
docker run -p 8080:8080 my-app
```

---

# 3. Difference

| One Stage               | Multi Stage              |
| ----------------------- | ------------------------ |
| You create JAR          | Docker creates JAR       |
| Maven runs locally      | Maven runs inside Docker |
| Manual build            | Automatic build          |
| Final image = JRE + JAR | Final image = JRE + JAR  |
| Good for learning       | Industry standard        |

> **If One-Stage already uses a JRE image, the final image size is nearly the same. Multi-stage mainly automates the build and keeps the process clean.**

---

# 4. Internal Working

```text
Spring Boot Project
        │
docker build
        │
Builder Stage
(Maven + JDK)
        │
RUN mvn clean package
        │
Creates app.jar
        │
COPY --from=builder
        ▼
Runtime Stage
(JRE)
        │
Final Image
        │
docker run
        ▼
Application Container
```

---

# 5. Important Concepts

### Builder Stage

* Uses Maven + JDK.
* Compiles the project.
* Creates the JAR.
* Temporary stage.

### Runtime Stage

* Uses lightweight JRE.
* Copies only the JAR.
* Produces the final runnable image.

### Temporary Build Containers

* Created automatically during `docker build`.
* Execute `RUN`, `COPY`, etc.
* Deleted after the build step completes.

### Images vs Containers

**Image stores**

* app.jar
* JRE
* Configuration
* Libraries

**Container stores**

* Logs
* Runtime files
* Temporary files
* Database changes (unless external DB/volume)

---

# 6. Important Dockerfile Instructions

```dockerfile
FROM maven:3.9.11-eclipse-temurin-21 AS builder
```

* Builder stage (Maven + JDK)

```dockerfile
WORKDIR /app
```

* Create & move to `/app`

```dockerfile
COPY . .
```

* Copy complete project

```dockerfile
RUN mvn clean package -DskipTests
```

* Docker creates the JAR

```dockerfile
FROM eclipse-temurin:21-jre
```

* Runtime stage

```dockerfile
COPY --from=builder /app/target/DockerApp-0.0.1-SNAPSHOT.jar app.jar
```

* Copy only the generated JAR

```dockerfile
EXPOSE 8080
```

* Documents the container port

```dockerfile
ENTRYPOINT ["java","-jar","app.jar"]
```

* Starts the application

---

# 7. Complete Dockerfile

```dockerfile
# ---------- Stage 1 : Build ----------
FROM maven:3.9.11-eclipse-temurin-21 AS builder

WORKDIR /app

COPY . .

RUN mvn clean package -DskipTests


# ---------- Stage 2 : Runtime ----------
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=builder /app/target/DockerApp-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","app.jar"]
```

---

# 8. Golden Rules

* Multiple `FROM` ⇒ Multiple stages.
* Builder Stage = **Compile & Package**.
* Runtime Stage = **Run Only**.
* Docker builds the JAR automatically.
* `COPY --from=builder` copies only required files.
* Builder stage is temporary.
* `docker build` uses temporary build containers.
* `docker run` creates the real application container.
* Final image contains **JRE + app.jar**.
* Prefer Multi-Stage Builds for production.

---

## Memory Trick

> **One-Stage:** *You build the JAR, Docker packages it.*

> **Multi-Stage:** *Give Docker the whole project. Docker builds the JAR, keeps only the JAR + JRE, and creates the final runnable image.*
