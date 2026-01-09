# 🧱 StorageClass — Dynamic Storage Provisioning

## Scenario (Real Problem)

You are running Kubernetes in cloud (AWS / Azure / GCP).

You have:

- 50 microservices
- 20 databases
- 100 PVCs

Old (Static PV) way ❌

- Admin manually creates 100 PVs
- Pre-decide disk size
- Map each PVC manually

👉 Impossible to scale
👉 Slow
👉 Operational nightmare

### What a StorageClass is (Core Concept)

>A StorageClass defines how storage should be dynamically created.

Think of it as:

>🏭 A storage template / blueprint

It tells Kubernetes:

- What type of disk?
- Which cloud backend?
- Performance type?
- Reclaim policy?

###  Key idea (VERY IMPORTANT — remember this)

```PVC``` + ```StorageClass``` = PV auto-created

You do NOT create PV manually anymore.

##  High-level architecture
```
Pod
 ↓
PVC (requests storage)
 ↓
StorageClass (rules)
 ↓
Provisioner (CSI driver)
 ↓
Actual Disk (EBS / Disk / PD)
 ↓
Auto-created PV
```
---
### 1️⃣ First: What is CSI (quick reminder)

CSI (Container Storage Interface) is the standard way Kubernetes talks to storage systems.

Without CSI:

- Kubernetes cannot create disks
- Cannot attach/detach volumes
- Cannot provision storage dynamically

👉 CSI = storage plugin

### 2️⃣ Do we NEED to install CSI? (CLEAR ANSWER)
✅ Short answer:  
 
Sometimes YES, sometimes NO — depends on how your cluster was created.

Let’s break it down.

### 3️⃣ Case 1: Managed Kubernetes (EKS / AKS / GKE)
Do you need to install CSI manually?

➡️ NO (usually)

Because:

>Cloud provider pre-installs or auto-manages CSI drivers

Examples: CSI Availability by Platform
| Platform | CSI Status                          |
|----------|-------------------------------------|
| EKS      | EBS CSI often installed as add-on   |
| AKS      | Disk CSI auto-installed             |
| GKE      | PD CSI auto-installed               |


📌 You may only need to enable or update, not install from scratch.

### 4️⃣ Case 2: Self-managed / On-prem / kubeadm
Do you need to install CSI?

>➡️ YES

Because:

>Kubernetes ships WITHOUT storage drivers

You must install:

- NFS CSI
- Ceph CSI
- EBS CSI (if AWS but self-managed)

👉 No CSI = no dynamic volumes

---
## Example (AWS-style)
### Simple StorageClass 
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-sc
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

📌 Provisioner = who creates storage  
📌 Parameters = disk type  
📌 ReclaimPolicy = what happens after delete  

### PVC using StorageClass 
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  storageClassName: standard-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

What happens automatically:  
1️⃣ Disk is created in cloud  
2️⃣ PV is created    
3️⃣ PVC binds to PV 
4️⃣ Pod can mount storage  

🚨 No PV YAML written by you

### Pod using the PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo hello > /data/file.txt; sleep 3600"]
    volumeMounts:
    - name: data
      mountPath: /data

  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: app-pvc
```

📌 Pod still talks only to PVC  
📌 Storage details are hidden

---
### Dynamic provisioning lifecycle (CRITICAL FLOW)
```
PVC created
   ↓
StorageClass selected
   ↓
CSI provisioner creates disk
   ↓
PV auto-created
   ↓
PVC bound
   ↓
Pod mounts PVC
```

### volumeBindingMode (INTERVIEW FAVORITE)
Mode	Meaning
Immediate	Disk created immediately
WaitForFirstConsumer	Disk created after Pod scheduling

📌 Best practice: WaitForFirstConsumer
(prevents AZ mismatch in cloud)

### Default StorageClass (IMPORTANT)
```
kubectl get storageclass
```

You’ll see:
```
standard (default)
```

If PVC does NOT specify storageClassName:  
👉 Default StorageClass is used automatically
👉 99% production clusters use StorageClass

## Interview-ready explanation

- A StorageClass defines how persistent storage is dynamically provisioned in Kubernetes. When a PVC references a StorageClass, Kubernetes automatically creates a matching PersistentVolume using the specified provisioner and parameters. This eliminates manual PV management and is the standard approach for persistent storage in cloud environments.

---
| Volume Type        | Persists Pod Deletion | Persists Container Restart | Shared Between Containers | Node Dependent | Used in Production | Typical Use Case           |
|--------------------|----------------------|----------------------------|---------------------------|---------------|--------------------|----------------------------|
| emptyDir           | ❌ No                | ✅ Yes                     | ✅ Yes                    | ❌ No         | ⚠️ Limited         | Cache, temp files          |
| hostPath           | ✅ Yes               | ✅ Yes                     | ✅ Yes                    | ✅ Yes        | ❌ (apps)          | Node logs, agents          |
| ConfigMap          | ❌ No                | ✅ Yes                     | ✅ Yes                    | ❌ No         | ✅ Yes             | App config files           |
| Secret             | ❌ No                | ✅ Yes                     | ✅ Yes                    | ❌ No         | ✅ Yes             | Passwords, certs           |
| PVC (PV-backed)    | ✅ Yes               | ✅ Yes                     | ⚠️ Depends                | ❌ No         | ✅ Yes             | Databases, uploads         |
| NFS                | ✅ Yes               | ✅ Yes                     | ✅ Yes                    | ❌ No         | ✅ Yes             | Shared storage             |
| CSI (EBS / Disk)   | ✅ Yes               | ✅ Yes                     | ⚠️ Depends                | ❌ No         | ✅ Yes             | Cloud block storage        |
