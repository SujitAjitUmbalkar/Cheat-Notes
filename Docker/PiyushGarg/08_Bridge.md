# Docker Bridge Network – Cheat Notes (Step by Step)

---

# 1. What is a Docker Bridge Network?

* Docker's **default private network**.
* Connects containers running on the **same Docker Host**.
* Works like a **virtual LAN (switch/router)** inside Docker.
* Every container gets its own **private IP address**.

```text
Docker Host
     │
 Bridge Network
 ┌────┴─────┐
 │          │
App      MySQL
```

---

# 2. What is a Docker Host?

A **Docker Host** is the machine where Docker Engine is installed.

Examples:

* Your laptop
* AWS EC2 instance
* Azure VM
* Linux server

```text
Windows/Linux
      │
Docker Engine
      │
Docker Host
```

---

# 3. Why Bridge Network?

Without a network:

```text
Container A    Container B

❌ Cannot communicate
```

With Bridge:

```text
Container A
      │
 Bridge
      │
Container B

✅ Can communicate
```

---

# 4. Default Bridge

Docker automatically creates a network named:

```text
bridge
```

Every new container joins it unless another network is specified.

Check:

```bash
docker network ls
```

---

# 5. Container Communication

Containers on the same bridge communicate using:

* Container Name ✅ (Recommended)
* Container IP

Example:

```text
Spring Boot
     │
mysql-db
```

Instead of

```text
172.18.0.3
```

Docker automatically resolves container names using its built-in DNS.

---

# 6. Docker DNS

Docker provides an internal DNS server.

```text
mysql-db
      │
Docker DNS
      │
172.x.x.x
```

No need to remember IP addresses.

---

# 7. Test Communication

Run two containers:

```bash
docker run -dit --name containerA ubuntu
```

```bash
docker run -dit --name containerB ubuntu
```

Open a terminal in `containerA`:

```bash
docker exec -it containerA bash
```

Ping by container name:

```bash
ping containerB
```

or by IP:

```bash
ping 172.18.0.3
```

If replies are received:

```text
64 bytes from containerB
```

Containers are communicating successfully.

---

# 8. Inspect Network

```bash
docker network inspect bridge
```

Shows:

* Subnet
* Gateway
* Connected containers
* Container IPs

---

# 9. Create Custom Bridge

```bash
docker network create my-network
```

Run containers on it:

```bash
docker run -d --name mysql-db --network my-network mysql
```

```bash
docker run -d --name spring-app --network my-network my-app
```

---

# 10. Spring Boot + MySQL

❌ Wrong

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/dockerDb
```

Inside a container, `localhost` means **that container itself**.

✅ Correct

```properties
spring.datasource.url=jdbc:mysql://mysql-db:3306/dockerDb
```

`mysql-db` is the MySQL container name.

---

# 11. Internet Access

Containers can also access the internet.

```text
Container
     │
Bridge
     │
Docker Host
     │
Internet
```

Example:

```text
Spring Boot
      │
Calls Google API
```

The request goes through the Docker Host to the internet.

---

# 12. Bridge vs Eureka

| Bridge                | Eureka                         |
| --------------------- | ------------------------------ |
| Docker networking     | Spring Cloud service discovery |
| Connects containers   | Finds microservices            |
| Provides connectivity | Provides service registry      |
| Uses Docker DNS       | Uses Eureka registry           |

---

# 13. Useful Commands

List networks

```bash
docker network ls
```

Inspect network

```bash
docker network inspect bridge
```

Create network

```bash
docker network create my-network
```

Run on network

```bash
docker run --network my-network my-app
```

Connect existing container

```bash
docker network connect my-network containerA
```

Disconnect

```bash
docker network disconnect my-network containerA
```

Remove network

```bash
docker network rm my-network
```

Open container terminal

```bash
docker exec -it containerA bash
```

Check IP

```bash
hostname -i
```

Ping another container

```bash
ping containerB
```

or

```bash
ping <container-ip>
```

---

# 14. Golden Rules

* Docker creates the **bridge** network by default.
* Containers on the same bridge can communicate.
* Every container gets its own private IP.
* Prefer **container names** over IP addresses.
* Docker's built-in DNS resolves container names automatically.
* `localhost` inside a container refers to **that container**, not the Docker Host.
* Bridge networking works only on the **same Docker Host**.
* Use a **custom bridge network** for your own applications instead of the default bridge.

---

## Memory Flow

```text
Internet
     │
Docker Host
     │
Bridge Network (Virtual LAN)
 ┌───┴─────────────┐
 │                 │
Spring Boot    MySQL
Container      Container
 │                 ▲
 │ ping mysql-db   │
 │ JDBC:mysql-db   │
 └────Docker DNS───┘
```

### One-line Memory Trick

> **A Docker Bridge Network is a private virtual LAN inside a Docker Host that lets containers communicate using container names or IP addresses, while also providing internet access through the Docker Host.**
