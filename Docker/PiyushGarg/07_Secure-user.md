# Docker Secure User (Non-Root User) – Cheat Notes

## What is a Secure User?

By default, Docker containers run as the **root** user.

```text
Container
   │
 root
```

Running as root gives full permissions, which is a security risk.

---

## Why create a Secure User?

* Prevent accidental deletion/modification of system files.
* Reduce damage if the application is compromised.
* Follow the **Principle of Least Privilege**.
* Industry best practice for production.

---

## Create a Secure User

```dockerfile
# Create a new Linux user
RUN useradd -m dockeruser
```

`-m` → Creates a home directory.

---

## Switch to Secure User

```dockerfile
USER dockeruser
```

Everything after this (`CMD`, `ENTRYPOINT`, scripts, etc.) runs as `dockeruser`.

---

## Verify Current User

Enter the container:

```bash
docker exec -it <container-id> sh
```

Check current user:

```bash
whoami
```

Output:

```text
dockeruser
```

---

# How to Become Admin (Root) Again?

Inside a running container, you **cannot** simply switch back to root unless you started the container with the necessary privileges or you're already root.

### Option 1 (Recommended): Open a shell as root

```bash
docker exec -u root -it <container-id> sh
```

or

```bash
docker exec -u 0 -it <container-id> sh
```

`0` is the UID of the root user.

Check:

```bash
whoami
```

Output:

```text
root
```

---

### Option 2: Change the Dockerfile

If you want the application itself to run as root again, remove or comment out:

```dockerfile
USER dockeruser
```

Then rebuild the image:

```bash
docker build -t my-app .
```

Now the container will run as `root`.

---

## Dockerfile Example

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

# Create secure user
RUN useradd -m dockeruser

# Run application as non-root user
USER dockeruser

EXPOSE 8080

ENTRYPOINT ["java","-jar","app.jar"]
```

---

# Golden Rules

* Containers run as **root** by default.
* Create a non-root user using `RUN useradd -m`.
* Switch using `USER dockeruser`.
* Verify with `whoami`.
* To open a root shell later:

```bash
docker exec -u root -it <container-id> sh
```

* If you remove the `USER` instruction and rebuild, the container runs as **root** again.

## Memory Trick

> **Build as root (when needed), run as a non-root user, use root only for maintenance or debugging.**
