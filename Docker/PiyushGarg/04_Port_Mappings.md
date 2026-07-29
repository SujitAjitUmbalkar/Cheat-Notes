# Docker Port Mapping – Concise Cheat Notes

### Application Port

```properties
server.port=8080
```

* Defines the port **Spring Boot listens on**.
* Default: **8080**.
* Override:

```bash
docker run -e SERVER_PORT=9090 my-app
```

or

```bash
docker run my-app --server.port=9090
```

---

### Dockerfile – EXPOSE

```dockerfile
EXPOSE 8080
```

* Declares the **expected container port**.
* Metadata/documentation only.
* Does **NOT** publish the port.
* Does **NOT** change the application's port.
* Optional but recommended.

Multiple ports:

```dockerfile
EXPOSE 8080
EXPOSE 9090
```

or

```dockerfile
EXPOSE 8080 9090
```

---

### Publish Port

Syntax:

```bash
docker run -p HostPort:ContainerPort image
```

Example:

```bash
docker run -p 8080:8080 my-app
```

```
Host:8080  ───►  Container:8080
```

---

### Different Host Port

```properties
server.port=8080
```

```dockerfile
EXPOSE 8080
```

```bash
docker run -p 9090:8080 my-app
```

```
localhost:9090 ───► Spring Boot:8080
```

---

### Different Application Port

```properties
server.port=9090
```

```dockerfile
EXPOSE 9090
```

```bash
docker run -p 8080:9090 my-app
```

```
localhost:8080 ───► Spring Boot:9090
```

---

### Override Application Port

```dockerfile
EXPOSE 8080
```

```bash
docker run -e SERVER_PORT=9090 -p 8080:9090 my-app
```

* App runs on **9090**.
* `EXPOSE 8080` is ignored at runtime (documentation only).

---

### No EXPOSE

```dockerfile
FROM eclipse-temurin:21-jdk
ENTRYPOINT ["java","-jar","app.jar"]
```

```bash
docker run -p 8080:8080 my-app
```

✅ Works normally.

---

### No `-p`

```bash
docker run my-app
```

* App runs.
* Accessible **only inside the container**.
* `localhost` on the host won't work.

---

### Multiple Containers

```bash
docker run -p 8080:8080 my-app
docker run -p 8081:8080 my-app
```

```
localhost:8080 ─► Container A:8080
localhost:8081 ─► Container B:8080
```

* Same **container port** ✅
* Different **host ports** ✅

---

## Golden Rules

* `server.port` → Application listening port.
* `EXPOSE` → Declares/document expected container port.
* `-p Host:Container` → Connects host port to container port.
* Host and container ports **can be different**.
* `EXPOSE` is **optional**.
* Without `-p`, the app is **not accessible from the host**.
* Multiple containers **can share the same container port**.
* Two containers **cannot share the same host port**.

---

## Memory Formula

```
server.port
      │
(Application listens)
      │
      ▼
EXPOSE
(Documentation)
      │
      ▼
docker run -p Host:Container
      │
(Port Mapping)
      ▼
Browser (localhost:HostPort)
```

### One-line Memory Trick

> **`server.port` listens → `EXPOSE` describes → `-p` connects.**


# Docker Commands Related to Ports

## 1. Publish Same Host & Container Port

```bash
docker run -p 8080:8080 my-app
```

> Host `8080` → Container `8080`

---

## 2. Different Host Port

```bash
docker run -p 9090:8080 my-app
```

> `localhost:9090` → Container `8080`

---

## 3. Different Application Port (Application listens on 9090)

```bash
docker run -p 8080:9090 my-app
```

> Host `8080` → Container `9090`

---

## 4. Override Spring Boot Port

```bash
docker run -e SERVER_PORT=9090 -p 8080:9090 my-app
```

> Application runs on `9090`, browser uses `8080`

---

## 5. Override Using Command-Line Argument

```bash
docker run -p 8080:9090 my-app --server.port=9090
```

---

## 6. No Port Mapping

```bash
docker run my-app
```

> Application runs, but **not accessible** from the host.

---

## 7. Run Two Containers

```bash
docker run -d --name app1 -p 8080:8080 my-app
```

```bash
docker run -d --name app2 -p 8081:8080 my-app
```

---

## 8. Random Host Port

```bash
docker run -P my-app
```

* Publishes all `EXPOSE`d ports.
* Docker chooses random host ports.

Example:

```text
32768 -> 8080
```

---

## 9. View Port Mapping

```bash
docker ps
```

Example:

```text
0.0.0.0:8080->8080/tcp
```

---

## 10. Inspect Container Ports

```bash
docker port <container-name>
```

Example:

```bash
docker port app1
```

Output:

```text
8080/tcp -> 0.0.0.0:8080
```

---

## 11. List Running Containers with Ports

```bash
docker ps
```

---

## 12. Show Detailed Port Information

```bash
docker inspect <container-name>
```

---

## 13. Stop Container Using a Port

```bash
docker stop app1
```

---

## 14. Remove Container

```bash
docker rm app1
```

---

## 15. Run Multiple Port Mappings

```bash
docker run -p 8080:8080 -p 5005:5005 my-app
```

Example:

* `8080` → Application
* `5005` → Java Remote Debugging

---

# Quick Revision

```bash
docker run -p 8080:8080 my-app
```

> Same host and container port

```bash
docker run -p 9090:8080 my-app
```

> Different host port

```bash
docker run -p 8080:9090 my-app
```

> Different container (application) port

```bash
docker run -e SERVER_PORT=9090 -p 8080:9090 my-app
```

> Override Spring Boot port

```bash
docker run my-app
```

> No host access

```bash
docker run -P my-app
```

> Docker assigns random host port(s) for exposed ports

These are the Docker commands you'll use most often when working with ports in Spring Boot applications.
