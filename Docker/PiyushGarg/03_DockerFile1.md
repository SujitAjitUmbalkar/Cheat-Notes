# 🐳 Docker Concept Cheat Notes (From Dockerfile Discussion)

## Dockerfile Location

* Create **Dockerfile in the root of Spring Boot project**.
* Location: same level as `pom.xml` / `build.gradle`.
* Reason: Docker build context can access project files easily.

Example:

```text
spring-app
│
├── src
├── pom.xml
├── Dockerfile
└── target
```

---

# Dockerfile Creation Flow

* Dockerfile can be created **before generating JAR**.
* But `docker build` requires all files mentioned in `COPY` to exist.
* For Spring Boot, create JAR before building Docker image.

Flow:

```
Code → mvn package → JAR → docker build → Image → docker run → Container
```

---

# Dockerfile Extension

* Dockerfile has **no extension**.
* Correct:

```
Dockerfile
```

* Wrong:

```
Dockerfile.txt
```

---

# Image Selection Concept

* Developer does not select image based on host OS (Windows/Linux).
* Select image based on **application stack/runtime requirement**.

Example:

```
Spring Boot → Java image
Node App → Node image
Python App → Python image
```

---

# Base Image (`FROM`)

* `FROM` decides the starting environment of your image.
* It provides required OS + runtime environment.

Example:

```dockerfile
FROM eclipse-temurin:21-jre
```

Means:

```
Linux environment
+
Java 21
```

---

# Eclipse Temurin Image Concept

* Eclipse Temurin is **not an operating system**.
* It is a Java runtime image built on top of a base OS (usually Linux).
* It provides Java so your Spring Boot application can run.

Concept:

```
Linux
 ↓
Java Runtime (Eclipse Temurin)
 ↓
Spring Boot Application
```

---

# Why Linux Comes in My Image?

* Your Windows laptop does not decide image contents.
* `FROM` downloads the required base image from Docker Hub.
* If the base image is Linux-based, your final image becomes Linux-based.

Example:

```
Windows Laptop
      |
docker build
      |
Download Linux + Java image
      |
Add Spring Boot JAR
      |
Final Image
```

---

# Image vs Container

* Image = Blueprint/template of application environment.
* Container = Running instance created from an image.

Example:

```
Image
 |
 ├── Container 1
 ├── Container 2
 └── Container 3
```

---

# Container OS Concept

* Container OS depends on the **base image**, not the developer machine.
* Linux image creates a Linux container.

Example:

```
FROM ubuntu
        ↓
Linux Container
```

```
FROM eclipse-temurin
        ↓
Linux + Java Container
```

---

# Running Linux Containers

### On Linux Host:

```
Linux Machine
   |
Docker Engine
   |
Container
```

* Container directly uses Linux kernel.

### On Windows:

```
Windows
 |
Docker Desktop
 |
WSL2 Linux VM
 |
Linux Container
```

* Docker provides Linux environment using WSL2.

### On macOS:

```
macOS
 |
Docker Desktop
 |
Linux VM
 |
Container
```

---

# Sharing Docker Image

* Developer creates an image and shares it through Docker Hub/registry.
* User only needs Docker installed to run that image.
* User does not need the same OS, Java version, or project setup.

---

# How Docker Image Runs on Another Machine?

Example:

```
Developer:
Windows + Spring Boot
        |
        ↓
Creates Linux + Java Image
        |
        ↓
Shares Image
        |
User:
Windows/Linux/macOS
        |
        ↓
docker run
        |
Container starts
```

---

# COPY Concept

* `COPY` moves files from your local project into the image.
* Node projects may copy individual files.

Example:

```dockerfile
COPY index.js /app/
```

* Spring Boot usually copies the generated JAR.

Example:

```dockerfile
COPY target/app.jar app.jar
```

---

# Why Spring Boot Copies JAR?

* Java source code is converted into a JAR using Maven.
* JAR already contains compiled classes and dependencies.
* Docker only needs the JAR to run the application.

---

# Docker Build Context

Command:

```bash
docker build -t app .
```

* `.` means current folder is the build context.
* Docker can access files inside this folder.

---

# Important Dockerfile Instructions

* `FROM` → Selects base image.
* `WORKDIR` → Sets working directory inside container.
* `COPY` → Copies files into image.
* `RUN` → Executes commands while building image.
* `EXPOSE` → Documents application port.
* `CMD/ENTRYPOINT` → Defines application startup command.

---

# Developer Thinking While Writing Dockerfile

1. What does my application need?

   * Java?
   * Node?
   * Python?

2. Select suitable base image.

3. Copy only required application files.

4. Define how the application starts.

5. Keep image small and avoid unnecessary files.

---

# Most Important Mental Model ⭐

```
Dockerfile
     |
     ↓
Build Image
     |
     ↓
Image
(OS + Runtime + Application)
     |
     ↓
Run
     |
     ↓
Container
```

**Image decides the environment. Container is only the running instance of that image.**
