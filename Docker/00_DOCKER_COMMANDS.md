# Docker Commands


### 1. Docker Images

| Command                        | Purpose                                  |
| ------------------------------ | ---------------------------------------- |
| `docker images`                | List all locally available Docker images |
| `docker image ls`              | Same as `docker images`                  |
| `docker pull <image>`          | Download an image from a registry        |
| `docker rmi <image>`           | Remove an image                          |
| `docker image inspect <image>` | View detailed information about an image |
| `docker image prune`           | Remove unused/dangling images            |

### 2. Docker Container

| Command                                     | Purpose                                      |
| ------------------------------------------- | -------------------------------------------- |
| `docker ps`                                 | List running containers                      |
| `docker ps -a`                              | List all containers, including stopped       |
| `docker run <image>`                        | Create and start a container                 |
| `docker start <container>`                  | Start a stopped container                    |
| `docker stop <container>`                   | Stop a running container                     |
| `docker restart <container>`                | Restart a container                          |
| `docker rm <container>`                     | Remove a stopped container                   |
| `docker logs <container>`                   | View container logs                          |
| `docker exec -it <container> <command>`     | Execute a command inside a running container |
| `docker inspect <container>`                | View detailed container information          |
| `docker rename <old> <new>`                 | Rename a container                           |
| `docker cp <container>:<path> <local-path>` | Copy files from container to host            |
| `docker stats`                              | View resource usage of running containers    |

### 3. DockerFile

| Command / Instruction            | Purpose                                    |
| -------------------------------- | ------------------------------------------ |
| `FROM <image>`                   | Specify the base image                     |
| `WORKDIR <path>`                 | Set the working directory                  |
| `COPY <src> <dest>`              | Copy files into the image                  |
| `RUN <command>`                  | Execute a command while building the image |
| `ENV KEY=value`                  | Set environment variables                  |
| `EXPOSE <port>`                  | Document the port the app uses             |
| `CMD ["command"]`                | Default command when container starts      |
| `ENTRYPOINT ["command"]`         | Define the main executable                 |
| `docker build -t <name> .`       | Build an image from the Dockerfile         |
| `docker build -t <name>:<tag> .` | Build an image with a specific tag         |

### 4. Port Mapping

| Command                                              | Purpose                                                        |
| ---------------------------------------------------- | -------------------------------------------------------------- |
| `docker run -p 8080:8080 my-app`                     | Map **host 8080 → container 8080**                             |
| `docker run -p 9090:8080 my-app`                     | Map **host 9090 → container 8080**                             |
| `docker run -p 8080:9090 my-app`                     | Map **host 8080 → container 9090**                             |
| `docker run -e SERVER_PORT=9090 -p 8080:9090 my-app` | Override Spring Boot port to `9090` and map host `8080 → 9090` |
| `docker run -p 8080:9090 my-app --server.port=9090`  | Override Spring Boot port using command-line argument          |
| `docker run my-app`                                  | Run without publishing any host port                           |
| `docker run -P my-app`                               | Publish all `EXPOSE`d ports using **random host ports**        |
| `docker ps`                                          | View running containers and their port mappings                |
| `docker port <container>`                            | Show port mappings of a specific container                     |
| `docker inspect <container>`                         | View detailed container/network/port information               |
| `docker run -d --name app1 -p 8080:8080 my-app`      | Run container in background with port mapping                  |
| `docker run -d --name app2 -p 8081:8080 my-app`      | Run another container using a different host port              |
| `docker run -p 8080:8080 -p 5005:5005 my-app`        | Publish multiple ports                                         |
| `docker stop <container>`                            | Stop a container using the port                                |
| `docker rm <container>`                              | Remove a stopped container                                     |

```
