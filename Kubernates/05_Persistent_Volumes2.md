# Dynamic Provisioning in Kubernetes

---

## 1. What is Dynamic Provisioning?

**Dynamic Provisioning** means Kubernetes automatically creates a PV when a PVC requests storage through a `StorageClass`.

In static provisioning:

```text
Admin creates PV 
      ↓ 
PVC 
      ↓ 
Pod 

```

In dynamic provisioning:

```text
PVC 
 ↓ 
StorageClass 
 ↓ 
Provisioner 
 ↓ 
Storage is created 
 ↓ 
PV is created automatically 
 ↓ 
PVC becomes Bound 
 ↓ 
Pod uses PVC 

```

The key idea is:

> **We create the PVC, not the PV. Kubernetes provisions the PV automatically.**

---

# 2. Static vs Dynamic Provisioning

| Static                       | Dynamic                         |
| ---------------------------- | ------------------------------- |
| PV is manually created       | PV is automatically created     |
| Admin prepares storage first | PVC triggers storage creation   |
| More manual work             | Less manual work                |
| PVC uses an existing PV      | PVC gets a newly provisioned PV |

**Remember:**

```text
Static  → PV → PVC → Pod 
 
Dynamic → PVC → StorageClass → PV → Pod 

```

---

# 3. StorageClass

A **StorageClass** tells Kubernetes:

> "When a PVC asks for this type of storage, use this provisioner/configuration."

Example:

```yaml
apiVersion: storage.k8s.io/v1 
kind: StorageClass 
 
metadata: 
  name: fast-storage 
 
provisioner: example.com/storage   # Example only; real provisioner depends on cluster 

```

In real environments, the provisioner is connected to a storage system/provider. The uploaded material describes StorageClasses as specifying storage types/configuration and provisioners that manage the actual storage.

### Important: Docker Desktop Provisioner

For your **Docker Desktop Kubernetes cluster**, Docker Desktop provides a storage provisioner. A commonly used provisioner is:

```text
docker.io/hostpath
```

So when you create your own StorageClass on Docker Desktop, you can use:

```yaml
provisioner: docker.io/hostpath
```

This is **specific to the Docker Desktop environment**.

In other Kubernetes environments, such as GKE, AWS, or Azure, the provisioner will be different because the underlying storage system is different.

---

# 4. Create a Valid StorageClass

For your **Docker Desktop Kubernetes cluster**, we can create a StorageClass using Docker Desktop's `docker.io/hostpath` provisioner.

Save this as:

```text
Storage-Class.yml

```

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-storage

provisioner: docker.io/hostpath   # Docker Desktop storage provisioner

```

### Apply the StorageClass

```cmd
kubectl apply -f Storage-Class.yml

```

Expected:

```text
storageclass.storage.k8s.io/fast-storage created

```

### Validate

```cmd
kubectl get storageclass

```

or:

```cmd
kubectl get sc

```

You should see something similar to:

```text
NAME                     PROVISIONER
fast-storage             docker.io/hostpath

```

To see details:

```cmd
kubectl describe storageclass fast-storage

```

### Important

The StorageClass itself does **not create a PV immediately**.

```text
StorageClass created
       ↓
No PV yet
       ↓
PVC requests storage
       ↓
Provisioner creates storage
       ↓
PV is created

```

---

# 5. Check Existing StorageClasses

```cmd
kubectl get storageclass 

```

or:

```cmd
kubectl get sc 

```

To see details:

```cmd
kubectl describe storageclass <storage-class-name> 

```

Example:

```cmd
kubectl describe storageclass standard 

```

---

# 6. Create a PVC for Dynamic Provisioning

The important part is:

```yaml
storageClassName: fast-storage 

```

Example:

```yaml
apiVersion: v1 
kind: PersistentVolumeClaim 
 
metadata: 
  name: dynamic-pvc 
 
spec: 
  storageClassName: fast-storage   # Ask this StorageClass to provision storage 
 
  accessModes: 
    - ReadWriteOnce 
 
  resources: 
    requests: 
      storage: 1Gi             # Request 1Gi of storage 

```

### Important

Notice that we **do not create a PV YAML**.

```text
PVC → StorageClass → Kubernetes creates PV automatically 

```

---

# 7. Apply the PVC

Save it as:

```text
Dynamic-Pvc.yml 

```

Then:

```cmd
kubectl apply -f Dynamic-Pvc.yml 

```

Expected:

```text
persistentvolumeclaim/dynamic-pvc created 

```

---

# 8. Validate the PVC

```cmd
kubectl get pvc 

```

Initially, you may see:

```text
NAME           STATUS    VOLUME   CAPACITY 
dynamic-pvc    Pending 

```

After provisioning succeeds:

```text
NAME           STATUS   VOLUME       CAPACITY   STORAGECLASS 
dynamic-pvc    Bound    pvc-xxxxx    1Gi        fast-storage 

```

### Important

`Bound` means:

```text
PVC successfully connected to a PV 

```

---

# 9. Check the Automatically Created PV

Run:

```cmd
kubectl get pv 

```

You should see something like:

```text
NAME        CAPACITY   STATUS   CLAIM                   STORAGECLASS 
pvc-xxxxx   1Gi        Bound    default/dynamic-pvc     fast-storage 

```

Notice:

```text
PVC 
dynamic-pvc 
    ↓ 
PV 
pvc-xxxxx 

```

The PV was created **automatically**.

---

# 10. Verify the Complete Binding

You can inspect both sides:

```cmd
kubectl get pvc dynamic-pvc 

```

and:

```cmd
kubectl get pv 

```

Or:

```cmd
kubectl describe pvc dynamic-pvc 

```

Check:

```text
StorageClass 
Volume 
Status 
Events 

```

The expected relationship is:

```text
StorageClass: fast-storage 
 
       ↓ 
 
PVC: dynamic-pvc 
 
       ↓ 
 
PV: pvc-xxxxx 
 
       ↓ 
 
Actual storage 

```

---

# 11. Use the PVC in a Deployment

Now create a Deployment that uses the PVC.

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: storage-test-deployment

spec:
  replicas: 1

  selector:
    matchLabels:
      app: storage-test

  template:
    metadata:
      labels:
        app: storage-test

    spec:
      containers:
        - name: nginx
          image: nginx

          volumeMounts:
            - name: data
              mountPath: /data

      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: dynamic-pvc   # Connect Deployment Pods to the PVC

```

Save it as:

```text
Deployment.yml

```

Apply:

```cmd
kubectl apply -f Deployment.yml

```

Expected:

```text
deployment.apps/storage-test-deployment created

```

### Validate the Deployment

```cmd
kubectl get deployment

```

Then:

```cmd
kubectl get pods

```

Expected:

```text
NAME                                      READY   STATUS
storage-test-deployment-xxxxx-xxxxx       1/1     Running

```

You can check the Pod's volume:

```cmd
kubectl describe pod <pod-name>

```

Look under:

```text
Volumes
Mounts

```

---

# 12. Test That Storage Actually Works

Enter the Pod:

```cmd
kubectl exec -it <pod-name> -- sh

```

Create a file:

```bash
echo "Hello Kubernetes" > /data/test.txt 

```

Check it:

```bash
cat /data/test.txt 

```

Expected:

```text
Hello Kubernetes 

```

Exit:

```bash
exit 

```

---

# 13. Test Persistence

Delete the Pod:

```cmd
kubectl delete pod <pod-name> 

```

Because this Pod is managed by a **Deployment**, Kubernetes automatically creates a replacement Pod.

Check:

```cmd
kubectl get pods

```

Wait until the new Pod is `Running`.

Then enter the **new Pod**:

```cmd
kubectl exec -it <new-pod-name> -- sh

```

Check:

```bash
cat /data/test.txt 

```

Expected:

```text
Hello Kubernetes 

```

### Why is the data still there?

Because:

```text
Pod deleted 
   ↓ 
PVC still exists 
   ↓ 
PV still exists 
   ↓ 
Actual storage still exists 
   ↓ 
New Pod mounts the same PVC 
   ↓ 
Old data available ✅ 

```

---

# 14. What if Multiple PVs Exist?

### Static provisioning

Kubernetes looks for a **suitable existing PV** based on the PVC's requirements.

```text
PVC 
 ↓ 
Find matching available PV 
 ↓ 
Bind 

```

### Dynamic provisioning

The PVC uses its StorageClass to request **new storage**.

```text
PVC-A → StorageClass → New PV-A 
 
PVC-B → StorageClass → New PV-B 

```

So you don't need to worry about Kubernetes randomly choosing one of several PVs during normal dynamic provisioning.

---

# 15. What if Pod Needs Last Month's Data?

A **new dynamically provisioned PV gives you new storage**. It does not automatically contain old data.

For old data, you need the existing storage:

```text
Old data 
   ↓ 
Existing PV 
   ↓ 
Existing PVC 
   ↓ 
New Pod 

```

The Pod should use the existing PVC:

```yaml
volumes: 
  - name: data 
    persistentVolumeClaim: 
      claimName: old-pvc 

```

Then:

```text
New Pod 
   ↓ 
Existing PVC 
   ↓ 
Existing PV 
   ↓ 
Existing storage 
   ↓ 
Last month's data 

```

If the PVC was deleted, whether the data can still be recovered depends on the PV's reclaim policy and the underlying storage.

---

# 16. Complete Testing Checklist

For dynamic provisioning, remember this practical sequence:

```text
1. Create StorageClass
   ↓ 
kubectl apply -f Storage-Class.yml

2. Check StorageClass 
   ↓ 
kubectl get sc 
 
3. Create PVC 
   ↓ 
kubectl apply -f Dynamic-Pvc.yml 
 
4. Check PVC 
   ↓ 
kubectl get pvc 
 
5. Check automatically created PV 
   ↓ 
kubectl get pv 
 
6. Check details/events 
   ↓ 
kubectl describe pvc dynamic-pvc 
 
7. Create Deployment using PVC 
   ↓ 
kubectl apply -f Deployment.yml 
 
8. Check Pods 
   ↓ 
kubectl get pods 
 
9. Write data 
   ↓ 
kubectl exec -it <pod-name> -- sh 
 
10. Delete Pod and let Deployment recreate it

11. Check whether the data is still present 

```

### Final mental model

```text
                 Dynamic Provisioning 
 
Developer 
    ↓ 
  PVC 
    ↓ 
StorageClass 
    ↓ 
Provisioner 
    ↓ 
Actual storage created 
    ↓ 
PV automatically created 
    ↓ 
PVC ←→ PV 
    ↓ 
  Pod 
    ↓ 
Uses persistent data 

```

**Most important commands:**

```cmd
kubectl apply -f Storage-Class.yml
kubectl get sc
kubectl apply -f Dynamic-Pvc.yml 
kubectl get pvc 
kubectl get pv 
kubectl describe pvc dynamic-pvc 
kubectl apply -f Deployment.yml 
kubectl get pods 
kubectl exec -it <pod-name> -- sh 

```

This gives you the full **create → provision → bind → use → test persistence** workflow.
