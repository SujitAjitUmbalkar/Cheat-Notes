Docker Images vs Containers
---

# 🐳 Step 1: Docker Terminology

## Docker Image 📦

An **Image** is a **read-only blueprint/template**.

It contains:

* Operating system files (Ubuntu, Alpine, etc.)
* Runtime (Java, Python...)
* Libraries
* Application (optional)

**Think:**

> Image = Class (Java)

You cannot change an image while running.

---

## Docker Container 📦➡️🏃

A **Container** is a **running instance of an Image**.

**Think:**

> Container = Object (Java)

One image can create many containers.

Example

```text
Ubuntu Image
     │
 ┌───┴────┐
 │        │
Container1 Container2
```

---

# Step 2: Image vs Container

| Image                      | Container              |
| -------------------------- | ---------------------- |
| Blueprint                  | Running Instance       |
| Read-only                  | Writable               |
| Static                     | Running                |
| Can create many containers | Created from one image |

---

# Step 3: Docker Flow

```text
Docker Hub
      │
 docker pull
      │
 Ubuntu Image
      │
 docker run
      │
 Ubuntu Container
```

---

# Step 4: What happened in your command?

You executed

```bash
docker run -it ubuntu
```

Docker internally did:

### Step 1

Look locally

```text
Is Ubuntu image available?
```

No

↓

### Step 2

Downloaded image

```text
docker pull ubuntu
```

↓

### Step 3

Created Container

↓

### Step 4

Started Container

↓

### Step 5

Attached Terminal

---

# Step 5: Image Lifecycle

```text
Docker Hub
      │
docker pull
      │
Image
      │
docker run
      │
Container
```

---

# Step 6: Container Lifecycle

```text
Created
   │
Running
   │
Stopped
   │
Started Again
   │
Removed
```

---

# Step 7: Most Used Commands

## Images

See images

```bash
docker images
```

Download image

```bash
docker pull ubuntu
```

Delete image

```bash
docker rmi ubuntu
```

---

## Containers

Run

```bash
docker run ubuntu
```

Interactive

```bash
docker run -it ubuntu
```

Show running

```bash
docker ps
```

Show all

```bash
docker ps -a
```

Stop

```bash
docker stop <container-id>
```

Start again

```bash
docker start <container-id>
```

Open terminal

```bash
docker exec -it <container-id> bash
```

Delete

```bash
docker rm <container-id>
```

---

# Step 8: Behaviour

## Image

✔ Immutable

Cannot change

Reusable

Can create multiple containers

---

## Container

✔ Mutable

Can create files

Can delete files

Can install software

Can stop/start

Can be deleted

---

# Step 9: Important Case

Create file

```bash
touch notes.txt
```

Container

```text
Ubuntu Image
      │
Container
      │
notes.txt
```

Image **does NOT** contain notes.txt.

Only that container has it.

Another container won't.

---

# Step 10: What happens after deleting Container?

Suppose

```bash
touch hello.txt
```

Delete container

```bash
docker rm container1
```

Create new container

```bash
docker run ubuntu
```

Result

```text
hello.txt ❌ Gone
```

Because files belonged to container.

Image never changed.

---

# Step 11: Important Directories

```text
/
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
```

Remember only these:

### /

Root Directory

Everything starts here.

---

### /home

Contains home directories of normal users.

Example

```text
/home/ubuntu
/home/jeetu
```

---

### /root

Home directory of root user.

---

### ~

Shortcut of current user's home.

For root

```text
~
=
/root
```

---

# Step 12: Commands inside Container

Current directory

```bash
pwd
```

List

```bash
ls
```

Go home

```bash
cd
```

Go root directory

```bash
cd /
```

Go one folder up

```bash
cd ..
```

Create file

```bash
touch file.txt
```

Create folder

```bash
mkdir demo
```

Delete file

```bash
rm file.txt
```

Delete folder

```bash
rm -r demo
```

---

# Step 13: One Image → Multiple Containers

```text
Ubuntu Image
      │
 ┌────┼────┐
 │    │    │
 C1   C2   C3
```

Each container has its **own writable filesystem**.

Changing one container doesn't affect the others.

---

# Step 14: Real Interview Questions

### Image?

> A Docker Image is a read-only blueprint used to create containers.

---

### Container?

> A Docker Container is a running instance of an image with its own writable layer.

---

### Difference?

> Image is a template; Container is its running instance.

---

### Why is Docker lightweight?

> Because containers share the Host OS kernel instead of including a full operating system.

---

### Why multiple containers from one image?

> Images are reusable templates. Each container gets its own isolated writable layer while sharing the same underlying image.

---

# ⭐ Final Memory Map

```text
Docker Hub
      │
docker pull
      │
Image (Blueprint)
      │
docker run
      │
Container (Running Instance)
      │
Files • Processes • Commands
      │
docker stop
      │
Stopped
      │
docker start
      │
Running
      │
docker rm
      │
Deleted
```

If you understand these concepts thoroughly, you'll have a strong foundation for the next topics: **Dockerfile → Volumes → Networking → Docker Compose → Running Spring Boot applications in Docker**.
