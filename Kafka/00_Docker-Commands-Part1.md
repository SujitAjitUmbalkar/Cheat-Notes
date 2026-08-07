Below is your **Docker command cheat sheet from fundamentals → Dockerfile**, keeping only the commands and concepts you actually need while learning/practicing.

## 1. Docker Fundamentals

| Command / Usage                                    | Parts described                                                            |
| -------------------------------------------------- | -------------------------------------------------------------------------- |
| `docker --version` — Check Docker version          | `docker` = Docker CLI, `--version` = installed version                     |
| `docker info` — Show Docker system information     | Shows containers, images, storage driver, resources, Docker Engine details |
| `docker help` — Show available commands            | `help` = command reference                                                 |
| `docker <command> --help` — Get help for a command | Example: `docker run --help`                                               |

---

# 2. Docker Images

An **image is the blueprint/template** from which containers are created.

| Command / Usage                                     | Parts described                                      |
| --------------------------------------------------- | ---------------------------------------------------- |
| `docker pull nginx` — Download image                | `pull` = download image, `nginx` = image name        |
| `docker images` — List downloaded images            | Lists repository, tag, image ID, size                |
| `docker image ls` — List images                     | Same purpose as `docker images`                      |
| `docker inspect nginx` — Detailed image information | `inspect` = detailed JSON information                |
| `docker history nginx` — Show image layers          | `history` = layers/instructions used to create image |
| `docker rmi nginx` — Remove image                   | `rmi` = remove image                                 |
| `docker image prune` — Remove unused images         | `prune` = clean unused image data                    |

### Image naming

```text
nginx:latest
│     │
│     └── tag
└──────── image/repository name
```

| Concept      | Meaning                    |
| ------------ | -------------------------- |
| `nginx`      | Repository/image name      |
| `latest`     | Tag                        |
| `nginx:1.27` | Specific image version     |
| `IMAGE_ID`   | Unique identifier of image |

---

# 3. Running Containers

A **container is a running/created instance of an image**.

| Command / Usage                                               | Parts described                                                         |
| ------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `docker run nginx` — Create and start container               | `run` = create + start, `nginx` = image                                 |
| `docker run -d nginx` — Run in background                     | `-d` = detached mode                                                    |
| `docker run --name myapp nginx` — Give container a name       | `--name` = custom container name                                        |
| `docker run -it ubuntu bash` — Open interactive shell         | `-i` = interactive, `-t` = terminal, `ubuntu` = image, `bash` = command |
| `docker run -d --name web nginx` — Named background container | Combines `-d` + `--name`                                                |
| `docker run -p 8080:80 nginx` — Map host port to container    | `-p HOST:CONTAINER`, `8080` = host, `80` = container                    |

### ⭐ Most important

```bash
docker run -d -p 8080:80 --name web nginx
```

```text
docker run
    ↓
-d                  → background
-p 8080:80          → host 8080 → container 80
--name web          → container name
nginx               → image
```

---

# 4. Container Management

| Command / Usage                                       | Parts described                    |
| ----------------------------------------------------- | ---------------------------------- |
| `docker ps` — Show running containers                 | `ps` = process/status              |
| `docker ps -a` — Show all containers                  | `-a` = all, including stopped      |
| `docker start myapp` — Start stopped container        | `start` = start existing container |
| `docker stop myapp` — Stop container                  | `stop` = gracefully stop           |
| `docker restart myapp` — Restart container            | `restart` = stop + start           |
| `docker rm myapp` — Remove stopped container          | `rm` = remove container            |
| `docker rm -f myapp` — Force remove running container | `-f` = force                       |
| `docker rename old new` — Rename container            | `rename` = change container name   |

---

# 5. Container Logs & Inspection

| Command / Usage                                         | Parts described                                           |
| ------------------------------------------------------- | --------------------------------------------------------- |
| `docker logs myapp` — View container logs               | `logs` = stdout/stderr logs                               |
| `docker logs -f myapp` — Follow live logs               | `-f` = follow                                             |
| `docker logs --tail 50 myapp` — Show last 50 lines      | `--tail 50` = last 50 log lines                           |
| `docker inspect myapp` — Detailed container information | `inspect` = configuration/network/mount/state information |
| `docker stats` — Live resource usage                    | Shows CPU, memory, network, etc.                          |
| `docker top myapp` — Processes inside container         | `top` = running processes                                 |

---

# 6. Execute Commands Inside Container

Very important when debugging containers.

| Command / Usage                                      | Parts described                                     |
| ---------------------------------------------------- | --------------------------------------------------- |
| `docker exec myapp ls` — Execute command             | `exec` = execute inside running container           |
| `docker exec -it myapp bash` — Open Bash shell       | `-i` = interactive, `-t` = terminal, `bash` = shell |
| `docker exec -it myapp sh` — Open shell              | `sh` = lightweight shell                            |
| `docker exec myapp env` — Show environment variables | `env` = environment variables                       |

### Example

```bash
docker exec -it mysql-container bash
```

Flow:

```text
Docker
  ↓
mysql-container
  ↓
bash shell
```

---

# 7. Copy Files

| Command / Usage                                     | Parts described                                                              |
| --------------------------------------------------- | ---------------------------------------------------------------------------- |
| `docker cp file.txt myapp:/app/` — Host → container | `cp` = copy, `myapp:/app/` = destination inside container                    |
| `docker cp myapp:/app/log.txt .` — Container → host | `myapp:/app/log.txt` = source inside container, `.` = current host directory |

---

# 8. Environment Variables

Environment variables are commonly used for **Spring Boot configuration, DB credentials, ports, etc.**

| Command / Usage                                              | Parts described             |
| ------------------------------------------------------------ | --------------------------- |
| `docker run -e APP_ENV=dev nginx` — Set environment variable | `-e` = environment variable |
| `docker run --env APP_ENV=dev nginx` — Same as above         | `--env` = long form of `-e` |
| `docker exec myapp env` — View variables                     | `env` = display environment |

Example:

```bash
docker run -d \
  --name myapp \
  -e SPRING_PROFILES_ACTIVE=dev \
  myapp:1.0
```

---

# 9. Container Restart Policy

Useful for applications such as **Spring Boot, Kafka, Redis, MySQL**.

| Command / Usage                                                               | Parts described                                                      |
| ----------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `docker run --restart no nginx` — Never automatically restart                 | `no` = default                                                       |
| `docker run --restart on-failure nginx` — Restart after failure               | `on-failure` = restart when container exits with error               |
| `docker run --restart always nginx` — Always restart                          | `always` = restart automatically                                     |
| `docker run --restart unless-stopped nginx` — Restart unless manually stopped | `unless-stopped` = survives Docker restart unless explicitly stopped |

---

# 10. Docker Port Mapping

This is **very important** for your Spring Boot/Kafka work.

```bash
docker run -p 8080:8080 myapp
```

| Part          | Meaning           |
| ------------- | ----------------- |
| `-p`          | Publish/map port  |
| First `8080`  | Host machine port |
| Second `8080` | Container port    |

Example:

```bash
-p 9092:9092
```

```text
Windows/Host
     │
     │ localhost:9092
     ↓
Docker Container
     │
     │ :9092
     ↓
Kafka
```

### Important distinction

| Thing          | Meaning                                 |
| -------------- | --------------------------------------- |
| `EXPOSE 8080`  | Documents that container uses port 8080 |
| `-p 8080:8080` | Actually publishes/maps the port        |

---

# 11. Docker Volumes

Containers are **ephemeral**. If the container is removed, data inside its writable layer can disappear.

Volumes provide persistent storage.

| Command / Usage                                                | Parts described                 |
| -------------------------------------------------------------- | ------------------------------- |
| `docker volume ls` — List volumes                              | `volume ls` = list volumes      |
| `docker volume create mysql-data` — Create volume              | `create` = create named volume  |
| `docker volume inspect mysql-data` — Volume details            | `inspect` = details             |
| `docker volume rm mysql-data` — Remove volume                  | `rm` = remove                   |
| `docker run -v mysql-data:/var/lib/mysql mysql` — Mount volume | `-v HOST_VOLUME:CONTAINER_PATH` |

### Volume mapping

```text
mysql-data
     ↓
/var/lib/mysql
     ↓
MySQL container
```

---

# 12. Bind Mount

A bind mount connects a **specific host directory** to a container directory.

| Command / Usage                                                                     | Parts described                                                 |
| ----------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `docker run -v ./app:/app nginx` — Mount host folder                                | `./app` = host directory, `/app` = container directory          |
| `docker run --mount type=bind,source=./app,target=/app nginx` — Explicit bind mount | `type=bind` = bind mount, `source` = host, `target` = container |

### Volume vs Bind Mount

| Volume             | Bind Mount                       |
| ------------------ | -------------------------------- |
| Managed by Docker  | Managed by you/OS                |
| `mysql-data:/data` | `./data:/data`                   |
| Good for databases | Good for development/source code |

---

# 13. Docker Networks

Containers often need to communicate with each other.

| Command / Usage                                                       | Parts described                   |
| --------------------------------------------------------------------- | --------------------------------- |
| `docker network ls` — List networks                                   | `network ls` = available networks |
| `docker network create mynetwork` — Create network                    | `create` = create custom network  |
| `docker network inspect mynetwork` — Network details                  | `inspect` = network configuration |
| `docker network rm mynetwork` — Remove network                        | `rm` = remove                     |
| `docker run --network mynetwork nginx` — Connect container to network | `--network` = network selection   |

### Container-to-container communication

If:

```text
Network: backend

Spring Boot → MySQL
Spring Boot → Redis
Spring Boot → Kafka
```

Containers can communicate using **container/service names** rather than `localhost`.

For example:

```text
jdbc:mysql://mysql:3306/mydb
```

Here:

```text
mysql = container/service name
3306 = MySQL container port
```

---

# 14. Docker Image Creation

Now we reach the important part: **Dockerfile**.

Basic flow:

```text
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Container
```

| Command / Usage                                          | Parts described                                                                              |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `docker build -t myapp:1.0 .` — Build image              | `build` = create image, `-t` = tag/name, `myapp:1.0` = image name + tag, `.` = build context |
| `docker build -t myapp .` — Build image with default tag | `.` = current directory as build context                                                     |
| `docker image ls` — Verify image                         | Lists newly created image                                                                    |

### ⭐ Most important build command

```bash
docker build -t my-spring-app:1.0 .
```

```text
docker build
     ↓
-t
     ↓
my-spring-app:1.0
     ↓
.
     ↓
Dockerfile + build context
```

---

# 15. Dockerfile

A **Dockerfile is a text file containing instructions to build an image.**

Example:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Now the important instructions:

| Dockerfile instruction / Usage                           | Parts described                                                        |
| -------------------------------------------------------- | ---------------------------------------------------------------------- |
| `FROM eclipse-temurin:21-jre` — Choose base image        | `FROM` = base image, `eclipse-temurin` = image, `21-jre` = tag         |
| `WORKDIR /app` — Set working directory                   | `WORKDIR` = directory used by subsequent instructions                  |
| `COPY target/app.jar app.jar` — Copy file into image     | First = source, second = destination                                   |
| `ADD file /app` — Add files to image                     | Similar to `COPY`, but has additional behavior; prefer `COPY` normally |
| `RUN apt-get update` — Execute command while building    | `RUN` executes during **image build**                                  |
| `ENV APP_ENV=prod` — Set environment variable            | `ENV` = environment variable inside image/container                    |
| `ARG VERSION=1.0` — Build-time variable                  | `ARG` = available during build                                         |
| `EXPOSE 8080` — Document container port                  | Does **not** publish the port                                          |
| `CMD ["java","-jar","app.jar"]` — Default command        | Can be overridden when container starts                                |
| `ENTRYPOINT ["java","-jar","app.jar"]` — Main executable | Defines the main process                                               |
| `USER appuser` — Run as specified user                   | Improves security by avoiding root                                     |
| `VOLUME /data` — Declare mount point                     | Indicates persistent/mountable storage                                 |
| `HEALTHCHECK ...` — Define container health test         | Docker can determine healthy/unhealthy state                           |


Example:

```dockerfile
FROM ubuntu

RUN apt-get update

CMD ["echo", "Hello"]
```

Flow:

```text
docker build
     ↓
RUN executes
     ↓
Image created
     ↓
docker run
     ↓
CMD executes
```

---



---

# 19. Dockerfile `.dockerignore`

`.dockerignore` prevents unnecessary files from being sent as the **build context**.

Example:

```text
.git
.idea
target
node_modules
*.log
.env
```

| Concept         | Purpose                                                 |
| --------------- | ------------------------------------------------------- |
| `.dockerignore` | Excludes unnecessary/sensitive files from build context |
| `.git`          | Don't send Git history                                  |
| `.idea`         | Don't send IDE files                                    |
| `target`        | Don't send Maven build directory if not needed          |
| `.env`          | Avoid accidentally sending environment secrets          |

---

# 20. Complete Basic Spring Boot Example

### Dockerfile

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/myapp.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Build

```bash
docker build -t myapp:1.0 .
```

### Run

```bash
docker run -d \
  --name myapp \
  -p 8080:8080 \
  myapp:1.0
```

### Check

```bash
docker ps
```

### Logs

```bash
docker logs myapp
```

### Stop

```bash
docker stop myapp
```

### Remove

```bash
docker rm myapp
```

### Overall flow to remember

```text
                 Dockerfile
                     │
                     ▼
              docker build
                     │
                     ▼
               Docker Image
                     │
                     ▼
                docker run
                     │
                     ▼
                Container
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Network     Volume      Port
          │          │          │
          ▼          ▼          ▼
       Other       Persistent   Host
     Containers      Data      Machine
```

This covers the **core Docker knowledge up through Dockerfile** without going into advanced internals yet.
