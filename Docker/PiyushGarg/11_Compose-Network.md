# Docker Compose Network (Short Notes)

* Docker Compose **automatically creates a bridge network** for all services in `docker-compose.yml`.
* Every service/container joins this network by default (or a custom one if specified).
* Containers communicate using **service names** (Docker DNS), not `localhost`.

Example:

```yaml
services:
  mysql:
    networks:
      - spring-network

  springboot:
    networks:
      - spring-network

networks:
  spring-network:
```

### Communication

```text
Spring Boot
     │
jdbc:mysql://mysql:3306/db
     │
Docker DNS
     │
MySQL
```

### Why use a Compose network?

* Enables **container-to-container communication**.
* Automatically provides **service name resolution**.
* Isolates your application's containers from other Docker projects.
* No need to manually create a bridge network.

### Important Points

* `localhost` inside a container refers to **that container itself**.
* Use **service names** (e.g., `mysql`, `redis`) to communicate between containers.
* `ports:` are required **only if the host (browser, IDE, MySQL Workbench, etc.) needs access**.
* Containers on the same Compose network communicate using **container ports**, not host ports.

### Memory Trick

```text
Docker Compose
      │
Creates Bridge Network
      │
All Services Join
      │
Communicate using Service Names
      │
Browser Access → ports:
```
