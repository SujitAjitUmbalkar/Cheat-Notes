
# Kubernetes Important Commands

## 1. Pods

| Command                                    | Usage                             |
| ------------------------------------------ | --------------------------------- |
| `kubectl run my-pod --image=nginx`         | Quickly create a Pod              |
| `kubectl get pods`                         | List Pods in current namespace    |
| `kubectl get pods -n <namespace>`          | List Pods in a specific namespace |
| `kubectl get pods -A`                      | List Pods in all namespaces       |
| `kubectl get pods -o wide`                 | Show Pods with IP, Node, etc.     |
| `kubectl get pods -l app=myapp`            | Find Pods using a label           |
| `kubectl describe pod <pod-name>`          | Detailed Pod information          |
| `kubectl logs <pod-name>`                  | View Pod/container logs           |
| `kubectl exec -it <pod-name> -- /bin/bash` | Open shell inside Pod             |
| `kubectl delete pod <pod-name>`            | Delete a Pod                      |
| `kubectl apply -f pod.yaml`                | Create/update Pod from YAML       |

The PDF specifically covers `run`, `get`, `describe`, `logs`, `exec`, `delete`, and `apply`. 

---

## 2. ReplicaSets

| Command                                   | Usage                               |
| ----------------------------------------- | ----------------------------------- |
| `kubectl get replicasets`                 | List ReplicaSets                    |
| `kubectl get rs`                          | Short version                       |
| `kubectl get rs -n <namespace>`           | ReplicaSets in a namespace          |
| `kubectl describe rs <rs-name>`           | Detailed ReplicaSet information     |
| `kubectl get pods -l app=myapp`           | See Pods selected by the ReplicaSet |
| `kubectl scale rs <rs-name> --replicas=5` | Change number of replicas           |
| `kubectl delete rs <rs-name>`             | Delete ReplicaSet                   |
| `kubectl apply -f replicaset.yaml`        | Create/update from YAML             |

### Important

```cmd
kubectl get rs
kubectl get pods -l app=myapp
kubectl describe rs <rs-name>
```

A ReplicaSet maintains the required number of Pods; if a Pod is terminated, it creates a replacement. 

---

# 3. Deployments

The PDF covers creating, scaling, changing images, rollback, and rollout history. 

| Command                                                   | Usage                           |
| --------------------------------------------------------- | ------------------------------- |
| `kubectl get deployments`                                 | List Deployments                |
| `kubectl get deploy`                                      | Short version                   |
| `kubectl get deploy -n <namespace>`                       | Deployments in a namespace      |
| `kubectl describe deploy <name>`                          | Detailed Deployment information |
| `kubectl create deploy <name> --replicas=3 --image=nginx` | Create Deployment from command  |
| `kubectl scale deployment <name> --replicas=5`            | Scale Deployment                |
| `kubectl set image deployment/<name> <container>=<image>` | Update container image          |
| `kubectl rollout status deployment/<name>`                | Check rollout progress          |
| `kubectl rollout history deployment/<name>`               | View rollout history            |
| `kubectl rollout undo deployment/<name>`                  | Roll back to previous revision  |
| `kubectl rollout undo deployment/<name> --to-revision=2`  | Roll back to specific revision  |
| `kubectl delete deployment <name>`                        | Delete Deployment               |
| `kubectl apply -f deployment.yaml`                        | Create/update from YAML         |

### Example

```cmd
kubectl set image deployment/myapp-deployment myapp=nginx:1.27
```

Then:

```cmd
kubectl rollout status deployment/myapp-deployment
```

---

# 4. Namespaces

| Command                                                   | Usage                                     |
| --------------------------------------------------------- | ----------------------------------------- |
| `kubectl get namespaces`                                  | List all namespaces                       |
| `kubectl get ns`                                          | Short version                             |
| `kubectl create namespace <name>`                         | Create namespace                          |
| `kubectl delete namespace <name>`                         | Delete namespace                          |
| `kubectl get pods -n <name>`                              | Get Pods from namespace                   |
| `kubectl config set-context --current --namespace=<name>` | Set default namespace for current context |
| `kubectl get pods -A`                                     | Get Pods from all namespaces              |
| `kubectl describe namespace <name>`                       | Namespace details                         |
| `kubectl edit namespace <name>`                           | Edit namespace                            |
| `kubectl get all -A`                                      | Get common resources from all namespaces  |

These namespace-management commands are included in your PDF. 

---

# 5. YAML / Declarative Commands

| Command                         | Usage                               |
| ------------------------------- | ----------------------------------- |
| `kubectl apply -f <file>.yaml`  | Create or update resource from YAML |
| `kubectl delete -f <file>.yaml` | Delete resource defined in YAML     |
| `kubectl get -f <file>.yaml`    | View resource defined in YAML       |

Example:

```cmd
kubectl apply -f deployment.yaml
```

Your PDF uses `kubectl apply` as the declarative configuration approach. 

---

# ⭐ Commands to Memorize First

| Command                                                 | Remember it as              |
| ------------------------------------------------------- | --------------------------- |
| `kubectl get pods`                                      | **See Pods**                |
| `kubectl get rs`                                        | **See ReplicaSets**         |
| `kubectl get deploy`                                    | **See Deployments**         |
| `kubectl get ns`                                        | **See Namespaces**          |
| `kubectl describe <resource> <name>`                    | **Investigate resource**    |
| `kubectl logs <pod>`                                    | **See application logs**    |
| `kubectl exec -it <pod> -- /bin/bash`                   | **Enter Pod**               |
| `kubectl apply -f file.yaml`                            | **Create/update from YAML** |
| `kubectl delete <resource> <name>`                      | **Delete resource**         |
| `kubectl scale deployment <name> --replicas=5`          | **Change replicas**         |
| `kubectl set image deployment/<name> ...`               | **Change image**            |
| `kubectl rollout status deployment/<name>`              | **Check update**            |
| `kubectl rollout history deployment/<name>`             | **See versions**            |
| `kubectl rollout undo deployment/<name>`                | **Rollback**                |
| `kubectl get pods -n prod`                              | **Pods in prod**            |
| `kubectl get pods -A`                                   | **Pods everywhere**         |
| `kubectl config set-context --current --namespace=prod` | **Make prod default**       |

### Easy mental model

```text
get        → See
describe   → Investigate
logs       → Application output
exec       → Enter container
apply      → Create / Update
delete     → Remove
scale      → Change replicas
set image  → Change version
rollout    → Manage Deployment updates
-n         → Specific namespace
-A         → All namespaces
-l         → Filter by label
```
