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
# 3.1 ClusterIP

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

---

## Configuration

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

---

## Validate Service

### 1. Apply the Service

```bash
kubectl apply -f order-service.yaml
```

### 2. Check the Service

```bash
kubectl get svc
```

Check that:

* `TYPE` is `ClusterIP`
* Service has a `CLUSTER-IP`
* Port is correctly configured

### 3. Describe the Service

```bash
kubectl describe svc order-service
```

Check:

* `Selector`
* `Port`
* `TargetPort`
* `Endpoints`

### 4. Check Endpoints

```bash
kubectl get endpoints order-service
```

Example:

```text
NAME            ENDPOINTS
order-service   10.244.0.5:8080,10.244.0.6:8080
```

This confirms that the Service has found the Pods matching:

```yaml
selector:
  app: order
```

---

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

this is configuration , now

# Test Communication Between Pods Through ClusterIP

The communication pattern we want to test is:

```
Test Pod
    ↓
ClusterIP Service
    ↓
Order Pod

```

### Step 1: Check target Pods

```
kubectl get pods
```

Make sure the target Pods are `Running`.

---

### Step 2: Check the Service

```
kubectl get svc
```

Make sure `order-service` exists and is of type `ClusterIP`.

---

### Step 3: Check Service **Endpoints**

```
kubectl get endpoints order-service
```

You should see the IP addresses and ports of the target Pods.

Example:

```
order-service   10.244.0.5:8080,10.244.0.6:8080

```

If you see:

```
<none>

```

the Service is not finding the Pods. Check the Service selector and Pod labels.

---

### Step 4: Create a temporary Test Pod

```
kubectl run test-pod --image=curlimages/curl -it --rm -- sh
```

This creates a temporary Pod and immediately opens its shell.

You are now **inside the Pod**.

You will see something similar to:

```
/ $

```

> Because `-it --rm -- sh` is used, you do **not** need a separate `kubectl exec` command.

---

### Alternative: Enter an Existing Pod

If the Pod is already running:

```
kubectl get pods
```

Then:

```
kubectl exec -it test-pod -- sh
```

Now you are inside the Pod.

---

### Step 5: Test the ClusterIP Service

From inside the `test-pod`:

```
curl http://order-service
```

Or call a specific application endpoint:

```
curl http://order-service/orders
```

### Why don't we use `:80` in the URL?

Our Service uses:

```yaml
port: 80
```

Port `80` is the **default port for HTTP**, so:

```
curl http://order-service
```

automatically means:

```
curl http://order-service:80
```

Therefore, `:80` is optional.

### What if the Service uses another port, such as `81`?

If the Service configuration is:

```yaml
ports:
  - port: 81
    targetPort: 8080
```

then you **must specify the port** in the URL:

```
curl http://order-service:81
```

Because `81` is not the default HTTP port.

> **Remember:** The URL uses the **Service `port`**, not the `targetPort`.

If the application returns a response, communication is successful.

```
Test Pod
    │
    │ HTTP request
    ↓
order-service
   (ClusterIP)
    │
    ↓
Order Pod

```

---

## Test Service DNS

Kubernetes automatically provides DNS for Services.

From inside the test Pod:

```
curl http://order-service
```

Kubernetes resolves:

```
order-service
      ↓
ClusterIP
      ↓
Order Pods

```

You can also use the full DNS name:

```
order-service.<namespace>.svc.cluster.local

```

For example, if the Service is in the `prod` namespace:

```
curl http://order-service.prod.svc.cluster.local
```

---

## Exit the Test Pod

```
exit
```

If the Pod was created using:

```
kubectl run test-pod --image=curlimages/curl -it --rm -- sh
```

the `--rm` option automatically removes the temporary Pod after you exit.

---
# 3.2 NodePort

### What is it?

`NodePort` exposes a Service through a port on **each Kubernetes Node**.

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

---

## Configuration

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

### Traffic Flow

```text
Client
  ↓
NodeIP:30001
  ↓
Service:80
  ↓
Pod:8080
```

---

# Validate NodePort Service

### Step 1: Apply the Service

```bash
kubectl apply -f order-service.yaml
```

Expected:

```text
service/order-service created
```

---

### Step 2: Check the Service

```bash
kubectl get svc
```

Example:

```text
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)
order-service   NodePort   10.96.45.120   <none>        80:30001/TCP
```

Look at:

```text
PORT(S)

80:30001/TCP
```

This means:

```text
Service Port = 80
NodePort     = 30001
```

---

### Step 3: Describe the Service

```bash
kubectl describe svc order-service
```

Check:

* `Type`
* `Selector`
* `Port`
* `TargetPort`
* `NodePort`
* `Endpoints`

---

### Step 4: Check the Nodes

```bash
kubectl get nodes -o wide
```

Example:

```text
NAME                   STATUS   ROLES           INTERNAL-IP
desktop-control-plane  Ready    control-plane   192.168.x.x
```

You need the appropriate Node address for your Kubernetes environment.

---

### Step 5: Check the Target Pods

```bash
kubectl get pods -o wide
```

Make sure the `order` Pods are:

```text
STATUS = Running
```

Also verify that the Service has endpoints:

```bash
kubectl get endpoints order-service
```

Example:

```text
NAME            ENDPOINTS
order-service   10.244.0.5:8080,10.244.0.6:8080
```

This confirms that the Service has Pods to forward traffic to.

---

# Test NodePort Communication

The important difference from ClusterIP is:

```text
ClusterIP:
Pod inside cluster
      ↓
ClusterIP Service

NodePort:
External Client
      ↓
NodeIP:NodePort
      ↓
Service
      ↓
Pod
```

### Step 6: Access the Service

If your application is HTTP-based, such as a Spring Boot application, you can test the NodePort using a web browser such as **Chrome**.

Use:

```text
http://<NodeIP>:30001
```

For example:

```text
http://192.168.x.x:30001
```

If your application has an endpoint:

```text
http://192.168.x.x:30001/orders
```

Enter the URL directly into Chrome.

If the application returns a response, NodePort is working.

Traffic flow:

```text
Chrome
  ↓
NodeIP:30001
  ↓
NodePort Service
  ↓
Order Pod:8080
```

> **Docker Desktop note:** Do not blindly assume that the `INTERNAL-IP` shown by `kubectl get nodes -o wide` is the address that Chrome should use. Docker Desktop's networking can differ from Minikube or cloud Kubernetes. Use the appropriate Node address for your Docker Desktop environment.

---

## Docker Desktop Note

You are using **Docker Desktop Kubernetes**, so do **not** use:

```bash
minikube service order-service
```

That command is Minikube-specific.

Instead, use the Node address and NodePort appropriate to your Docker Desktop Kubernetes environment.

---

# Test From Inside the Cluster

You can also verify that the NodePort Service is reachable from another Pod.

### Step 1: Create a temporary Test Pod

```bash
kubectl run test-pod --image=curlimages/curl -it --rm -- sh
```

You are now inside the Pod.

---

### Step 2: Test the Service

```bash
curl http://order-service
```

This tests the Service through its normal **ClusterIP**.

---

### Step 3: Test the NodePort specifically

```bash
curl http://<NodeIP>:30001
```

However, for learning NodePort, the more important test is accessing:

```text
NodeIP:30001
```

from **outside the cluster**, such as Chrome.

---

### Step 4: Exit the Test Pod

```bash
exit
```

Because the Pod was created with `--rm`, Kubernetes automatically removes the temporary Pod after you exit.

---

## Complete Testing Flow

```text
1. Apply Service
       ↓
kubectl apply -f order-service.yaml

2. Check Service
       ↓
kubectl get svc

3. Check Service details
       ↓
kubectl describe svc order-service

4. Check Nodes
       ↓
kubectl get nodes -o wide

5. Check Pods
       ↓
kubectl get pods -o wide

6. Check Endpoints
       ↓
kubectl get endpoints order-service

7. Access NodePort
       ↓
Chrome → http://<NodeIP>:30001

8. Request reaches Service
       ↓
Order Pod
```

### Important

NodePort exposes the Service through a **port on the Kubernetes Node**.

```text
External Client
       ↓
NodeIP:30001
       ↓
NodePort Service
       ↓
Order Pod
```

Remember:

```text
ClusterIP
    ↓
Internal cluster access

NodePort
    ↓
External access through NodeIP:NodePort
```

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

Your API Gateway is an **application**.

---

### Use cases

Use it when:

* An application needs external/public access.
* You want an external load-balancing entry point.
* Your cloud/Kubernetes environment supports `LoadBalancer` Services.

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

---

## Configuration

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

### Traffic Flow

```text
Client
   ↓
External Load Balancer
   ↓
LoadBalancer Service :80
   ↓
API Gateway Pod :8080
   ↓
Order Service (ClusterIP)
   ↓
Order Pods
```

---

# Validate LoadBalancer Service

### Step 1: Apply the Service

```bash
kubectl apply -f api-gateway-service.yaml
```

Expected:

```text
service/api-gateway created
```

---

### Step 2: Check the Service

```bash
kubectl get svc
```

Example in a cloud environment:

```text
NAME          TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)
api-gateway   LoadBalancer   10.96.45.120   34.120.50.100     80:31234/TCP
```

Check:

* `TYPE` = `LoadBalancer`
* `EXTERNAL-IP` has an address if the infrastructure has provisioned one
* `PORT(S)` is correct

---

### Step 3: Describe the Service

```bash
kubectl describe svc api-gateway
```

Check:

* `Type`
* `Selector`
* `Port`
* `TargetPort`
* `Endpoints`
* Events related to LoadBalancer provisioning

---

### Step 4: Check the API Gateway Pods

```bash
kubectl get pods -o wide
```

Make sure the Gateway Pods are:

```text
STATUS = Running
```

Also check the Service endpoints:

```bash
kubectl get endpoints api-gateway
```

Example:

```text
NAME          ENDPOINTS
api-gateway   10.244.0.5:8080,10.244.0.6:8080
```

This confirms that the LoadBalancer Service has Gateway Pods to forward traffic to.

---

# Test LoadBalancer Communication

The important difference from NodePort is:

```text
NodePort:
External Client
      ↓
NodeIP:NodePort
      ↓
Service
      ↓
Pods

LoadBalancer:
External Client
      ↓
External Load Balancer
      ↓
Service
      ↓
Pods
```

### Step 5: Get the External Address

```bash
kubectl get svc api-gateway
```

Look at:

```text
EXTERNAL-IP
```

For example:

```text
34.120.50.100
```

---

### Step 6: Access the Service

If the API Gateway is an HTTP application, you can test it using **Chrome**.

Use:

```text
http://<EXTERNAL-IP>
```

For example:

```text
http://34.120.50.100
```

If your Gateway has an endpoint:

```text
http://34.120.50.100/orders
```

If the application returns a response, the LoadBalancer Service is working.

Traffic flow:

```text
Chrome
   ↓
External Load Balancer
   ↓
api-gateway Service
   ↓
API Gateway Pod
```

---

## Test Using curl

You can also test from a terminal:

```bash
curl http://<EXTERNAL-IP>
```

Or:

```bash
curl http://<EXTERNAL-IP>/orders
```

For learning, Chrome is enough when the application is HTTP-based.

---

# Docker Desktop Important Note

You are using **Docker Desktop Kubernetes**.

A `LoadBalancer` Service on a local Kubernetes environment does **not necessarily create a real public cloud Load Balancer**.

Therefore, after:

```bash
kubectl apply -f api-gateway-service.yaml
```

you may see:

```text
NAME          TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
api-gateway   LoadBalancer   10.96.45.120   <pending>      80:31234/TCP
```

`<pending>` means that your local environment has not provisioned an external LoadBalancer address.

Do **not** assume that Docker Desktop behaves like AWS, Azure, or GCP.

In a cloud Kubernetes environment:

```text
LoadBalancer Service
        ↓
Cloud Provider
        ↓
External Load Balancer
        ↓
Public IP / DNS
```

Locally:

```text
LoadBalancer Service
        ↓
Depends on local Kubernetes implementation
        ↓
May not receive a public EXTERNAL-IP
```

---

# What Happens in a Cloud Environment?

For example:

```text
Internet
    ↓
Cloud Load Balancer
    ↓
LoadBalancer Service
    ↓
API Gateway Pods
```

The cloud provider provisions the external load-balancing infrastructure when Kubernetes requests a `LoadBalancer` Service.

The exact implementation depends on the cloud provider and Kubernetes environment.

---

## Complete Testing Flow

```text
1. Apply Service
       ↓
kubectl apply -f api-gateway-service.yaml

2. Check Service
       ↓
kubectl get svc

3. Check Service details
       ↓
kubectl describe svc api-gateway

4. Check Gateway Pods
       ↓
kubectl get pods -o wide

5. Check Endpoints
       ↓
kubectl get endpoints api-gateway

6. Check EXTERNAL-IP
       ↓
kubectl get svc api-gateway

7. If an external address is provisioned
       ↓
Chrome → http://<EXTERNAL-IP>

8. Request reaches
       ↓
External Load Balancer
       ↓
api-gateway Service
       ↓
API Gateway Pod
```

### Important

`LoadBalancer` provides an **external entry point** through infrastructure that supports external load balancing.

```text
Internet
    ↓
External Load Balancer
    ↓
LoadBalancer Service
    ↓
API Gateway Pods
```

Remember:

```text
ClusterIP
    ↓
Internal cluster access

NodePort
    ↓
External access through NodeIP:NodePort

LoadBalancer
    ↓
External access through an infrastructure-provided
Load Balancer / external address
```

**Important architecture point:**

```text
Load Balancer ≠ API Gateway

Load Balancer
    → Infrastructure

API Gateway
    → Application
```

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
