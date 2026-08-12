# Kubernetes 

## 1. Why Kubernetes?

You have applications running in containers.

With a small application:

```text
Container 1
Container 2
Container 3
```

You can manage them manually or with Docker Compose.

But as your system grows, you may have:

```text
Many services
Many containers
Many machines
Many replicas
```

Now you need something that can automatically:

* run applications
* scale them
* distribute them
* restart failed workloads
* maintain the state you asked for

That's where Kubernetes comes in. The PDF describes Kubernetes around deployment, scaling, management, self-healing, and load balancing. 

---

# 2. Cluster

A **Kubernetes Cluster** is the whole Kubernetes environment.

```text
             CLUSTER
                │
       ┌────────┴────────┐
       │                 │
 Control Plane      Worker Nodes
```

A cluster can contain multiple Nodes.

There is **no universal fixed number** of Nodes.

---

# 3. Node

A **Node is a machine** participating in the Kubernetes cluster.

```text
Cluster
 ├── Node 1
 ├── Node 2
 └── Node 3
```

Worker Nodes are where your application workloads actually run.

---

# 4. Pod

Kubernetes uses **Pods** as its basic workload unit.

For your microservices mental model:

```text
Order Service
     ↓
   Pod
     ↓
Container
```

Usually:

> **One microservice instance → one Pod → one application container**

But a Pod **can contain multiple containers** when those containers need to be tightly coupled.

---

# 5. Image vs Container vs Pod

This was your major confusion, so lock this in:

```text
Dockerfile
    ↓
 Docker Image
    ↓
 Container
```

Kubernetes doesn't convert an existing container into a Pod.

Instead:

```text
Container Image
      ↓
 Kubernetes Pod
      ↓
 Container
```

And if you want three instances:

```text
             Image
               │
       ┌───────┼───────┐
       ↓       ↓       ↓
     Pod 1   Pod 2   Pod 3
       ↓       ↓       ↓
   Container Container Container
```

---

# 6. Docker Compose

Compose is useful for running multiple services/containers together.

```text
docker-compose.yml
        │
   ┌────┼────┐
   ↓    ↓    ↓
 User Order Inventory
   │    │    │
   ↓    ↓    ↓
Container Container Container
   └────┼────┘
        ↓
   Shared Network
```

Compose doesn't create **one giant image containing all services**.

Each service can have its own image.

And Compose can either:

```text
image: existing-image
```

or:

```text
build: ./service
```

where it builds an image.

---

# 7. Kubernetes Architecture

Now the big picture:

```text
                 CLUSTER
                    │
          ┌─────────┴─────────┐
          │                   │
     CONTROL PLANE       WORKER NODES
       "Brain"             "Workers"
```

The Control Plane manages the cluster.

Worker Nodes run the applications.

---

# 8. Control Plane

The main components we learned:

```text
              CONTROL PLANE
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
   API Server     etcd       Scheduler
       │
       ↓
 Controller Manager
```

### API Server

The **front door**.

You use:

```text
kubectl
   ↓
API Server
```

It receives requests and provides communication between Kubernetes components. 

---

### etcd

Think:

> **Kubernetes' persistent state/configuration store.**

It is a key-value store containing Kubernetes cluster state/configuration. 

Don't confuse it with your application's MySQL database.

---

### Scheduler

Think:

> **"Where should this Pod run?"**

Example:

```text
Node 1 → busy
Node 2 → available
Node 3 → busy

        ↓

Scheduler chooses Node 2
```

That's its basic role. 

---

### Controller Manager

Think:

> **"Is reality still matching what the developer asked for?"**

You want:

```text
3 Pods
```

Reality:

```text
2 Pods
```

Controller mechanisms work toward restoring:

```text
3 Pods
```

That's the desired-state idea described in the PDF. 

---

# 9. Worker Node

Once the Scheduler chooses a Node:

```text
Scheduler
    ↓
Node 1
```

Inside that Worker Node, we discussed:

```text
Worker Node
│
├── kubelet
│
├── Container Runtime
│
└── Pods
      ↓
   Containers
```

`kubelet` is the node-level agent that makes sure the assigned Pods are running.

The container runtime actually runs the containers.

**Important:** the PDF itself only labels "Worker Nodes"; these internal details were our additional conceptual teaching, not content explicitly explained in the PDF. 

---

# 10. The entire flow

This is the **one flow you should remember**:

```text
Developer
    │
    │ "I want 3 Order Service instances"
    ↓
  kubectl
    ↓
API Server
    ↓
Control Plane
    │
    ├── etcd
    │     └── stores cluster state
    │
    ├── Scheduler
    │     └── chooses suitable Nodes
    │
    └── Controllers
          └── maintain desired state
                    │
                    ↓
               Worker Node
                    │
                  kubelet
                    │
             Container Runtime
                    │
                   Pod
                    │
                Container
                    │
             Order Service
```

Then Kubernetes keeps watching:

```text
DESIRED STATE
      ↕
ACTUAL STATE
```

If they differ, Kubernetes works toward correcting the difference.

---

# One final mental picture

If someone asks you:

**"What is Kubernetes?"**

You should now be able to explain it naturally:

> **Kubernetes is a system for managing containerized applications across a cluster of machines. I tell Kubernetes what state I want, the Control Plane processes that desired state and decides where workloads should run, Worker Nodes run the Pods containing my containers, and Kubernetes continuously monitors the system and works to keep the actual state matching the desired state.**

That's the foundation.

**Next topic after this should be `Deployment`**, because now you'll naturally ask:

> "Okay, who actually tells Kubernetes that I want 3 Pods of my Order Service?"
