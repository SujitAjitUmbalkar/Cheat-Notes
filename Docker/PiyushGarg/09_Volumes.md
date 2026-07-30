# Docker Volumes – Complete Cheat Notes (Step by Step)

---

# 1. Why do we need Volumes?

Containers are **temporary**.

If a container is deleted:

```text
Container Deleted
        │
Application Data ❌ Lost
```

Example:

* MySQL Database
* Uploaded Images
* Log Files

Everything inside the container is lost unless stored in a volume.

---

# 2. What is a Volume?

A **Docker Volume** is a **persistent storage location** managed by Docker.

```text
Container
     │
 Volume
     │
Host Machine
```

Even if the container is deleted:

```text
Container Deleted

↓

Volume Still Exists ✅

↓

New Container

↓

Same Data
```

---

# 3. Types of Storage

## (A) Anonymous Volume

Docker creates it automatically.

```text
Container
    │
Anonymous Volume
```

Hard to identify and reuse.

---

## (B) Named Volume (Custom Volume)

You create it yourself.

```text
Container
     │
dockerDb
```

Reusable.

Docker manages its location.

---

## (C) Bind Mount (Host Volume)

Uses an actual folder from your computer.

```text
C:\Projects\Data
       │
Container
```

You know exactly where the files are stored.

---

# 4. Difference

| Named Volume                  | Bind Mount (Host Volume)       |
| ----------------------------- | ------------------------------ |
| Managed by Docker             | Managed by you                 |
| Docker decides location       | You decide location            |
| Easier for databases          | Easier for source code & logs  |
| Portable                      | Depends on host path           |
| Recommended for production DB | Mostly used during development |

---

# 5. Named Volume Flow

```text
Container
      │
Named Volume
      │
Docker Storage
```

Delete container

↓

```text
Volume Still Exists
```

Attach another container

↓

```text
Same Data Available
```

---

# 6. Bind Mount Flow

```text
Host Folder
C:\Projects

      │

Bind Mount

      │

Container
```

If a file changes:

```text
Host

↓

Container
```

Immediately visible.

Useful while coding.

---

# 7. Create Named Volume

```bash
docker volume create mysql-data
```

Definition

Creates a reusable Docker-managed volume.

---

# 8. List Volumes

```bash
docker volume ls
```

Definition

Displays all Docker volumes.

---

# 9. Inspect Volume

```bash
docker volume inspect mysql-data
```

Definition

Shows:

* Mount location
* Driver
* Metadata

---

# 10. Remove Volume

```bash
docker volume rm mysql-data
```

Definition

Deletes the specified volume.

---

# 11. Remove Unused Volumes

```bash
docker volume prune
```

Definition

Deletes all unused volumes.

---

# 12. Run Container with Named Volume

```bash
docker run -d \
--name mysql \
-v mysql-data:/var/lib/mysql \
mysql
```

Definition

Mounts Docker-managed volume `mysql-data` to MySQL's data directory.

---

# 13. Run Container with Bind Mount

Windows

```bash
docker run -v C:\Docker\Data:/app/data my-app
```

Linux

```bash
docker run -v /home/user/data:/app/data my-app
```

Definition

Maps a folder from the host machine into the container.

---

# 14. Volume Syntax

Named Volume

```text
-v volume-name:container-path
```

Example

```bash
-v mysql-data:/var/lib/mysql
```

---

Bind Mount

```text
-v host-path:container-path
```

Example

```bash
-v C:\Projects:/app
```

---

# 15. What happens internally?

Named Volume

```text
Container
      │
Docker Volume
      │
Host Disk
```

Bind Mount

```text
Container
      │
Direct Mapping
      │
Your Folder
```

---

# 16. Spring Boot Example

Store uploaded images

```text
Host

C:\Uploads

      │

Bind Mount

      │

Container

/app/uploads
```

Database

```text
Container

↓

Named Volume

↓

Database Files
```

---

# 17. Real Industry Usage

| Use Case          | Storage      |
| ----------------- | ------------ |
| MySQL             | Named Volume |
| PostgreSQL        | Named Volume |
| Redis Persistence | Named Volume |
| Uploads           | Bind Mount   |
| Logs              | Bind Mount   |
| Source Code       | Bind Mount   |

---

# 18. Useful Commands

Create volume

```bash
docker volume create my-volume
```

List

```bash
docker volume ls
```

Inspect

```bash
docker volume inspect my-volume
```

Remove

```bash
docker volume rm my-volume
```

Delete unused

```bash
docker volume prune
```

Run using named volume

```bash
docker run -v my-volume:/app/data my-app
```

Run using bind mount

```bash
docker run -v C:\Data:/app/data my-app
```

---

# 19. Golden Rules

* Containers are **temporary**.
* Volumes are **persistent**.
* Deleting a container does **not** delete its volume.
* Named volumes are managed by Docker.
* Bind mounts use your own folders.
* Databases should usually use **named volumes**.
* Source code and logs commonly use **bind mounts**.
* Multiple containers can mount the same volume if the application supports it.

---

# 20. Concepts Beginners Often Don't Understand ⭐

### 1. Volume ≠ Container

Deleting:

```bash
docker rm my-container
```

does **not** delete:

```text
Named Volume
```

Data is still there.

---

### 2. Bind Mount is Live

Edit a file on your laptop:

```text
Host

↓

Container immediately sees it
```

No rebuild required.

---

### 3. Named Volume Location

You usually **don't need to know** where Docker stores a named volume.

Docker manages it.

If needed:

```bash
docker volume inspect my-volume
```

shows the actual mount point.

---

### 4. First Mount Copies Existing Data

If the container path already contains files and the named volume is empty:

```text
Container Files

↓

Copied into Volume (first time only)
```

This surprises many beginners.

---

### 5. Mount Hides Existing Files

Suppose the image contains:

```text
/app
 ├── config.yml
 └── data.txt
```

If you bind mount:

```bash
-v C:\MyFolder:/app
```

Inside the container you'll now see the contents of `C:\MyFolder` at `/app`; the original `/app` files from the image are **hidden** while the mount is active.

---

### Memory Flow

```text
Without Volume

Container
     │
Delete Container
     │
Data Lost ❌

--------------------------------

With Named Volume

Container
     │
Named Volume
     │
Delete Container
     │
Volume Exists ✅
     │
New Container
     │
Same Data ✅

--------------------------------

With Bind Mount

Host Folder
      │
Container
      │
Edit File on Host
      │
Container Instantly Sees Changes ✅
```

## Memory Trick

> **Container = Temporary Compute**
>
> **Volume = Persistent Data**
>
> **Named Volume = Docker manages storage**
>
> **Bind Mount = You manage storage**
