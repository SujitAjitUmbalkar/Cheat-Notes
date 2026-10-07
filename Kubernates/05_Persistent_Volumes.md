# Kubernetes Persistent Volumes (PV)

---

# 1. Why do we need Persistent Storage?

Containers and Pods are **temporary**.

If a Pod is deleted:

```text
Pod deleted
    ↓
Container deleted
    ↓
Container's temporary data ❌
```

Example:

```text
MySQL
Order Service
Inventory Service
Uploaded Images
User Files
Logs
```

We don't want important data to disappear whenever a Pod is recreated.

So Kubernetes provides **Persistent Volumes**.

### Requirements of persistent storage

We generally want storage that:

1. Does not depend on the Pod lifecycle.
2. Can remain available independently of a particular Pod.
3. Can survive Pod recreation and, depending on the storage backend, cluster/node failures.

These are the storage requirements highlighted in the reference material.

---

# 2. Normal Volume vs Persistent Volume

## Normal Volume

A normal Kubernetes Volume belongs to the Pod.

```text
Pod
 │
 └── Volume
```

Its lifecycle is tied to the Pod.

The data may disappear when the Pod is removed, depending on the volume type.

---

## Persistent Volume

A Persistent Volume is designed to separate **storage lifecycle** from **Pod lifecycle**.

```text
Pod
 │
 ↓
PVC
 │
 ↓
PV
 │
 ↓
Actual Storage
```

Now:

```text
Pod deleted
     ↓
New Pod created
     ↓
Same PVC
     ↓
Same persistent storage
     ↓
Data still exists
```

This is the most important mental model for PVs.

---

# 3. PV, PVC and Actual Storage

There are three different things.

## PV — PersistentVolume

A **PV is a Kubernetes storage resource**.

It describes storage that Kubernetes can provide to Pods.

Example:

```yaml
kind: PersistentVolume
metadata:
  name: mypv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce
```

Think:

> **PV = storage available to Kubernetes.**

---

## PVC — PersistentVolumeClaim

A **PVC is an application's request for storage**.

Example:

```yaml
kind: PersistentVolumeClaim

metadata:
  name: mypvc

spec:
  resources:
    requests:
      storage: 1Gi

  accessModes:
    - ReadWriteOnce
```

Think:

> **PVC = application asking Kubernetes for storage.**

---

## Actual Storage

The PV must ultimately connect to some actual storage backend.

Examples:

```text
PV
 ↓
hostPath
 ↓
Node's filesystem
```

or:

```text
PV
 ↓
NFS
 ↓
NFS Server
```

or:

```text
PV
 ↓
CSI Driver
 ↓
AWS EBS / Azure Disk / other storage
```

So:

```text
Application
    ↓
   Pod
    ↓
   PVC
    ↓
    PV
    ↓
Storage Backend
```

---

# 4. The Most Important Rule

A **Pod does not directly use a PV**.

The normal relationship is:

```text
Pod → PVC → PV → Storage
```

The Pod references the **PVC**.

The PVC binds to the **PV**.

The PV connects to the actual storage.

---

# 5. PV Type 1 — hostPath

## What is hostPath?

`hostPath` uses a directory on the Kubernetes node's filesystem.

Example:

```yaml
hostPath:
  path: /tmp/demo-pv
```

The actual data exists on the node:

```text
Kubernetes Node
│
└── /tmp/demo-pv
      ├── test.txt
      ├── orders.txt
      └── ...
```

The reference material recommends `hostPath` for testing/development rather than production because it does not provide replication or dynamic provisioning.

---

## Where does the data live?

On the **Kubernetes node's filesystem**.

For example:

```text
PV
 ↓
hostPath
 ↓
Node
 ↓
/tmp/demo-pv
```

Important:

> `1Gi` in the PV YAML does NOT automatically create a separate 1Gi disk.

It is the capacity advertised by the PV.

---

## When should I use hostPath?

Good for:

* Learning Kubernetes
* Local development
* Testing PV/PVC
* Simple Docker Desktop/Minikube experiments

Avoid for:

* Production databases
* Multi-node production applications
* Highly available storage

Why?

Because the data belongs to that node.

If your Pod moves to another node:

```text
Node 1
/tmp/demo-pv
      ↓
     DATA

Pod moves → Node 2

Node 2
/tmp/demo-pv
      ↓
     ??? 
```

Node 2 does not automatically have Node 1's data.

---

## hostPath PV configuration

### PV

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: mypv

spec:
  capacity:
    storage: 1Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Delete

  hostPath:
    path: /tmp/demo-pv
```

### PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mypvc

spec:
  resources:
    requests:
      storage: 1Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteOnce

  volumeName: mypv
```

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: myapp-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: myapp

  template:
    metadata:
      labels:
        app: myapp

    spec:
      containers:
        - name: myapp
          image: nginx:latest

          volumeMounts:
            - name: data
              mountPath: /data

      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: mypvc
```

Notice:

```yaml
claimName: mypvc
```

The Pod references the **PVC**, not the PV.

---

# 6. hostPath — How to Validate

### Step 1 — Create PV

```cmd
kubectl apply -f My-Pv.yml
```

### Step 2 — Create PVC

```cmd
kubectl apply -f My-Pvc.yml
```

### Step 3 — Verify binding

```cmd
kubectl get pv
kubectl get pvc
```

Expected:

```text
PV     Bound
PVC    Bound
```

Example:

```text
mypv     1Gi   RWO   Bound   default/mypvc
```

---

### Step 4 — Create Deployment

```cmd
kubectl apply -f Deployment.yml
```

Check:

```cmd
kubectl get pods
```

---

### Step 5 — Enter Pod

```cmd
kubectl exec -it <pod-name> -- bash
```

Inside:

```bash
ls /data
```

The `/data` directory should exist.

---

### Step 6 — Write data

```bash
echo "hello kubernetes" > /data/test.txt
```

Check:

```bash
cat /data/test.txt
```

Output:

```text
hello kubernetes
```

---

### Step 7 — Delete the Pod

Exit:

```bash
exit
```

Delete the Pod:

```cmd
kubectl delete pod <pod-name>
```

Deployment creates a replacement Pod.

```cmd
kubectl get pods
```

Enter the new Pod:

```cmd
kubectl exec -it <new-pod-name> -- bash
```

Check:

```bash
cat /data/test.txt
```

If you still get:

```text
hello kubernetes
```

you have successfully verified **persistent storage**.

---

# 7. PV Type 2 — NFS

## What is NFS?

NFS = **Network File System**.

Instead of storing data on the Kubernetes node, storage is provided by a **central NFS server**.

```text
                NFS Server
                    │
             /shared-storage
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
        Node 1    Node 2    Node 3
          │         │         │
         Pods      Pods      Pods
```

The reference describes NFS as centralized storage that can be shared by different clients over the network.

---

## Where does the data live?

On the **NFS server**.

Not inside the Pod.

Not necessarily on the Kubernetes node.

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
NFS
 ↓
NFS Server
 ↓
/shared/data
```

---

## Why use NFS?

NFS is useful when multiple Pods/nodes need access to the **same files**.

For example:

```text
Pod 1 ──┐
Pod 2 ──┼──→ NFS shared storage
Pod 3 ──┘
```

This is particularly useful for shared file storage.

---

## NFS vs hostPath

| Feature                     | hostPath        | NFS        |
| --------------------------- | --------------- | ---------- |
| Storage location            | Node filesystem | NFS server |
| Shared across nodes         | ❌ No            | ✅ Yes      |
| Good for learning           | ✅               | ✅          |
| Production                  | Usually ❌       | ✅ Possible |
| Network storage             | ❌               | ✅          |
| Multiple Pods sharing files | Limited         | ✅          |
| Node dependency             | High            | Lower      |

---

## NFS PV configuration

Assume:

```text
NFS Server: 192.168.1.100
NFS Path:   /shared/k8s
```

### PV

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: nfs-pv

spec:
  capacity:
    storage: 10Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteMany

  persistentVolumeReclaimPolicy: Delete

  nfs:
    server: 192.168.1.100
    path: /shared/k8s
```

### PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: nfs-pvc

spec:
  resources:
    requests:
      storage: 10Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteMany

  volumeName: nfs-pv
```

### Pod/Deployment

```yaml
volumes:
  - name: shared-data
    persistentVolumeClaim:
      claimName: nfs-pvc
```

Mount it:

```yaml
volumeMounts:
  - name: shared-data
    mountPath: /data
```

---

## NFS validation

Check:

```cmd
kubectl get pv
kubectl get pvc
```

Expected:

```text
nfs-pv     Bound
nfs-pvc    Bound
```

Then:

```cmd
kubectl get pods
```

Enter:

```cmd
kubectl exec -it <pod-name> -- bash
```

Test:

```bash
echo "NFS test" > /data/test.txt
cat /data/test.txt
```

If another Pod can access the same:

```bash
cat /data/test.txt
```

then both Pods are accessing the same shared NFS storage.

---

# 8. PV Type 3 — CSI Storage

## What is CSI?

CSI = **Container Storage Interface**.

CSI provides a standard way for Kubernetes to communicate with external storage systems.

Think:

```text
Kubernetes
    ↓
CSI Driver
    ↓
Storage Provider
    ↓
Actual Storage
```

The reference specifically describes CSI as the integration mechanism for storage providers such as Amazon EBS and Azure Disk.

---

## Why do we need CSI?

Cloud providers have different storage systems.

For example:

```text
AWS    → EBS
Azure  → Azure Disk
Google → Persistent Disk
```

Kubernetes shouldn't need completely different storage logic for every provider.

CSI provides a standard interface.

---

## Where does the data live?

With CSI, the actual storage is managed by the storage provider.

For example:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
CSI Driver
 ↓
AWS EBS
```

or:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
CSI Driver
 ↓
Azure Disk
```

---

## When should I choose CSI?

For cloud production environments, **CSI-backed storage is usually the normal choice**.

Examples:

```text
AWS EKS
    ↓
EBS CSI
    ↓
EBS Volume
```

```text
Azure AKS
    ↓
Azure Disk CSI
    ↓
Azure Disk
```

Use it when you need:

* Cloud-managed storage
* Persistent application/database storage
* Dynamic provisioning
* Production-grade storage
* Storage integrated with cloud infrastructure

---

# 9. CSI PV Configuration

The exact YAML depends on the CSI driver.

A generic CSI PV looks like:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: csi-pv

spec:
  capacity:
    storage: 10Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Delete

  csi:
    driver: <csi-driver-name>
    volumeHandle: <volume-id>
```

The values:

```yaml
driver:
volumeHandle:
```

are **provider/driver-specific**.

So don't blindly copy a CSI configuration from AWS into Azure or another cluster.

---

## CSI PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: csi-pvc

spec:
  resources:
    requests:
      storage: 10Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteOnce

  volumeName: csi-pv
```

Then a Deployment uses:

```yaml
volumes:
  - name: app-storage
    persistentVolumeClaim:
      claimName: csi-pvc
```

and:

```yaml
volumeMounts:
  - name: app-storage
    mountPath: /data
```

---

# 10. Which Storage Should I Choose?

Use this simple decision:

```text
Need storage?
     │
     ├── Just learning/testing locally?
     │       ↓
     │    hostPath
     │
     ├── Need shared filesystem across nodes?
     │       ↓
     │      NFS / suitable RWX storage
     │
     └── Cloud production application?
             ↓
        CSI-backed storage
```

### Practical choice

| Situation                    | Recommended                               |
| ---------------------------- | ----------------------------------------- |
| Kubernetes learning          | hostPath                                  |
| Local development            | hostPath                                  |
| Shared files between nodes   | NFS / RWX-capable storage                 |
| AWS production               | EBS via CSI, depending on workload        |
| Azure production             | Azure Disk via CSI, depending on workload |
| Production database          | CSI-backed storage generally              |
| Multi-node shared filesystem | NFS or another RWX-capable backend        |

**Important:** Storage choice depends not only on the application but also on required access mode, performance, failure model, and provider support.

---

# 11. Access Modes

Kubernetes defines three important access modes:

```text
RWO
ROX
RWX
```

The reference defines these as ReadWriteOnce, ReadOnlyMany and ReadWriteMany.

---

## RWO — ReadWriteOnce

```text
Read + Write
      ↓
Single Node
```

```yaml
accessModes:
  - ReadWriteOnce
```

Useful for:

* Single-instance applications
* Many database workloads
* Storage where one node should mount it read/write

Example:

```text
Order DB Pod
     ↓
   PVC
     ↓
   PV
```

### Important

RWO does **not** mean:

> Only one Pod can ever use the volume.

It means the volume is mounted read/write by **one node**.

Multiple Pods on that node may potentially use it, depending on the storage implementation.

---

# 12. ROX — ReadOnlyMany

```text
Many Nodes
   ↓
Read Only
```

```yaml
accessModes:
  - ReadOnlyMany
```

Example:

```text
Node 1 ──┐
Node 2 ──┼──→ Storage
Node 3 ──┘       ↑
                 READ ONLY
```

Useful when many Pods need to read the same data but should not modify it.

Example:

```text
Shared configuration
Static files
Read-only datasets
```

The reference gives read replicas as one possible example.

---

# 13. RWX — ReadWriteMany

```text
Many Nodes
    ↓
Read + Write
```

```yaml
accessModes:
  - ReadWriteMany
```

Example:

```text
Pod 1 ──┐
Pod 2 ──┼──→ Shared Storage
Pod 3 ──┘
```

Useful for:

* Shared files
* NFS
* Logging/data aggregation
* Applications requiring shared filesystem access

The storage backend must actually support RWX.

Simply writing:

```yaml
accessModes:
  - ReadWriteMany
```

doesn't magically make a storage backend support it.

---

# 14. PV/PVC Lifecycle

A simplified lifecycle is:

```text
Provisioning
     ↓
Binding
     ↓
Using
     ↓
Reclaiming
```

This is the lifecycle described in the reference.

---

## 14.1 Provisioning

Storage is made available.

For static provisioning:

```text
Developer/Admin
      ↓
Creates PV
      ↓
PV available
```

Example:

```cmd
kubectl apply -f My-Pv.yml
```

Check:

```cmd
kubectl get pv
```

---

# 15. Binding

Now a PVC requests storage.

```text
PVC
 ↓
Find compatible PV
 ↓
Bind
```

Example:

```cmd
kubectl apply -f My-Pvc.yml
```

Check:

```cmd
kubectl get pv
kubectl get pvc
```

Expected:

```text
PV     Bound
PVC    Bound
```

A PV is bound to **one PVC at a time**.

---

# 16. Using

A Pod consumes the PVC.

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage
```

Deployment:

```yaml
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: mypvc
```

Mount:

```yaml
volumeMounts:
  - name: data
    mountPath: /data
```

Now the application can use:

```text
/data
```

---

# 17. Reclaiming

When the PVC is deleted:

```text
PVC deleted
     ↓
PV is no longer bound
     ↓
Reclaim policy determines what happens
```

Common reclaim policies you should know:

### Retain

```text
PVC deleted
    ↓
PV retained
    ↓
Data is preserved
```

Useful when you don't want accidental deletion of important data.

### Delete

```text
PVC deleted
    ↓
Storage/PV is deleted
```

Useful for dynamically provisioned temporary/application storage when the provider supports deletion.

### Recycle

You may see `Recycle` in older learning material.

It is **deprecated/not the policy to use for modern Kubernetes setups**.

For your current Kubernetes learning environment, prefer:

```yaml
persistentVolumeReclaimPolicy: Delete
```

or:

```yaml
persistentVolumeReclaimPolicy: Retain
```

depending on the desired data lifecycle.

---
