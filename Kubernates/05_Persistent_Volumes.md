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
# 18. Static Provisioning vs Dynamic Provisioning

Before StorageClass, understand one simple question:

> **Who creates the PV?**

There are two approaches.

---

## 18.1 Static Provisioning

In **static provisioning**, the administrator/developer creates the PV manually.

Flow:

```text
Actual Storage
      ↓
     PV
      ↓
     PVC
      ↓
     Pod
```

You manually write:

```yaml
kind: PersistentVolume
```

Then:

```yaml
kind: PersistentVolumeClaim
```

### Example

You create:

```text
mypv
```

Then:

```text
mypvc
```

Then:

```text
Pod
 ↓
mypvc
 ↓
mypv
 ↓
Storage
```

This is what we did with our `hostPath` example.

---

## 18.2 Dynamic Provisioning

In **dynamic provisioning**, you don't manually create the PV.

Instead, you create a:

```text
PVC
```

and tell Kubernetes what kind of storage you want using:

```text
StorageClass
```

Kubernetes then uses a **provisioner/storage driver** to create the required storage and PV automatically.

Flow:

```text
PVC
 ↓
StorageClass
 ↓
Provisioner / CSI Driver
 ↓
Actual Storage
 ↓
PV automatically created
 ↓
PVC becomes Bound
 ↓
Pod uses PVC
```

This is the important difference:

|                  | Static                   | Dynamic                       |
| ---------------- | ------------------------ | ----------------------------- |
| PVC              | You create               | You create                    |
| PV               | You create               | Kubernetes creates            |
| StorageClass     | Not required             | Used                          |
| Storage creation | Manual/pre-existing      | Automatic                     |
| Good for         | Learning/special storage | Cloud/production environments |

---

# 19. Why Do We Need Dynamic Provisioning?

Imagine a company has 500 applications.

With static provisioning:

```text
Application 1 → PV 1
Application 2 → PV 2
Application 3 → PV 3
...
Application 500 → PV 500
```

Someone has to manually create and manage hundreds of PVs.

That's inconvenient.

With dynamic provisioning:

```text
Application
     ↓
    PVC
     ↓
StorageClass
     ↓
Kubernetes creates storage
     ↓
PV automatically created
```

The developer simply says:

> "I need 20Gi of this type of storage."

The infrastructure handles the provisioning.

---

# 20. What is a StorageClass?

A **StorageClass defines a category/configuration of storage**.

Think of it like choosing a storage plan.

For example:

```text
standard
fast-ssd
premium
```

Different StorageClasses can represent different:

* Storage providers
* Performance levels
* Storage types
* Configuration parameters
* Provisioners

The reference describes StorageClass as a way to specify different types of storage and their configurations, while provisioners manage the actual storage creation.

---

# 21. StorageClass Mental Model

Suppose you have:

```text
fast-storage
```

A developer creates:

```text
PVC
10Gi
fast-storage
```

Kubernetes sees:

```text
"I need 10Gi of fast-storage."
```

Then:

```text
PVC
 ↓
StorageClass: fast-storage
 ↓
Provisioner
 ↓
Create actual storage
 ↓
Create PV
 ↓
Bind PV ↔ PVC
```

The developer doesn't need to manually create the PV.

---

# 22. What is a Provisioner?

The **provisioner** is responsible for creating/provisioning the actual storage required by a StorageClass.

Conceptually:

```text
StorageClass
      ↓
Provisioner
      ↓
Storage Provider
```

Examples from the reference include storage systems associated with:

```text
AWS EBS
Google Persistent Disk
Azure Disk
NFS
Local storage
```

The PDF lists these as examples of provisioners/storage integrations.

In modern Kubernetes environments, cloud storage is commonly accessed through **CSI drivers**.

---

# 23. StorageClass Configuration

A simplified StorageClass looks like:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-storage

provisioner: <storage-provider-driver>

parameters:
  type: <storage-type>

reclaimPolicy: Delete

volumeBindingMode: WaitForFirstConsumer
```

### Important fields

```yaml
metadata:
  name: fast-storage
```

The name of the StorageClass.

---

```yaml
provisioner: <storage-provider-driver>
```

Tells Kubernetes which storage provisioner/driver should create the storage.

---

```yaml
parameters:
  type: <storage-type>
```

Provider-specific storage configuration.

---

```yaml
reclaimPolicy: Delete
```

Controls what happens to the dynamically provisioned storage when its PVC is deleted.

---

```yaml
volumeBindingMode: WaitForFirstConsumer
```

Controls when the volume should be provisioned/bound in environments where Pod scheduling matters.

Don't memorize the provider-specific parameters yet.

The important mental model is:

```text
StorageClass
     ↓
"What kind of storage should I create?"
```

---

# 24. Dynamic PVC

Now comes the important part.

With static provisioning, we explicitly used:

```yaml
volumeName: mypv
```

Example:

```yaml
spec:
  volumeName: mypv
```

This says:

> "Bind this PVC to this particular PV."

With dynamic provisioning, we normally **don't specify `volumeName`**.

Instead:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mypvc

spec:
  storageClassName: fast-storage

  resources:
    requests:
      storage: 10Gi

  accessModes:
    - ReadWriteOnce
```

Notice:

```yaml
storageClassName: fast-storage
```

but:

```yaml
volumeName:
```

is absent.

This is the important point demonstrated by the reference material.

---

# 25. What Happens When We Apply This PVC?

Run:

```cmd
kubectl apply -f pvc.yaml
```

You created only:

```text
PVC
```

But Kubernetes sees:

```text
storageClassName: fast-storage
```

So it starts the dynamic provisioning process.

```text
             PVC
              │
              │
              ↓
       StorageClass
              │
              ↓
        Provisioner
              │
              ↓
       Actual Storage
              │
              ↓
         PV created
              │
              ↓
       PVC ↔ PV Bound
```

So unlike static provisioning:

```text
YOU create PV
```

dynamic provisioning does:

```text
KUBERNETES creates PV
```

---

# 26. Static vs Dynamic — Side-by-Side

## Static

You have existing storage.

```text
Existing Storage
      ↓
You create PV
      ↓
You create PVC
      ↓
PVC binds to PV
      ↓
Pod
```

Example:

```yaml
volumeName: mypv
```

---

## Dynamic

You request storage.

```text
You create PVC
      ↓
StorageClass
      ↓
Provisioner
      ↓
Storage created
      ↓
PV created
      ↓
PVC binds to PV
      ↓
Pod
```

No manual PV YAML is required.

---

# 27. How Does the Pod Use Dynamically Created Storage?

The Pod doesn't care whether the PV was created manually or dynamically.

This is extremely important.

The Pod still does:

```yaml
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: mypvc
```

and:

```yaml
volumeMounts:
  - name: data
    mountPath: /data
```

The Pod only knows:

```text
mypvc
```

It doesn't need to know:

```text
Which PV?
Which disk?
Which cloud?
Which CSI driver?
```

That is infrastructure's responsibility.

---

# 28. Dynamic Provisioning — Complete Example

### StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-storage

provisioner: <storage-provider-driver>

parameters:
  type: <storage-type>

reclaimPolicy: Delete
```

### PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mypvc

spec:
  storageClassName: fast-storage

  resources:
    requests:
      storage: 10Gi

  accessModes:
    - ReadWriteOnce
```

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: myapp

spec:
  replicas: 1

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
          image: nginx

          volumeMounts:
            - name: data
              mountPath: /data

      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: mypvc
```

Notice the chain:

```text
Deployment
    ↓
PVC
    ↓
StorageClass
    ↓
Provisioner
    ↓
Storage
    ↓
PV
```

---

# 29. How to Validate Dynamic Provisioning

First see what StorageClasses are available:

```cmd
kubectl get storageclass
```

Example:

```text
NAME       PROVISIONER
standard   ...
```

---

### Create PVC

```cmd
kubectl apply -f pvc.yaml
```

Then:

```cmd
kubectl get pvc
```

You want:

```text
NAME    STATUS   VOLUME
mypvc   Bound    pvc-xxxxx
```

Now check PV:

```cmd
kubectl get pv
```

You should see a PV that you **didn't manually create**:

```text
NAME                                      STATUS   CLAIM
pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  Bound    default/mypvc
```

That's the proof that dynamic provisioning happened.

---

# 30. Test the Actual Application Storage

Create the Deployment:

```cmd
kubectl apply -f Deployment.yml
```

Check:

```cmd
kubectl get pods
```

Enter:

```cmd
kubectl exec -it <pod-name> -- bash
```

Check mount:

```bash
ls /data
```

Create data:

```bash
echo "persistent data" > /data/test.txt
```

Verify:

```bash
cat /data/test.txt
```

Expected:

```text
persistent data
```

---

# 31. Test Persistence

Now exit:

```bash
exit
```

Delete the Pod:

```cmd
kubectl delete pod <pod-name>
```

The Deployment creates a new Pod.

Check:

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

Expected:

```text
persistent data
```

This proves:

```text
Old Pod
  ↓
Data written to persistent storage
  ↓
Old Pod deleted
  ↓
New Pod
  ↓
Same PVC
  ↓
Same persistent storage
  ↓
Data still exists
```

---

# 32. PV/PVC Lifecycle

Now understand what happens throughout the life of a PV/PVC.

The reference describes four major stages:

```text
Provisioning
      ↓
Binding
      ↓
Using
      ↓
Reclaiming
```

---

## Stage 1 — Provisioning

Storage is made available.

### Static

You create:

```text
PV
```

### Dynamic

StorageClass/provisioner creates the storage and PV.

```text
PVC
 ↓
StorageClass
 ↓
Provisioner
 ↓
PV
```

---

# 33. Stage 2 — Binding

Kubernetes finds a compatible PV for the PVC.

Example:

```text
PV:
10Gi
RWO

PVC:
5Gi
RWO
```

If compatible:

```text
PV
 ↕
PVC
```

Status becomes:

```text
Bound
```

Check:

```cmd
kubectl get pv
kubectl get pvc
```

---

# 34. Stage 3 — Using

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

The application can now write:

```text
/data/orders.txt
/data/images/
```

etc.

---

# 35. Stage 4 — Reclaiming

Suppose:

```cmd
kubectl delete pvc mypvc
```

Now the PVC is gone.

What happens to the PV/storage depends on the **reclaim policy**.

### Retain

```text
PVC deleted
     ↓
PV retained
     ↓
Data can be preserved
```

Good when data is important.

---

### Delete

```text
PVC deleted
     ↓
PV/storage is deleted
```

Useful when the storage should disappear along with the claim.

---

### Recycle

You may see this in older tutorials:

```yaml
persistentVolumeReclaimPolicy: Recycle
```

Do **not** use it for modern Kubernetes learning based on your current cluster.

It is deprecated/removed behavior in modern Kubernetes.

Use:

```yaml
persistentVolumeReclaimPolicy: Retain
```

or:

```yaml
persistentVolumeReclaimPolicy: Delete
```

depending on the desired behavior.

---

# 36. Access Modes

Now another important question:

> **Who can mount this storage, and with what permissions?**

Kubernetes has three traditional access modes:

```text
RWO
ROX
RWX
```

The reference defines them as ReadWriteOnce, ReadOnlyMany and ReadWriteMany.

---

## RWO — ReadWriteOnce

```yaml
accessModes:
  - ReadWriteOnce
```

Meaning:

> Storage can be mounted read/write by a single node.

Think:

```text
Node 1
 ├── Pod A ──┐
 └── Pod B ──┼──→ RWO Storage
             │
             WRITE
```

Good for:

* Databases
* Single-instance applications
* Storage that doesn't need multi-node shared writes

### Important

RWO does **not** simply mean:

> "Only one Pod."

It means:

> "Read/write access from one node."

Multiple Pods on that node may be able to use it, depending on the storage implementation.

---

# 37. ROX — ReadOnlyMany

```yaml
accessModes:
  - ReadOnlyMany
```

Meaning:

> Multiple nodes can mount the volume, but as read-only.

```text
Node 1 ──┐
Node 2 ──┼──→ Storage
Node 3 ──┘
             READ ONLY
```

Useful when many applications need to read the same data but shouldn't modify it.

---

# 38. RWX — ReadWriteMany

```yaml
accessModes:
  - ReadWriteMany
```

Meaning:

> Multiple nodes can mount the storage for reading and writing.

```text
Node 1 ──┐
Node 2 ──┼──→ Shared Storage
Node 3 ──┘
          READ + WRITE
```

Good for:

* Shared files
* NFS
* Shared application data
* Some logging/data aggregation systems

But:

> **The storage backend must support RWX.**

You cannot make an RWO-only backend into RWX just by changing the YAML.

---

# 39. Access Mode Comparison

| Mode | Read | Write | Multiple Nodes |
| ---- | ---- | ----- | -------------- |
| RWO  | ✅    | ✅     | ❌              |
| ROX  | ✅    | ❌     | ✅              |
| RWX  | ✅    | ✅     | ✅              |

Think:

```text
RWO = One node can read/write

ROX = Many nodes can read

RWX = Many nodes can read/write
```

---

# 40. Choosing Access Mode

### Database

Usually:

```text
RWO
```

because a database commonly uses block storage attached to one node.

### Shared files

Usually:

```text
RWX
```

if multiple Pods/nodes must modify the same files.

### Read-only shared data

Potentially:

```text
ROX
```

if the backend supports it.

But don't choose the access mode first and then search for storage.

Instead:

```text
Application requirement
       ↓
Required access pattern
       ↓
Storage backend that supports it
       ↓
Access mode
```

---

# 41. One More Important Concept — Storage Location

Different PV types put the actual data in different places.

### hostPath

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Kubernetes Node
 ↓
/tmp/demo-pv
```

### NFS

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Network
 ↓
NFS Server
 ↓
/shared/data
```

### CSI cloud storage

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
CSI Driver
 ↓
Cloud Storage
```

This is one of the biggest differences between PV types.

---

# 42. Final Decision Tree

When you need persistent storage, think in this order:

```text
1. Does my application need persistent data?
             ↓
            YES
             ↓
2. What kind of storage do I need?
             ↓
     ┌───────┼────────┐
     ↓       ↓        ↓
  Local    Shared    Cloud
     │       │        │
 hostPath   NFS      CSI
```

Then ask:

```text
3. Who should create the PV?
```

If manually:

```text
Static Provisioning
```

If automatically:

```text
StorageClass
     ↓
Dynamic Provisioning
```

Then ask:

```text
4. How should Pods access it?
```

```text
RWO
ROX
RWX
```

Then:

```text
5. What should happen when PVC is deleted?
```

```text
Retain
or
Delete
```

---

# 43. Complete Kubernetes Storage Mental Model

This is the final model you should remember:

```text
                         APPLICATION
                              │
                              ↓
                             POD
                              │
                       volumeMounts
                              │
                              ↓
                             PVC
                   "I need persistent storage"
                              │
               ┌──────────────┴──────────────┐
               │                             │
          Static PV                    StorageClass
               │                             │
               │                       Provisioner/CSI
               │                             │
               │                             ↓
               │                       Actual Storage
               │                             │
               │                             ↓
               │                        PV created
               │                             │
               └──────────────┬──────────────┘
                              ↓
                             PV
                              │
                ┌─────────────┼─────────────┐
                ↓             ↓             ↓
             hostPath        NFS            CSI
                ↓             ↓             ↓
             Node FS      NFS Server    Cloud Storage
```

### The entire story in one sentence:

> **A Pod consumes a PVC; the PVC gets storage from a PV; the PV connects to an actual storage backend; and StorageClass + dynamic provisioning can automatically create that storage/PV when needed.**

---

# 44. Commands You Should Actually Remember

### PV/PVC

```cmd
kubectl get pv
kubectl get pvc
kubectl describe pv <pv-name>
kubectl describe pvc <pvc-name>
```

### StorageClass

```cmd
kubectl get storageclass
kubectl describe storageclass <name>
```

### Pod storage testing

```cmd
kubectl get pods
kubectl exec -it <pod-name> -- bash
```

Inside:

```bash
ls /data
echo "hello" > /data/test.txt
cat /data/test.txt
```

### Persistence test

```cmd
kubectl delete pod <pod-name>
kubectl get pods
kubectl exec -it <new-pod-name> -- bash
```

Inside:

```bash
cat /data/test.txt
```

If the data is still there:

```text
Pod recreated
     ↓
PVC still exists
     ↓
PV still exists
     ↓
Storage still exists
     ↓
DATA SURVIVED
```

---

# 45. What You Should Know for Interviews/Real Projects

You should be able to explain these without notes:

1. Why do we need PV?
2. Difference between normal Volume and PV.
3. PV vs PVC.
4. Why Pod uses PVC instead of directly using PV.
5. `hostPath` vs NFS vs CSI.
6. Where the actual data lives for each.
7. RWO vs ROX vs RWX.
8. Static vs dynamic provisioning.
9. What StorageClass does.
10. What a provisioner/CSI driver does.
11. How dynamic provisioning creates a PV automatically.
12. PV/PVC lifecycle.
13. Retain vs Delete.
14. How to verify that a Pod is actually using persistent storage.
15. How to prove data survives Pod deletion/recreation.

The core sequence to keep in your head is:

```text
PV/PVC basics
      ↓
Storage backend types
      ↓
Mount PVC into Pod
      ↓
Test persistence
      ↓
PV/PVC lifecycle
      ↓
Access Modes
      ↓
Static Provisioning
      ↓
StorageClass
      ↓
Dynamic Provisioning
      ↓
CSI / Production Storage
```
