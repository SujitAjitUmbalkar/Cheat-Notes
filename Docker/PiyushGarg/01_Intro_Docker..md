# 🐳 Docker Cheat Notes (0 → 100)

---

# 1. Why Docker?

## Problems Before Docker

* ❌ "Works on my machine" problem.
* ❌ Different Java/Python/Node versions.
* ❌ Missing dependencies.
* ❌ Manual server setup.
* ❌ Difficult deployment and scaling.

## Docker Solution

* Packages everything together:

  * Application
  * Dependencies
  * Runtime
  * Configuration
* Runs the same everywhere.

> **One Line:** Docker solves the **environment inconsistency ("Works on my machine")** problem.

---

# 2. What is Docker?

> **Docker is a platform that creates and runs lightweight containers.**

Container = Application + Dependencies

---

# 3. What is a Container?

A **Container** is an isolated environment that contains:

* Application
* Required libraries
* Runtime (Java, Python, Node, etc.)
* Configuration

**It DOES NOT contain an Operating System.**

---

# 4. Docker Architecture

```text
Application
      │
Container
(App + Dependencies)
      │
Docker Engine
      │
Host OS Kernel (Shared)
      │
Hardware
```

---

# 5. Biggest Concept (Most Important)

### Containers DO NOT have their own Kernel.

They **share the Host OS Kernel.**

```
Container
      │
Shared Host Kernel
```

Remember:

> **Container ≠ Operating System**

---

# 6. Why is Docker Lightweight?

No separate OS.

Only:

* Application
* Libraries

Shared:

* Host Kernel

Result:

* Less RAM
* Less Storage
* Faster Startup

---

# 7. OS Compatibility

### Linux Host + Linux Container

✅ Works

Same Linux Kernel.

---

### Windows Host + Windows Container

✅ Works

Same Windows Kernel.

---

### Linux Host + Windows Container

❌ Doesn't Work

Needs Windows Kernel.

---

### Windows Host + Linux Container

❌ Doesn't Work Directly

Needs Linux Kernel.

---

# 8. Then How Does Docker Desktop Run Linux Containers on Windows?

It creates a **small Linux Virtual Machine (WSL2 / Hyper-V).**

```
Windows
     │
Linux VM (WSL2)
     │
Linux Kernel
     │
Docker Engine
     │
Containers
```

So containers actually run **inside Linux VM**, not directly on Windows.

---

# 9. Docker vs Virtualization

| Docker             | Virtual Machine      |
| ------------------ | -------------------- |
| Container          | Virtual Machine (VM) |
| Shares Host Kernel | Own Kernel           |
| No Guest OS        | Has Guest OS         |
| Lightweight        | Heavyweight          |
| Starts in Seconds  | Starts in Minutes    |
| Low RAM            | High RAM             |
| MB Size            | GB Size              |

---

# 10. What is Virtualization?

Virtualization creates a **Virtual Machine (VM).**

A VM is a **complete computer** having:

* Operating System
* Kernel
* Applications

```
VM
├── Ubuntu OS
├── Linux Kernel
├── Java
└── Spring Boot App
```

---

# 11. VM vs Container

### Docker

```
Container
├── Application
├── Dependencies
└── Shared Host Kernel
```

### Virtual Machine

```
VM
├── Operating System
├── Kernel
├── Application
└── Dependencies
```

---

# 12. Can One VM Have Multiple Applications?

✅ Yes.

A VM is just like a normal computer.

Example:

```
Ubuntu VM
├── MySQL
├── Nginx
├── Java
└── Spring Boot
```

Usually in production:

**1 VM → 1 Main Application** (Best Practice)

---

# 13. How Do We Store an Application in a VM?

Exactly like a normal computer.

Steps:

1. Create VM
2. Install OS
3. Install Java / Python / etc.
4. Copy Application
5. Run Application

---

# 14. Main Difference (Interview)

### Docker

> Package the application and run it inside a **Container** sharing the Host OS Kernel.

### Virtualization

> Create a complete **Virtual Machine** with its own Operating System and Kernel.

---

# 15. Memory Trick

### 🐳 Docker

```
Host
 ├── Container
 ├── Container
 └── Container
```

Containers share one kernel.

---

### 🖥️ Virtualization

```
Host
 ├── VM (Windows)
 ├── VM (Ubuntu)
 └── VM (CentOS)
```

Each VM has its own OS + Kernel.

---

# 16. Interview One-Liners

### Docker

> Docker packages an application with its dependencies into a lightweight container that runs consistently across environments.

### Container

> A container is an isolated environment containing an application and its dependencies, sharing the host OS kernel.

### Virtual Machine

> A VM is a complete computer with its own operating system and kernel.

### Biggest Difference

> **Docker shares the Host Kernel, whereas a Virtual Machine has its own Kernel.**

### Golden Rule ⭐

* 🐳 **Container = Application**
* 🖥️ **VM = Complete Computer**
* 🔑 **Docker shares the Host Kernel; Virtualization provides a separate Kernel for every VM.**
