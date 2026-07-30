# Docker Compose – Complete Cheat Notes (Step by Step)

---

# 1. What is Docker Compose?

Docker Compose is a tool used to **define and run multiple containers together** using a single YAML configuration file.

Instead of running many `docker run` commands, you write one file:

```text
docker-compose.yml
```

and start the entire application with one command.

---

# 2. Why do we need Docker Compose?

Imagine a Spring Boot project with:

* Spring Boot
* MySQL
* Redis
* RabbitMQ

Without Compose:

```text
Run MySQL
↓

Run Redis
↓

Run RabbitMQ
↓

Run Spring Boot
```

With Compose:

```text
docker compose up

↓

Everything starts automatically
```

---

# 3. Why is Compose used in Industry?

It solves problems like:

* Starting multiple containers
* Creating bridge network automatically
* Creating volumes automatically
* Managing environment variables
* Container dependency
* One command to start everything

---

# 4. Without Compose

```bash
docker network create my-network

docker volume create mysql-data

docker run ...

docker run ...

docker run ...

docker run ...
```

Lots of commands.

---

# 5. With Compose

```bash
docker compose up
```

Everything is created automatically.

---

# 6. Compose Workflow

```text
docker-compose.yml
        │
docker compose up
        │
Reads YAML
        │
Creates Network
        │
Creates Volumes
        │
Builds Images
        │
Creates Containers
        │
Starts Containers
```

---

# 7. Spring Boot Example

Project

```text
Spring Boot

MySQL

Redis
```

Compose automatically creates

```text
Bridge Network

↓

MySQL

↓

Redis

↓

Spring Boot
```

---

# 8. docker-compose.yml Structure

```yaml
services:
```

Defines all containers.

---

```yaml
build:
```

Build image from Dockerfile.

---

```yaml
image:
```

Use an existing image from Docker Hub.

---

```yaml
container_name:
```

Custom container name.

---

```yaml
ports:
```

Port mapping.

---

```yaml
environment:
```

Environment variables.

---

```yaml
volumes:
```

Attach named volumes or bind mounts.

---

```yaml
depends_on:
```

Starts one container before another.

---

```yaml
networks:
```

Custom network.

---

# 9. Spring Boot + MySQL Example

```yaml
version: "3.9"

services:

  mysql:
    image: mysql:8.4
    container_name: mysql-db

    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: dockerdb

    ports:
      - "3307:3306"

    volumes:
      - mysql-data:/var/lib/mysql

  springboot:

    build: .

    container_name: spring-app

    ports:
      - "8080:8080"

    environment:
      SPRING_PROFILES_ACTIVE: docker

    depends_on:
      - mysql

volumes:

  mysql-data:
```

---

# 10. What Compose Creates Automatically

```text
docker compose up

↓

Image

↓

Bridge Network

↓

Volume

↓

Containers

↓

Application Starts
```

No manual commands required.

---

# 11. Communication

Compose automatically creates a bridge network.

So Spring Boot connects like:

```properties
jdbc:mysql://mysql:3306/dockerdb
```

NOT

```properties
localhost
```

Because

```text
mysql

↓

Docker DNS

↓

Container IP
```

---

# 12. Compose File Explained

```yaml
services:
```

All containers.

---

```yaml
build: .
```

Build image using current folder's Dockerfile.

---

```yaml
image: mysql:8.4
```

Download image from Docker Hub.

---

```yaml
ports:
```

Host ↔ Container mapping.

---

```yaml
environment:
```

Pass environment variables.

---

```yaml
depends_on:
```

Start MySQL before Spring Boot.

---

```yaml
volumes:
```

Persistent storage.

---

# 13. Commands

Start

```bash
docker compose up
```

Start in background

```bash
docker compose up -d
```

Stop

```bash
docker compose stop
```

Stop and remove

```bash
docker compose down
```

Build again

```bash
docker compose build
```

Rebuild + Start

```bash
docker compose up --build
```

View logs

```bash
docker compose logs
```

Logs of one service

```bash
docker compose logs springboot
```

List running containers

```bash
docker compose ps
```

Restart

```bash
docker compose restart
```

Stop one service

```bash
docker compose stop mysql
```

Start one service

```bash
docker compose start mysql
```

Execute inside service

```bash
docker compose exec springboot sh
```

Pull latest images

```bash
docker compose pull
```

Remove everything including volumes

```bash
docker compose down -v
```

---

# 14. Compose vs Docker Run

| Docker Run        | Docker Compose      |
| ----------------- | ------------------- |
| One container     | Multiple containers |
| Many commands     | One YAML file       |
| Manual network    | Auto network        |
| Manual volume     | Auto volume         |
| Manual dependency | `depends_on`        |
| Hard to manage    | Easy to manage      |

---

# 15. Beginners' Doubts ⭐

### Does Compose replace Docker?

❌ No.

Compose is **built on top of Docker**.

Docker Engine still creates:

* Images
* Containers
* Networks
* Volumes

Compose simply automates the process.

---

### Does Compose create images?

If you use:

```yaml
build: .
```

✅ Yes.

If you use:

```yaml
image: mysql
```

It downloads the image.

---

### Does Compose create a bridge network?

Yes.

Every Compose project gets its own bridge network automatically.

---

### Does Compose create volumes?

Yes.

If declared:

```yaml
volumes:
```

Compose creates them automatically.

---

### Do I need `docker network create`?

No.

Compose creates the network automatically.

---

### Do I need `docker volume create`?

No.

Compose creates declared volumes automatically.

---

### Do containers communicate automatically?

Yes.

Using **service names**.

Example:

```properties
jdbc:mysql://mysql:3306/dockerdb
```

where `mysql` is the service name from the Compose file.

---

### If I change the Compose file, what should I do?

Run:

```bash
docker compose up --build
```

to rebuild images if needed and recreate containers with the updated configuration.

---

### When should I use Compose?

* Spring Boot + MySQL
* Spring Boot + Redis
* Microservices on one development machine
* Local development environments
* Integration testing

For large-scale production deployments, orchestration tools such as Kubernetes are commonly used instead of Docker Compose.

---

# 16. Golden Rules

* `docker-compose.yml` defines your entire application.
* One file can manage multiple containers.
* Compose automatically creates networks and declared volumes.
* Containers communicate using **service names**, not `localhost`.
* Use `build:` for your own application and `image:` for prebuilt images.
* `depends_on` controls startup order but does **not** guarantee that a service (like MySQL) is fully ready to accept connections.
* `docker compose up -d` is the command you'll use most often.

---

# Memory Flow

```text
docker-compose.yml
        │
docker compose up
        │
Read Configuration
        │
Create Network
        │
Create Volumes
        │
Build/Pull Images
        │
Create Containers
        │
Start Containers
        │
Spring Boot ⇄ MySQL ⇄ Redis
```

## Memory Trick

> **Docker = manages one container at a time. Docker Compose = manages an entire multi-container application from one YAML file with a single command.**


## STEPS 


* Spring Boot Application: **DockerApp**
* Docker profile: **docker**
* MySQL Database: **DockerVirtualDB**
* MySQL Root Password: **Sujit@123**
* Spring Boot listens on **8080**
* MySQL container name: **mysql-db**

Here's a clean `docker-compose.yml`.

```yaml
version: "3.9"

services:

  mysql:
    image: mysql:8.4
    container_name: mysql-db

    environment:
      MYSQL_ROOT_PASSWORD: Sujit@123
      MYSQL_DATABASE: DockerVirtualDB

    ports:
      - "3307:3306"

    volumes:
      - mysql-data:/var/lib/mysql

    networks:
      - spring-network

  springboot:
    build: .
    container_name: spring-app

    ports:
      - "8080:8080"

    environment:
      SPRING_PROFILES_ACTIVE: docker

    depends_on:
      - mysql

    networks:
      - spring-network

volumes:
  mysql-data:

networks:
  spring-network:
```

---

## Your `application-docker.properties`

Since both containers are on the same bridge network, **don't use `localhost`**.

Use:

```properties
spring.application.name=DockerApp

server.port=8080

spring.datasource.url=jdbc:mysql://mysql-db:3306/DockerVirtualDB
spring.datasource.username=root
spring.datasource.password=Sujit@123

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

Notice:

```properties
jdbc:mysql://mysql-db:3306
```

NOT

```properties
jdbc:mysql://localhost:3307
```

because inside Docker Compose, Spring Boot reaches MySQL through the **service/container name** `mysql-db`.

---

# Commands

Build images

```bash
docker compose build
```

Start everything

```bash
docker compose up
```

Start in background

```bash
docker compose up -d
```

Rebuild after code changes

```bash
docker compose up --build
```

View logs

```bash
docker compose logs
```

View Spring Boot logs

```bash
docker compose logs springboot
```

View MySQL logs

```bash
docker compose logs mysql
```

Show running services

```bash
docker compose ps
```

Stop services

```bash
docker compose stop
```

Restart services

```bash
docker compose restart
```

Stop and remove containers

```bash
docker compose down
```

Stop and remove containers + volumes

```bash
docker compose down -v
```

Enter Spring Boot container

```bash
docker compose exec springboot sh
```

Enter MySQL container

```bash
docker compose exec mysql sh
```

---

## Flow

```text
docker compose up
        │
        ▼
Build Spring Boot Image
        │
Pull MySQL Image
        │
Create Bridge Network
        │
Create mysql-data Volume
        │
Start mysql-db
        │
Start spring-app
        │
Spring Boot connects to:
jdbc:mysql://mysql-db:3306/DockerVirtualDB
```

This is very close to how a typical Spring Boot + MySQL setup is run locally using Docker Compose.

1.note
Docker Compose automatically names images like:

<project-folder>-<service-name>

In your case:

Project Folder : DockerApp
Service Name   : springboot

↓

dockerapp-springboot
