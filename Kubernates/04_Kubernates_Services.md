# Kubernetes Services

## 1. Introduction

A **Service** provides a stable network endpoint for a group of Pods.

Pods are **ephemeral**:

* Pods can be deleted/recreated.
* Their IP addresses can change.
* Clients should not communicate directly with Pod IPs.

A Service solves this by providing a **stable endpoint** and routing traffic to matching Pods.

```text
Client
   ↓
Service
   ↓
Pod  Pod  Pod
```

### Why Services are needed

* Stable endpoint for Pods
* Service discovery
* Load balancing across matching Pods
* Allows communication between microservices
* Pods can change without changing the Service endpoint

A Service selects Pods using **labels**, not Pod names/IPs.

```yaml
selector:
  app: order
```

It sends traffic to Pods having:

```yaml
labels:
  app: order
```

---

# 2. Service Ports

A Service commonly involves these ports:

| Port         | Meaning                                                     |
| ------------ | ----------------------------------------------------------- |
| `port`       | Port exposed by the **Service**                             |
| `targetPort` | Port where the **Pod/application is listening**             |
| `nodePort`   | Port exposed on the Kubernetes **Node**; used with NodePort |

### Example

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Traffic:

```text
Service :80
     ↓
Pod :8080
```

For NodePort:

```yaml
ports:
  - port: 80
    targetPort: 8080
    nodePort: 30001
```

Traffic:

```text
Node :30001
     ↓
Service :80
     ↓
Pod :8080
```

**Remember:**

```text
nodePort → port → targetPort
  Node      Service     Pod
```

---

# 3. Service Types

Kubernetes provides three commonly used Service types:

1. `ClusterIP`
2. `NodePort`
3. `LoadBalancer`

---

## 3.1 ClusterIP

### What is it?

`ClusterIP` exposes the Service **inside the Kubernetes cluster**.

It is the **default Service type**.

```text
Application A
     ↓
Service B
 (ClusterIP)
     ↓
Pods B
```

### Use cases

Use ClusterIP when:

* One microservice needs to communicate with another.
* The application should **not be directly exposed to the Internet**.
* You want internal service discovery.

Typical microservice architecture:

```text
API Gateway
    ↓
Order Service (ClusterIP)
    ↓
Order Pods
    ↓
Inventory Service (ClusterIP)
    ↓
Inventory Pods
```

### Configuration

```yaml
apiVersion: v1
kind: Service

metadata:
  name: order-service

spec:
  type: ClusterIP       # Internal cluster access

  selector:
    app: order           # Selects Pods having app=order

  ports:
    - port: 80           # Service port
      targetPort: 8080   # Application/Pod port
```

### Validate

```cmd
kubectl apply -f order-service.yaml

kubectl get svc

kubectl describe svc order-service
```

Check the `Selector` and `Endpoints`.

```cmd
kubectl get endpoints order-service
```

Test service DNS **from inside the cluster**:

```text
http://order-service
```

Full DNS:

```text
order-service.<namespace>.svc.cluster.local
```

---

# 3.2 NodePort

### What is it?

`NodePort` exposes a Service through a port on each Kubernetes Node.

```text
External Client
      ↓
NodeIP:30001
      ↓
Service
      ↓
Pods
```

NodePort range:

```text
30000 - 32767
```

### Use cases

Commonly used for:

* Development
* Testing
* Learning Kubernetes
* Simple external access

It is generally not the preferred public entry point for a production application when a proper external load-balancing solution is available.

### Configuration

```yaml
apiVersion: v1
kind: Service

metadata:
  name: order-service

spec:
  type: NodePort

  selector:
    app: order             # Routes to Pods with app=order

  ports:
    - port: 80             # Service port
      targetPort: 8080     # Pod/application port
      nodePort: 30001      # External Node port
```

Traffic:

```text
Client
  ↓
NodeIP:30001
  ↓
Service:80
  ↓
Pod:8080
```

### Validate

```cmd
kubectl apply -f order-service.yaml

kubectl get svc

kubectl describe svc order-service

kubectl get nodes -o wide
```

Look at:

```text
PORT(S)
80:30001/TCP
```

Then access the NodePort using the appropriate Node address for your Kubernetes environment.

**Docker Desktop note:** don't use:

```cmd
minikube service order-service
```

because that command is Minikube-specific.

---

# 3.3 LoadBalancer

### What is it?

`LoadBalancer` exposes a Service externally using an **external load-balancing mechanism provided by the underlying infrastructure/cloud environment**.

```text
Internet
    ↓
External Load Balancer
    ↓
Kubernetes Service
    ↓
Pods
```

The external Load Balancer is **infrastructure**, not your API Gateway.

Your API Gateway is an application.

### Use cases

Use it when:

* An application needs external/public access.
* You want an external load-balancing entry point.
* Your cloud/Kubernetes environment supports LoadBalancer Services.

Typical architecture:

```text
Internet
    ↓
LoadBalancer Service
    ↓
API Gateway Pods
    ↓
Order Service (ClusterIP)
    ↓
Order Pods
```

Usually, you expose the **API Gateway**, not every microservice.

### Configuration

```yaml
apiVersion: v1
kind: Service

metadata:
  name: api-gateway

spec:
  type: LoadBalancer       # Request external LB

  selector:
    app: api-gateway       # Select Gateway Pods

  ports:
    - port: 80             # External Service port
      targetPort: 8080     # Gateway container port
```

### Validate

```cmd
kubectl apply -f api-gateway-service.yaml

kubectl get svc

kubectl describe svc api-gateway
```

Check:

```text
EXTERNAL-IP
```

If your environment/cloud provisions one, the external address will appear there.

**Docker Desktop:** `LoadBalancer` behavior may differ from a cloud Kubernetes environment, so don't assume it will provide a real public cloud load balancer locally.

---

# 4. Service Commands

### Most frequently used

| Command                                                                          | Purpose                      |
| -------------------------------------------------------------------------------- | ---------------------------- |
| `kubectl get svc`                                                                | List Services                |
| `kubectl get svc -A`                                                             | Services in all namespaces   |
| `kubectl describe svc <name>`                                                    | Detailed Service information |
| `kubectl get endpoints <name>`                                                   | See backend endpoints        |
| `kubectl get endpointslices`                                                     | View EndpointSlices          |
| `kubectl apply -f service.yaml`                                                  | Create/update Service        |
| `kubectl delete svc <name>`                                                      | Delete Service               |
| `kubectl edit svc <name>`                                                        | Edit existing Service        |
| `kubectl get nodes -o wide`                                                      | See Node addresses           |
| `kubectl expose deployment <name> --port=80 --target-port=8080 --type=ClusterIP` | Quickly create a Service     |

---

# 5. Service Configuration — Things to Check

When writing a Service YAML, always check these:

### ① Selector must match Pod labels

Pod:

```yaml
labels:
  app: order
```

Service:

```yaml
selector:
  app: order
```

If they don't match:

```text
Service
   ↓
No matching Pods ❌
```

---

### ② targetPort must match the application port

If your application listens on:

```text
8080
```

use:

```yaml
targetPort: 8080
```

---

### ③ Choose the correct Service type

```text
Internal communication
        ↓
    ClusterIP

External testing/simple access
        ↓
     NodePort

External/public access
        ↓
   LoadBalancer
```

---

### ④ NodePort must be valid

For NodePort:

```text
30000 - 32767
```

Example:

```yaml
nodePort: 30001
```

---

### ⑤ Verify the Service has endpoints

```cmd
kubectl get endpoints <service-name>
```

If there are no endpoints, first check:

```cmd
kubectl get pods --show-labels
```

Then compare the Pod labels with the Service selector.

---

# 6. Validate the Complete Service

After creating a Service:

### Step 1 — Service exists

```cmd
kubectl get svc
```

### Step 2 — Check configuration

```cmd
kubectl describe svc <service-name>
```

Check:

```text
Type
Selector
Port
TargetPort
Endpoints
```

### Step 3 — Check matching Pods

```cmd
kubectl get pods --show-labels
```

### Step 4 — Check endpoints

```cmd
kubectl get endpoints <service-name>
```

### Step 5 — Test according to Service type

```text
ClusterIP
   → Test from inside cluster

NodePort
   → Test using Node IP + nodePort

LoadBalancer
   → Test using external address when provided
```

---

# 🔥 Final Memory

```text
SERVICE
   │
   ├── ClusterIP
   │      └── Internal communication
   │
   ├── NodePort
   │      └── External access through NodeIP:port
   │
   └── LoadBalancer
          └── External access through external LB
```

And always remember:

```text
Service
   ↓
selector
   ↓
matching Pod labels
   ↓
Endpoints
   ↓
Pods
```

**Most important rule:**

> A Service is a stable network endpoint; its `selector` determines which Pods receive the traffic.
