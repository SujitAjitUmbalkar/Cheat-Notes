
# Kubernetes Service DNS Resolution

## 1. What is Service DNS?

Kubernetes provides **DNS-based service discovery**.

Instead of calling a Service using its changing ClusterIP:

```text
http://10.96.120.15
```

another application inside the cluster can use the Service name:

```text
http://order-service
```

Kubernetes DNS is provided by **CoreDNS**.

### Why DNS is useful

Pods are temporary and their IPs can change.

```text
Order Pod
   ↓
order-service
   ↓
Inventory Pods
```

The Order application doesn't need to know:

* Inventory Pod IP
* Which Inventory Pod is currently running
* Which Pod gets recreated

It simply calls:

```text
http://inventory-service
```

Kubernetes DNS resolves that Service name to the Service's network endpoint.

---

# 2. Service DNS Name

For a Service:

```text
inventory-service
```

inside namespace:

```text
prod
```

the full DNS name is:

```text
inventory-service.prod.svc.cluster.local
```

### DNS structure

```text
<service-name>.<namespace>.svc.cluster.local
```

Example:

```text
inventory-service.prod.svc.cluster.local
       │                 │
       │                 └── namespace
       └── Service name
```

Kubernetes also allows the shorter name when the caller is in the same namespace:

```text
http://inventory-service
```

---

# 3. Complete Request Flow

Suppose:

```text
Order Service
     ↓
inventory-service
     ↓
Inventory Pods
```

The Order application makes:

```http
GET http://inventory-service/products/10
```

The flow is approximately:

```text
Order Pod
   ↓
DNS lookup: inventory-service
   ↓
CoreDNS
   ↓
inventory-service Service
   ↓
Matching Inventory Pods
```

The application doesn't need to know the Pod IP.

---

# 4. DNS Names in Different Namespaces

Suppose:

```text
Order Service → namespace: prod
Inventory Service → namespace: prod
```

Order can simply use:

```text
http://inventory-service
```

If Inventory is in another namespace, use the namespace:

```text
http://inventory-service.inventory.svc.cluster.local
```

General form:

```text
http://<service-name>.<namespace>.svc.cluster.local
```

---

# 5. Practical Testing

We'll create a small Nginx Deployment and Service.

## Step 1 — Create Deployment

```cmd
kubectl create deployment nginx --image=nginx --replicas=2
```

Check:

```cmd
kubectl get pods
```

---

## Step 2 — Create ClusterIP Service

```cmd
kubectl expose deployment nginx --name=nginx-service --port=80 --target-port=80 --type=ClusterIP
```

Check:

```cmd
kubectl get svc
```

You should see something similar to:

```text
NAME            TYPE        CLUSTER-IP      PORT(S)
nginx-service   ClusterIP   10.x.x.x        80/TCP
```

---

# 6. Check Service Details

```cmd
kubectl describe svc nginx-service
```

Important things to check:

```text
Type:       ClusterIP
IP:         10.x.x.x
Port:       80
TargetPort: 80
Selector:   app=nginx
Endpoints:  ...
```

The Service should have endpoints because our Nginx Pods match:

```text
app=nginx
```

Check directly:

```cmd
kubectl get endpoints nginx-service
```

---

# 7. Test DNS Resolution

Your Windows CMD/browser is **outside the Kubernetes cluster**.

Therefore, don't test:

```text
nslookup nginx-service
```

from your normal Windows machine.

Instead, run a temporary Pod **inside Kubernetes**.

```cmd
kubectl run dns-test --rm -it --image=busybox:1.36 -- nslookup nginx-service
```

You should get a result containing the Service's ClusterIP.

Conceptually:

```text
nginx-service
      ↓
CoreDNS
      ↓
ClusterIP
```

This proves that Kubernetes DNS can resolve the Service name.

---

# 8. Test Full DNS Name

Test:

```cmd
kubectl run dns-test --rm -it --image=busybox:1.36 -- nslookup nginx-service.prod.svc.cluster.local
```

If your current namespace is `prod`, this should resolve to the same Service.

Full name:

```text
nginx-service.prod.svc.cluster.local
```

---

# 9. Test the Actual API/HTTP URL

DNS resolution alone isn't enough.

We also want to verify:

```text
DNS → Service → Pod → Application
```

Run a temporary Pod with `wget`:

```cmd
kubectl run http-test --rm -it --image=busybox:1.36 -- wget -qO- http://nginx-service
```

Nginx should return its HTML response.

This proves:

```text
http://nginx-service
       ↓
     CoreDNS
       ↓
nginx-service
       ↓
    Nginx Pods
       ↓
    HTTP response
```

---

# 10. Test Using the Full API URL

You can also call:

```cmd
kubectl run http-test --rm -it --image=busybox:1.36 -- wget -qO- http://nginx-service.prod.svc.cluster.local
```

Both should reach the same Service:

```text
http://nginx-service
```

and:

```text
http://nginx-service.prod.svc.cluster.local
```

---

# 11. Real Microservices Example

Suppose you have:

```text
order-service
inventory-service
payment-service
```

all inside namespace:

```text
prod
```

Your Order application can call Inventory using:

```http
GET http://inventory-service/products/10
```

And Payment:

```http
POST http://payment-service/payments
```

No Pod IPs are hardcoded.

### Example Spring Boot configuration

Instead of:

```properties
inventory.url=http://10.96.120.20:8080
```

use the Service DNS name:

```properties
inventory.url=http://inventory-service:8080
```

Then:

```text
Order Pod
   ↓
http://inventory-service:8080
   ↓
Kubernetes DNS
   ↓
Inventory Service
   ↓
Inventory Pods
```

---

# 12. Very Important: Browser vs Kubernetes DNS

This is a common confusion.

### ❌ Browser outside Kubernetes

You cannot normally type:

```text
http://inventory-service
```

into your browser and expect it to work.

Why?

Because:

```text
inventory-service
```

is Kubernetes' internal DNS name.

### ✅ Application inside Kubernetes

An application Pod can use:

```text
http://inventory-service
```

because it uses Kubernetes DNS.

### External user

The normal flow is:

```text
Browser
   ↓
External API Gateway
   ↓
Order Service
   ↓
Inventory Service
```

The browser talks to the **externally exposed entry point**.

The microservices talk to each other using **ClusterIP + DNS**.

---

# 13. Useful Commands

| Command                                                                             | Purpose                    |
| ----------------------------------------------------------------------------------- | -------------------------- |
| `kubectl get svc`                                                                   | List Services              |
| `kubectl describe svc <name>`                                                       | Service details            |
| `kubectl get endpoints <name>`                                                      | Check backend endpoints    |
| `kubectl get endpointslices`                                                        | Check EndpointSlices       |
| `kubectl run dns-test --rm -it --image=busybox:1.36 -- nslookup <service>`          | Test DNS                   |
| `kubectl run http-test --rm -it --image=busybox:1.36 -- wget -qO- http://<service>` | Test HTTP through Service  |
| `kubectl get pods --show-labels`                                                    | Check Pod labels           |
| `kubectl get svc -A`                                                                | Services across namespaces |

---

# 14. Debugging Service DNS

If:

```text
http://inventory-service
```

doesn't work, check in this order:

### ① Does the Service exist?

```cmd
kubectl get svc inventory-service
```

### ② Does the Service have endpoints?

```cmd
kubectl get endpoints inventory-service
```

If empty:

```cmd
kubectl get pods --show-labels
```

Check whether the Service selector matches the Pod labels.

### ③ Does DNS resolve?

```cmd
kubectl run dns-test --rm -it --image=busybox:1.36 -- nslookup inventory-service
```

### ④ Does HTTP work?

```cmd
kubectl run http-test --rm -it --image=busybox:1.36 -- wget -qO- http://inventory-service
```

---

# 15. The Complete Mental Model

```text
                 KUBERNETES CLUSTER

Order Pod
   │
   │  GET http://inventory-service/products/10
   ↓
Kubernetes DNS / CoreDNS
   │
   │ resolves name
   ↓
inventory-service
   │
   │ selects Pods using labels
   ↓
┌──────────────┬──────────────┐
↓              ↓              ↓
Inventory Pod  Inventory Pod  Inventory Pod
```

### Remember

```text
Service name
     ↓
DNS resolution
     ↓
Service
     ↓
Selector
     ↓
Matching Pods
     ↓
Application response
```

**Core rule:**

> Inside Kubernetes, use the **Service DNS name**, not the Pod IP, to communicate with another application.

The Kubernetes Service DNS format is:

```text
<service-name>.<namespace>.svc.cluster.local
```

and for the same namespace, usually:

```text
<service-name>
```

Kubernetes uses CoreDNS for Service DNS discovery. 

**Best practical test sequence to memorize:**

```cmd
kubectl get svc
kubectl get endpoints <service-name>
kubectl run dns-test --rm -it --image=busybox:1.36 -- nslookup <service-name>
kubectl run http-test --rm -it --image=busybox:1.36 -- wget -qO- http://<service-name>
```

That takes you from **“Does the Service exist?” → “Does it have Pods?” → “Does DNS work?” → “Does the actual API request work?”**
