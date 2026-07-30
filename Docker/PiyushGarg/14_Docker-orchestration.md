# Docker Orchestration (Cheat Notes)

## What is Docker Orchestration?

Docker Orchestration is the **automatic management of multiple Docker containers**.

Instead of manually starting, stopping, scaling and monitoring containers, an orchestration tool does it automatically.

---

## Why is it Needed?

Suppose your application has:

```text
Spring Boot
MySQL
Redis
RabbitMQ
API Gateway
Order Service
Inventory Service
Payment Service
```

Managing them manually becomes difficult.

Docker Orchestration automates everything.

---

## Responsibilities of an Orchestrator

* Start containers
* Stop containers
* Restart failed containers
* Scale containers (increase/decrease instances)
* Service Discovery
* Load Balancing
* Rolling Updates
* Health Checks
* Networking
* Volume Management

---

## Without Orchestration

```text
You

docker run
docker stop
docker restart
docker network
docker volume
docker logs

(All managed manually)
```

---

## With Orchestration

```text
You
    │
Deploy Application
    │
Orchestrator
    │
Automatically
├── Creates Containers
├── Creates Networks
├── Creates Volumes
├── Restarts Failed Containers
├── Load Balances Traffic
├── Scales Containers
└── Updates Application
```

---

# Popular Orchestration Tools

| Tool             | Usage                        |
| ---------------- | ---------------------------- |
| Docker Compose   | Single Machine (Development) |
| Docker Swarm     | Multiple Docker Hosts        |
| Kubernetes (K8s) | Industry Standard            |

---

# Docker Compose vs Docker Swarm vs Kubernetes

| Feature          | Compose    | Swarm       | Kubernetes            |
| ---------------- | ---------- | ----------- | --------------------- |
| Single Machine   | ✅          | ❌           | ❌                     |
| Multi Machine    | ❌          | ✅           | ✅                     |
| Auto Scaling     | ❌          | ✅           | ✅                     |
| Self Healing     | ❌          | ✅           | ✅                     |
| Load Balancing   | ❌          | ✅           | ✅                     |
| Production Ready | Small Apps | Medium Apps | Large Enterprise Apps |

---

# Example

Without Orchestration

```text
Spring Boot crashes

↓

Application Down

↓

You restart it manually
```

With Orchestration

```text
Spring Boot crashes

↓

Orchestrator detects failure

↓

Automatically creates new container

↓

Application continues running
```

---

# Flow

```text
Developer
      │
      ▼
Docker Image
      │
      ▼
Docker Orchestrator
      │
      ├── Container 1
      ├── Container 2
      ├── Container 3
      └── Container N
```

---

# Memory Trick

```text
Docker
=
Run Containers

Docker Compose
=
Manage Multiple Containers (Single Machine)

Docker Swarm / Kubernetes
=
Manage Multiple Containers on Multiple Machines
```

---

# Golden Rules

* **Docker** → Creates and runs containers.
* **Docker Compose** → Manages multiple containers on **one host**.
* **Docker Orchestration** → Manages containers **automatically**, especially across multiple hosts.
* **Docker Swarm** and **Kubernetes** are orchestration platforms.
* The orchestrator handles deployment, scaling, recovery, networking, and updates so you don't have to do them manually.

> **One-line definition:** Docker Orchestration is the automated deployment, management, scaling, networking, and recovery of multiple Docker containers.
