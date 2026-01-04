# Cluster IP


**Purpose**: Exposes the service within the cluster.  
**Access**: Internal (cluster-internal IP only).  
**Use Case**: For internal communication between pods (e.g., microservices communicating with each other).


## Architecture
```
Client Pod
   |
   |  (DNS name)
   v
my-service.default.svc.cluster.local
   |
   |  (ClusterIP: 10.96.120.50)
   v
kube-proxy (iptables / ipvs)
   |
   |  (Load balancing)
   v
Pod-1     Pod-2     Pod-3
```
🚨 ClusterIP is NOT a Pod IP
🚨 It is a VIRTUAL IP

### Example
```
Frontend Pod
   |
   v
backend-service (ClusterIP)
   |
   v
Backend Pods
   |
   v
mysql-service (ClusterIP)
   |
   v
MySQL Pod
```

## 🪜 Step-by-step

### 🔹 STEP 1: Pod sends request
```
curl http://backend-service
```

Inside Pod A:
```
Destination = backend-service
```

### 🔹 STEP 2: DNS resolution

Kubernetes DNS converts name → IP
```
backend-service  →  10.96.120.50
```

Now Pod A sends a packet to:
```
DEST IP = 10.96.120.50
```

🚨 Still NOT a real machine

### 🔹 STEP 3: Packet reaches the node kernel

Packet enters Linux networking stack on the node where Pod A runs.

Kernel sees:
```
DEST IP = 10.96.120.50
```
### 🔹 STEP 4: kube-proxy MAGIC happens 🧙

kube-proxy has already programmed iptables rules like this:
```
IF destination IP == 10.96.120.50
THEN redirect to one of these Pod IPs:
- 10.244.1.5:8080
- 10.244.2.7:8080
```

📌 This happens inside the kernel
📌 No user-space app involved

### 🔹 STEP 5: Packet is rewritten (important)

The kernel changes the destination:
```
FROM: 10.96.120.50:80
TO:   10.244.1.5:8080
```

This is called DNAT (Destination NAT).

### 🔹 STEP 6: Packet reaches the Pod

Now the packet goes to:
```
Pod IP:Port
```
The container app is listening → request is served 🎉

## EXAMPLE

### 1️⃣ Backend Deployment (Pods)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: nginx
        ports:
        - containerPort: 80
```
📌 This creates 3 backend Pods
📌 All Pods have label: app=backend

### 2️⃣ Backend Service (ClusterIP)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP        # DEFAULT
  selector:
    app: backend
  ports:
  - port: 80             # Service port
    targetPort: 80       # Pod port
```
What Kubernetes creates here:

- ✅ Virtual IP (example): 10.96.120.50
- ✅ DNS name: backend-service
- ✅ Load-balancing rules

### 3️⃣ Frontend Pod (Client)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: frontend
    image: curlimages/curl
    command: ["sleep", "3600"]
```

### 4️⃣ How traffic actually flows

From Frontend Pod:
```
kubectl exec -it frontend -- curl http://backend-service
```
What happens internally:
```
frontend pod
   |
   | DNS lookup
   v
backend-service → 10.96.120.50 (ClusterIP)
   |
   | kube-proxy (iptables / ipvs)
   v
backend-pod-1 OR backend-pod-2 OR backend-pod-3
```
---


### 🔍 Comparison: ClusterIP vs NodePort

| Feature             | ClusterIP | NodePort |
|---------------------|-----------|----------|
| Internal access     | ✅        | ✅       |
| External access     | ❌        | ✅       |
| Opens node port     | ❌        | ✅       |
| Load balances Pods  | ✅        | ✅       |

### 🎤 Interview-Ready Explanation

- NodePort exposes a Kubernetes service externally by opening a fixed port on every worker node. Traffic sent to <NodeIP>:NodePort is intercepted by kube-proxy, forwarded to the service’s ClusterIP, and then load-balanced to healthy Pods. NodePort is mainly used in environments without cloud load balancers or for development and debugging.