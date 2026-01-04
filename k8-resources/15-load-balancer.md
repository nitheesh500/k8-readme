# 🌐 Kubernetes LoadBalancer Service"



# 🌐 Kubernetes LoadBalancer Service — Complete Guide


In Kubernetes, a **LoadBalancer Service** exposes an application **outside the cluster** using a
**cloud-provider managed load balancer** such as:

- AWS (ELB / ALB / NLB)
- Azure Load Balancer
- Google Cloud Load Balancer

It is the **production-grade and recommended way** to expose applications to the internet
in cloud-based Kubernetes clusters.

---

## ❓ Why Do We Need LoadBalancer?

Earlier service types have limitations:

- **ClusterIP** → Internal only
- **NodePort** → Opens static ports on every node (not ideal for production)

A LoadBalancer Service solves these problems by:
- Providing a **single public IP or DNS**
- Handling traffic distribution
- Integrating with cloud networking features

---

⚠️ Important:

- Kubernetes does NOT implement the load balancer
- AWS / Azure / GCP creates it
- Kubernetes just requests it

## Architecture
```
Internet / Browser
        |
        |  https://myapp.example.com
        v
Cloud Load Balancer (AWS ALB / NLB)
        |
        | forwards traffic
        v
NodePort (auto-created)
        |
        | kube-proxy
        v
ClusterIP Service
        |
        | Load balancing
        v
      Pod-1   Pod-2   Pod-3
```

🚨 LoadBalancer ALWAYS uses NodePort internally
🚨 NodePort is hidden from users

## Step-by-step: WHAT HAPPENS INTERNALLY

### Step 1️⃣ You create a Service
```
type: LoadBalancer
```
Kubernetes:

- Allocates ClusterIP
- Allocates NodePort
- Tells cloud provider:
   - 👉 “Create a Load Balancer”

### Step 2️⃣ Cloud provider creates LB

Example on AWS:

- ALB or NLB is created
- Public DNS assigned:
```
a1b2c3.elb.amazonaws.com
```
- Health checks configured

### Step 3️⃣ External traffic arrives
```
Browser → LoadBalancer DNS
```

Load balancer:

- Selects a healthy node
- Sends traffic to:
  ```
    NodeIP:NodePort
  ```
### Step 4️⃣ kube-proxy forwards traffic
```
NodePort → ClusterIP → Pod
```
✔️ Pod selected dynamically  
✔️ Failed Pods removed automatically

---
## 5️⃣ EXAMPLE
### 🔹 Application Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: nginx
        ports:
        - containerPort: 80
```
### 🔹 LoadBalancer Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-lb
spec:
  type: LoadBalancer
  selector:
    app: app
  ports:
  - port: 80
    targetPort: 80
```
## 6️⃣ What Kubernetes creates automatically
```
kubectl get svc app-lb
```

Output:
```
NAME     TYPE           CLUSTER-IP      EXTERNAL-IP
app-lb   LoadBalancer   10.96.21.45     a1b2.elb.amazonaws.com
```

✔️ ClusterIP
✔️ NodePort
✔️ External Load Balancer

7️⃣ Cloud-side view (AWS example)

Behind the scenes:

- ELB created
- Target group created
- Nodes registered as targets
- Health checks enabled

👉 If a node fails → removed automatically

## 8️⃣ Why LoadBalancer is Production Ready

| Feature          | NodePort | LoadBalancer |
|------------------|----------|--------------|
| External access  | ✅       | ✅           |
| Stable DNS       | ❌       | ✅           |
| SSL / TLS        | ❌       | ✅           |
| Health checks    | ❌       | ✅           |
| Auto scaling     | ❌       | ✅           |
| Cloud-native     | ❌       | ✅           |

LoadBalancer relies on cloud integration.


1️⃣ Who actually chooses the Load Balancer?

The decision is made by:
```
Cloud Controller Manager (CCM)
```
This component:

- Talks to AWS / Azure / GCP APIs
- Reads your Service YAML
- Reads annotations
- Creates the actual load balancer

📌 Kubernetes core has no logic for ALB vs NLB vs ELB.

---
## How do you CONTROL which LB is created?

✅ Using Annotations (THIS IS THE KEY)

Annotations are instructions to the cloud provider.

## 🔹 Force AWS NLB
```yaml
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

Result:

- AWS creates Network Load Balancer
- L4, high performance
- Static IPs

### 🔹 Internal Load Balancer (private)
```
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-internal: "true"
```

Result:

- No public IP
- Only VPC access

### 🔹 Control target type (instance vs IP)
```
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"
```

| Target Type | Meaning                |
|-------------|------------------------|
| `instance`  | Traffic → Node → Pod   |
| `ip`        | Traffic → Pod directly |

## Complete decision flow
```
Did you create a Service?
        |
        v
Is type = LoadBalancer?
        |
        v
Cloud Controller Manager
        |
        v
Are annotations present?
        |
   Yes / No
    |     |
    v     v
Specified LB   Default LB
```
## 🎤 Interview-ready explanation

- A LoadBalancer Service exposes Kubernetes applications externally by provisioning a cloud provider’s load balancer. Incoming traffic hits the external load balancer, which forwards requests to NodePorts on worker nodes. kube-proxy then routes the traffic to the service’s ClusterIP and load-balances it to healthy Pods. This provides a stable, secure, and scalable production-grade entry point.