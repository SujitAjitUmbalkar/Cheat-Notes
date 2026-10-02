# Kubernetes LoadBalancer

```yaml
spec:
  type: LoadBalancer
```

A `LoadBalancer` Service exposes an application outside the Kubernetes cluster.

**How you get the LoadBalancer endpoint depends on the Kubernetes environment.**

---

# 1. Docker Desktop Kubernetes

Docker Desktop is a **local Kubernetes cluster**. It does not provide a cloud Load Balancer with a real public IP.

### Steps

Create/apply your Service:

```bash
kubectl apply -f service.yaml
```

Check:

```bash
kubectl get svc
```

You may see:

```text
NAME           TYPE           EXTERNAL-IP
api-gateway    LoadBalancer   <pending>
```

`<pending>` is expected because Docker Desktop does not have a cloud provider creating a public Load Balancer.

### How to access locally?

Use **NodePort** instead:

```yaml
spec:
  type: NodePort
```

Then:

```bash
kubectl get svc
```

Find the NodePort:

```text
80:30001/TCP
```

Access the application using the appropriate local node/host address and port.

For quick testing, you can also use:

```bash
kubectl port-forward svc/api-gateway 8080:80
```

Then:

```text
http://localhost:8080
```

### Remember

```text
Docker Desktop
     ↓
Local Kubernetes
     ↓
LoadBalancer Service
     ↓
❌ No real cloud LoadBalancer IP
```

---

# 2. Minikube

Minikube is also a **local Kubernetes cluster**, but it provides `minikube tunnel` for testing `LoadBalancer` Services.

### Step 1 — Create LoadBalancer Service

```yaml
spec:
  type: LoadBalancer
```

Apply:

```bash
kubectl apply -f service.yaml
```

### Step 2 — Start tunnel

Open a terminal and run:

```bash
minikube tunnel
```

Keep this terminal running.

### Step 3 — Check Service

In another terminal:

```bash
kubectl get svc
```

You can now get an external address for the LoadBalancer Service.

### Step 4 — Access it

Use the address shown by:

```bash
kubectl get svc
```

For example:

```text
NAME           TYPE           EXTERNAL-IP
api-gateway    LoadBalancer   10.x.x.x
```

Then access:

```text
http://10.x.x.x
```

### What does the tunnel do?

```text
Your PC
   ↓
minikube tunnel
   ↓
Minikube Kubernetes
   ↓
LoadBalancer Service
   ↓
Pods
```

⚠️ This is **local access**, not a real public cloud Load Balancer.

---

# 3. AWS Kubernetes (EKS)

AWS provides the cloud infrastructure required for a real external Load Balancer.

### Step 1 — Create/use an EKS cluster

Your Kubernetes cluster must be running on AWS, for example using **Amazon EKS**.

### Step 2 — Create LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
spec:
  type: LoadBalancer
  selector:
    app: api-gateway
  ports:
    - port: 80
      targetPort: 8080
```

### Step 3 — Apply

```bash
kubectl apply -f service.yaml
```

### Step 4 — Check Service

```bash
kubectl get svc
```

Initially:

```text
NAME           TYPE           EXTERNAL-IP
api-gateway    LoadBalancer   <pending>
```

Wait for AWS to provision the external Load Balancer.

Then:

```bash
kubectl get svc
```

You may get a hostname such as:

```text
NAME           TYPE           EXTERNAL-IP
api-gateway    LoadBalancer   xxx.elb.amazonaws.com
```

### Step 5 — Access it

Use the AWS Load Balancer endpoint:

```text
http://xxx.elb.amazonaws.com
```

The request flows:

```text
Internet
    ↓
AWS Load Balancer
    ↓
Kubernetes LoadBalancer Service
    ↓
API Gateway Pods
```

---

# Quick Comparison

| Environment        | `LoadBalancer` Service    | How to access                         |
| ------------------ | ------------------------- | ------------------------------------- |
| **Docker Desktop** | No cloud LB IP            | NodePort / port-forward               |
| **Minikube**       | Local LoadBalancer access | `minikube tunnel`                     |
| **AWS EKS**        | Real AWS Load Balancer    | `kubectl get svc` → external endpoint |

## Most Important

```text
Docker Desktop
→ Local testing
→ No real cloud LoadBalancer

Minikube
→ Local testing
→ minikube tunnel

AWS EKS
→ Cloud environment
→ AWS creates external LoadBalancer
→ kubectl get svc gives endpoint
```

**The Kubernetes YAML can be almost identical in all three cases. The difference is who provides the external Load Balancer.**
