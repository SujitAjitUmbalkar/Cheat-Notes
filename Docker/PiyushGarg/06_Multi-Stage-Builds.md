# Docker Multi-Stage Builds (Spring Boot) – Cheat Notes

---

# 1. What is a Multi-Stage Build?

* A Dockerfile with **multiple `FROM`** instructions.
* Each `FROM` creates a separate **build stage**.
* Usually:

  * **Stage 1 → Build** the application.
  * **Stage 2 → Run** the application.

---

# 2. Why use Multi-Stage Builds?

* Docker creates the JAR automatically.
* No need to run `mvn clean package` manually.
* Final image contains only what is required to run.
* No Maven, source code or build tools in the final image.
* Smaller, cleaner and more secure images.
* Industry standard.

---

# 3. One-Stage vs Multi-Stage

## One-Stage

```text
You
 │
mvn clean package
 │
Creates app.jar
 │
docker build
 │
Copies app.jar
 │
docker run
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
Creates app.jar
 │
Copies app.jar
 │
docker run
```

Commands

```bash
docker build -t my-app .

docker run -p 8080:8080 my-app
```

---

# 4. Difference

| One Stage                        | Multi Stage              |
| -------------------------------- | ------------------------ |
| You create JAR                   | Docker creates JAR       |
| Run Maven manually               | Docker runs Maven        |
| Simpler                          | Industry standard        |
| Final image = JRE + JAR          | Final image = JRE + JAR  |
| Requires Maven installed locally | Docker handles the build |

> **If your one-stage Dockerfile already uses a JRE image, the final image size is usually almost the same. The biggest benefit is automation and a cleaner build process.**

---

# 5. How Docker Creates the JAR

```text
Spring Boot Project
        │
docker build
        │
Builder Stage
        │
RUN mvn clean package
        │
Docker creates app.jar
```

The JAR is created **inside the builder stage**, not on your local machine.

---

# 6. What Happens Internally?

```text
Dockerfile
      │
      ▼
Builder Image
      │
Temporary Build Container
      │
RUN mvn clean package
      │
Creates app.jar
      │
Save Image Layer
      │
Delete Temporary Container
```

Then

```text
Runtime Image
      │
Temporary Build Container
      │
COPY app.jar
      │
Save Final Image
      │
Delete Temporary Container
```

---

# 7. Does Docker Create Two Images?

During build:

```text
Builder Stage
        │
        ▼
Runtime Stage
```

✔ Internally, yes.

After build:

```text
Only Runtime Image is kept
```

The builder stage is only used to produce the final image unless you explicitly build or tag it.

---

# 8. Does Builder Stage Create a Container?

Yes.

Docker creates **temporary build containers** to execute instructions like:

```dockerfile
RUN
COPY
WORKDIR
```

After the step finishes:

* Save changes to the image layer.
* Delete the temporary build container.

Only `docker run` creates the container that runs your application.

---

# 9. Does the Image Store Data?

### Image stores

* app.jar
* Configuration files
* Static resources
* Libraries

### Image does NOT store

* Logs
* Runtime files
* Database records
* Uploaded files
* Temporary files

Those belong to the **container**.

---

# 10. Why Isn't the Final Image Much Smaller?

If both Dockerfiles use:

```dockerfile
FROM eclipse-temurin:21-jre
```

then both final images contain:

```text
JRE
+
app.jar
```

So the image size is almost the same.

Multi-stage mainly automates the build and keeps the final image clean.

---

# 11. Multi-Stage Workflow

```text
Spring Boot Project
        │
docker build
        │
Stage 1
(Maven + JDK)
        │
Creates app.jar
        │
COPY --from=builder
        ▼
Stage 2
(JRE)
        │
Final Image
        │
docker run
        ▼
Container
```

---

# 12. Important Dockerfile Instructions

```dockerfile
FROM maven:3.9.11-eclipse-temurin-21 AS builder
```

* Builder stage
* Maven + JDK

---

```dockerfile
WORKDIR /app
```

* Creates `/app`
* Switches to `/app`

---

```dockerfile
COPY . .
```

* Copies complete Spring Boot project.

---

```dockerfile
RUN mvn clean package -DskipTests
```

* Docker builds the JAR.

---

```dockerfile
FROM eclipse-temurin:21-jre
```

* Runtime stage.

---

```dockerfile
COPY --from=builder /app/target/DockerApp-0.0.1-SNAPSHOT.jar app.jar
```

* Copy only the generated JAR.

---

```dockerfile
EXPOSE 8080
```

* Documents the container port.

---

```dockerfile
ENTRYPOINT ["java","-jar","app.jar"]
```

* Starts the Spring Boot application.

---

# 13. Complete Dockerfile

```dockerfile
# ============================
# Stage 1 : Build Application
# ============================

# Maven + JDK image
FROM maven:3.9.11-eclipse-temurin-21 AS builder

# Working directory
WORKDIR /app

# Copy complete project
COPY . .

# Docker builds the JAR
RUN mvn clean package -DskipTests

# ============================
# Stage 2 : Runtime Image
# ============================

# Lightweight JRE image
FROM eclipse-temurin:21-jre

# Working directory
WORKDIR /app

# Copy only the JAR from builder stage
COPY --from=builder /app/target/DockerApp-0.0.1-SNAPSHOT.jar app.jar

# Application listens on 8080
EXPOSE 8080

# Start Spring Boot
ENTRYPOINT ["java","-jar","app.jar"]
```

---

# 14. Golden Rules

* One Dockerfile can have **multiple `FROM`** instructions.
* Each `FROM` starts a **new stage**.
* **Builder stage** → Compile/package the application.
* **Runtime stage** → Run the application.
* Docker creates the **JAR automatically** in the builder stage.
* `COPY --from=builder` copies only the required files.
* The **builder stage is temporary** unless explicitly kept.
* `docker build` creates images using temporary build containers.
* `docker run` creates the actual application container.
* Images store **application files**; containers store **runtime changes**.
* Use a **JRE** in the runtime stage instead of a **JDK**.
* Multi-stage builds are the preferred approach for production applications.

---

## Memory Flow

```text
Spring Boot Project
        │
docker build
        │
Builder Stage (Maven + JDK)
        │
Docker creates app.jar
        │
COPY --from=builder
        ▼
Runtime Stage (JRE)
        │
Final Image
        │
docker run
        ▼
Application Container
```

### One-Line Memory Trick

> **One-stage:** *You build the JAR, Docker packages it.*
> **Multi-stage:** *Docker builds the JAR, then packages only the JAR into the final runtime image.*
