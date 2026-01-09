1️⃣ Scenario (Real Problem)

You manage a Kubernetes cluster with many nodes.

You want to run:

- Log collection agent (Fluentd / Filebeat)
- Monitoring agent (Node Exporter)
- Security agent
- Network plugin (CNI)
- Storage plugin (CSI)

The requirement:

>Each node must have EXACTLY one copy of the Pod

Questions:

- How do we ensure a Pod runs on every node?
- How do we auto-handle new nodes?
- How do we avoid manually creating Pods?

👉 This is where DaemonSet is required.

---
### 2️⃣ Why Pods or Deployments are NOT enough
❌ Using Pods

- You must manually create Pods
- New node added → no Pod runs there
- No self-healing

❌ Using Deployment

- Deployment manages number of replicas
- It does NOT care where Pods run
- You might get:
  - 3 Pods on 1 node
  - 0 Pods on another node

🚨 Deployment = count-based
🚨 DaemonSet = node-based

### 3️⃣ What a DaemonSet is (Core Concept)

>A DaemonSet ensures that one (and only one) Pod runs on each selected node in the cluster.

Think of it as:

🧠 “If a node exists, run this Pod on it.”

### 4️⃣ How DaemonSet works internally
```
DaemonSet
   ↓
Scheduler logic (node-aware)
   ↓
One Pod per Node
```

Behavior:

- New node joins → Pod auto-created ✅
- Node removed → Pod auto-removed ✅
- Node reboot → Pod restarted ✅

---
## 5️⃣ Simple DaemonSet Example
### 🔹 DaemonSet
```YAML
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-logger
spec:
  selector:
    matchLabels:
      app: logger
  template:
    metadata:
      labels:
        app: logger
    spec:
      containers:
      - name: logger
        image: busybox
        command: ["sh", "-c", "echo logging on $(hostname); sleep 3600"]
```

📌 No replicas field
📌 Kubernetes decides how many Pods

6️⃣ How Pods are placed (VERY IMPORTANT)
```
Node-1 → logger Pod
Node-2 → logger Pod
Node-3 → logger Pod
```

- Exactly one Pod per node
- Even if node has no workloads

### 7️⃣ What happens when nodes change?
➕ New node added
```
Node-4 joins cluster
→ DaemonSet creates Pod automatically
```
➖ Node removed
```
Node deleted
→ DaemonSet Pod removed
``` 

✔️ Fully automatic
✔️ Zero manual effort