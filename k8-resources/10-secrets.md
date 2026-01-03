# Kubernetes Secrets vs HashiCorp Vault

This guide explains **WHY Kubernetes Secrets exist**, **WHERE they fall short**,  
and **WHY tools like HashiCorp Vault are used in real production systems**.

Learning flow:
1. Scenario
2. Kubernetes Secrets (small example)
3. Limitations of Kubernetes Secrets
4. Why Vault exists
5. How Vault works with Kubernetes
6. When to use what

---

## 📌 Scenario (Real Problem)

Your application needs:
- Database password
- API token
- Cloud credentials

Example:
```text
DB_PASSWORD=admin123
```
Question:

>Where do we store this securely in Kubernetes?  
>Can we store this directly in Pod YAML or ConfigMap?

❌ No — this is unsafe.

## ❓ The Problem

If secrets are:

- Hardcoded in YAML
- Stored in ConfigMaps
- Committed to Git

Then:

- Anyone with repo access can see them
- Credentials can be leaked
- Security incidents happen

### ✅ The Solution: Kubernetes Secrets

A Secret is a Kubernetes object used to store:

- Sensitive data
- Passwords
- Tokens
- Keys

Secrets allow:

- Separation of code and credentials
- Safer handling than plain text config


### 📄 Create a Secret (Simple Example)
```
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: YWRtaW4xMjM=
```

⚠️ The value is base64 encoded, not encrypted.

### 🔍 What Is Base64?
```
admin123  →  YWRtaW4xMjM=
```

Anyone can decode it:
```
echo YWRtaW4xMjM= | base64 -d
```

---
## 📄 Using Secret as Environment Variable
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_PASSWORD
```

### 🔶 What happens at runtime (Important)

When Pod starts:

1️⃣ Kubernetes reads Pod YAML
2️⃣ Finds secretKeyRef
3️⃣ Fetches Secret db-secret
4️⃣ Decodes Base64 value
5️⃣ Injects it into container env. i.e DB_PASSWORD=admin123
6️⃣ App reads it as env var

🔥 Secret never appears in Pod YAML.

### 🔶 How to inject ALL keys (FYI)?

If you want ALL keys from a Secret as env vars:
```
envFrom:
- secretRef:
    name: db-secret
```

👉 This injects:
```
DB_PASSWORD=admin123
DB_USER=admin
```

But ⚠️ this is less controlled.

---

## ❓ Is This Used in Real-Time Projects?
✅ YES — but with conditions

Real projects use Kubernetes Secrets:

- For small / medium setups
- With RBAC restrictions
- With etcd encryption enabled

## The Hidden Problems with Kubernetes Secrets


### ❌ Problem 1: Not real encryption (by default)

- Stored in etcd
- Base64 encoding ≠ encryption
- Anyone with access can decode

### ❌ Problem 2: No automatic rotation

- Password expires → app breaks
- Manual update required
- Restart Pods manually

### ❌ Problem 3: Secrets live inside the cluster

- If cluster is compromised → secrets exposed
- No centralized secret control across systems

## ❌ Problem 4: No audit trail

You cannot easily answer:

- Who accessed which secret?
- When?
- From where?



---
## 🔐 What About HashiCorp Vault or Cloud Secrets?
✅ YES — very common in real projects

In production, teams often use:

- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault
- Google Secret Manager

## Why HashiCorp Vault Exists

HashiCorp Vault exists to solve what Kubernetes Secrets cannot.

Vault is:

- A dedicated secret management system
- Built for security, rotation, auditing
- Used across many platforms, not just Kubernetes
- Centralized control

```
Pod
 |
 | authenticate using ServiceAccount
 |
 v
Vault
 |
 | returns secret
 |
 v
Pod uses secret (in memory)
```

>Secrets are NOT stored in Kubernetes as plain data

They are:

- Fetched dynamically
- Often short-lived
- Rotated automatically

---
## Common Real-World Vault Patterns

###  High-Level Kubernetes + Vault Flow

1. Pod starts
2. Pod authenticates to Vault using ServiceAccount
3. Vault verifies Pod identity
4. Vault returns secret
5. Pod uses secret
### ✅ Pattern 1: Vault Agent Sidecar
```
Pod
├── App Container
└── Vault Agent (sidecar)
        |
        v
     Vault
```

- Sidecar fetches secrets
- Writes them to memory / file
- App reads them

### ✅ Pattern 2: CSI Driver (Very Common)
```
Vault
  |
  | mount secrets
  v
Pod filesystem
```
- Secrets mounted as files
- No secrets in etcd
- Very secure
