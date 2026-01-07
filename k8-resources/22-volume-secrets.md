# 🔐 Kubernetes Secrets (Volume)

## 📌 What is a Secret?

A **Secret** is a Kubernetes object used to store **sensitive data**, such as:
- Passwords
- API tokens
- Database credentials
- TLS certificates
- Private keys

> A **Secret** stores sensitive data as key–value pairs, encoded in Base64.

When used as a **volume**:
- Each key → becomes a **file**
- File content → decoded secret value
```
/Secret
 ├─ as env    → injects VALUES
 └─ as volume → generates FILES
```
---
## 🛠️ Step 1: Create a Secret

### Option 1️⃣ Using kubectl (Recommended)
```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=MyP@ssw0rd
```
### Option 2️⃣ Using YAML (Base64 required)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=        # admin
  password: TXlQQHNzdzByZA== # MyP@ssw0rd
```
📌 Values must be Base64 encoded

### 🛠️ Step 2: Mount Secret as a Volume
```yaml
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-demo
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret

  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "ls /secrets && cat /secrets/username; sleep 3600"]
    volumeMounts:
    - name: secret-volume
      mountPath: /secrets
      readOnly: true
```      
