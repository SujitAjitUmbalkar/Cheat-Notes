
**VS Code:** Kubernetes extension → create `.yml` → use YAML autocomplete/snippets → fill `apiVersion → kind → metadata → spec`.


# 1. Pod YAML

```yaml
apiVersion: v1                    # Kubernetes API version for Pod
kind: Pod                         # Resource type

metadata:
  name: my-nginx-pod              # Unique name of this Pod
  labels:
    app: myapp                    # Label attached to the Pod

spec:
  containers:
    - name: myapp                 # Container name inside the Pod
      image: nginx:latest         # Docker image used for the container

      ports:
        - containerPort: 80       # Port on which Nginx listens inside the Pod

      resources:
        limits:
          memory: "128Mi"         # Maximum memory container can use
          cpu: "500m"             # Maximum CPU = 0.5 CPU
```

### Quick notes

* Pod directly runs the container.
* Standalone Pod does **not** maintain replicas.
* If Pod dies → Kubernetes does not automatically create another Pod.
* `metadata.name` identifies the Pod.

### Run & test

```cmd
kubectl apply -f pod.yml
```

Check:

```cmd
kubectl get pods
```

Detailed:

```cmd
kubectl describe pod my-nginx-pod
```

Run the same YAML again:

```cmd
kubectl apply -f pod.yml
```

Result:

```text
Already exists → no second Pod
```

If you want another Pod, you need another resource/name or use a ReplicaSet/Deployment.

---

# 2. ReplicaSet YAML

```yaml
apiVersion: apps/v1                # API version for ReplicaSet
kind: ReplicaSet                   # Resource type

metadata:
  name: my-replica-set             # Name of the ReplicaSet
  labels:
    app: myapp                     # Label of the ReplicaSet

spec:
  replicas: 3                      # ReplicaSet must maintain 3 Pods

  selector:
    matchLabels:
      app: myapp                   # Manage Pods having app=myapp

  template:                        # Blueprint used to create Pods
    metadata:
      labels:
        app: myapp                 # These labels MUST match selector

    spec:
      containers:
        - name: myapp              # Container inside each Pod
          image: nginx:latest      # Image used by every Pod
          ports:
            - containerPort: 80    # Application/container port
```

### Quick notes

* ReplicaSet maintains the desired number of Pods.
* `replicas: 3` → maintains **3 Pods**.
* If one Pod dies → ReplicaSet creates a replacement.
* `3 → 5` → 2 Pods added.
* `5 → 2` → 3 Pods removed.
* Changing the Pod template does **not** automatically replace existing Pods.

### Run & test

```cmd
kubectl apply -f replicaset.yml
```

Check ReplicaSet:

```cmd
kubectl get replicasets
```

Check Pods:

```cmd
kubectl get pods
```

See which Pods belong to it:

```cmd
kubectl get pods -l app=myapp
```

Detailed:

```cmd
kubectl describe rs my-replica-set
```

### Test replica behavior

Change:

```yaml
replicas: 3
```

to:

```yaml
replicas: 5
```

Then:

```cmd
kubectl apply -f replicaset.yml
kubectl get pods
```

You should see **5 Pods**.

Delete one:

```cmd
kubectl delete pod <pod-name>
```

Then:

```cmd
kubectl get pods
```

ReplicaSet automatically creates a replacement → back to **5 Pods**.

---

# 3. Deployment YAML

```yaml
apiVersion: apps/v1                # API version for Deployment
kind: Deployment                   # Resource type

metadata:
  name: myapp-deployment            # Name of the Deployment
  namespace: prod                   # Create Deployment inside prod namespace

spec:
  replicas: 3                       # Desired number of Pods

  selector:
    matchLabels:
      app: myapp                    # Deployment manages Pods with this label

  template:                         # Blueprint for the Pods
    metadata:
      labels:
        app: myapp                  # Must match selector

    spec:
      containers:
        - name: myapp               # Container name
          image: nginx:latest       # Container image/version

          ports:
            - containerPort: 80     # Port used by application

          resources:
            limits:
              memory: "128Mi"       # Maximum memory
              cpu: "500m"           # Maximum CPU
```

### Quick notes

Deployment creates/manages:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

* `replicas: 3` → maintains 3 Pods.
* Pod dies → replacement is created.
* `3 → 5` → creates 2 more Pods.
* `5 → 2` → removes 3 Pods.
* Change image/version → **new ReplicaSet + rollout**.
* Deployment is normally used for applications.

### Run & test

Because your YAML uses `namespace: prod`:

```cmd
kubectl apply -f deployment.yml
```

Check:

```cmd
kubectl get deployments -n prod
```

Check ReplicaSets:

```cmd
kubectl get replicasets -n prod
```

Check Pods:

```cmd
kubectl get pods -n prod
```

See the whole relationship:

```cmd
kubectl get all -n prod
```

---

## Important Deployment cases

### Case 1 — Run same YAML again

```cmd
kubectl apply -f deployment.yml
```

Again:

```cmd
kubectl apply -f deployment.yml
```

Result:

```text
Same Deployment
Same ReplicaSet
Same desired number of Pods
```

No duplicate Deployment/Pods.

---

### Case 2 — Change replicas

```yaml
replicas: 3
```

→

```yaml
replicas: 5
```

```cmd
kubectl apply -f deployment.yml
kubectl get pods -n prod
```

Result → **5 Pods**.

---

### Case 3 — Change image/version

```yaml
image: nginx:1.25
```

→

```yaml
image: nginx:1.26
```

Run:

```cmd
kubectl apply -f deployment.yml
```

Check:

```cmd
kubectl get replicasets -n prod
kubectl get pods -n prod
```

Deployment creates a **new ReplicaSet** and rolls out the new Pods.

Check rollout:

```cmd
kubectl rollout status deployment/myapp-deployment -n prod
```

---

### Case 4 — Check rollout history

```cmd
kubectl rollout history deployment/myapp-deployment -n prod
```

Rollback if needed:

```cmd
kubectl rollout undo deployment/myapp-deployment -n prod
```

---

# Final cheat sheet

```text
Pod
 ↓
Runs container
 ↓
No replica management


ReplicaSet
 ↓
Maintains N Pods
 ↓
Pod dies → replacement


Deployment
 ↓
Manages ReplicaSet
 ↓
Maintains N Pods
 ↓
Image/config change → new ReplicaSet + rollout
```

### Main commands to remember

```cmd
kubectl apply -f file.yml

kub..
kubectl get pods
kubectl get replicasets
kubectl get deployments

kubectl get all -n prod

kubectl describe pod <pod-name>
kubectl describe rs <rs-name>

kubectl rollout status deployment/<deployment-name> -n prod
kubectl rollout history deployment/<deployment-name> -n prod
kubectl rollout undo deployment/<deployment-name> -n prod
```
