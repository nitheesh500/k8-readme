# Kubernetes ConfigMap 

This guide explains **ConfigMap** in a simple, practical way.

We will follow this order:
1. Scenario (why ConfigMap is needed)
2. The problem
3. The solution (ConfigMap)
4. Simple examples
5. How ConfigMap is used (env & files)
6. Diagrams
7. Key rules

---

## 📌 Scenario (Why ConfigMap?)

Imagine this situation:

You have an application running in Kubernetes.
It needs configuration like:
- Environment name
- Log level
- App mode
- Feature flags

Example:
```
ENV=dev
LOG_LEVEL=info
```


These values:

- Change between environments (dev / test / prod)
- Are NOT secrets

### Question:

**Should we hardcode these values inside the Pod YAML or Dockerfile?**

❌ No.
---
## ❓ The Problem

If configuration is hardcoded:

- Every change needs a YAML update
- Same config is repeated in many Pods
- Configuration management becomes messy

Example (bad):
```
env:
- name: ENV
  value: dev
- name: LOG_LEVEL
  value: info
```

Now imagine 50 Pods with the same config ❌

### ✅ The Solution: ConfigMap

A ConfigMap is a Kubernetes object used to store:

- Non-sensitive configuration data
- Key–value pairs
- Config files

ConfigMap allows you to:

- Store config once
- Reuse it across many Pods
- Change config without touching Pod YAML


Pods read configuration, they don’t own it. 

### 📄 Create a ConfigMap (Key–Value)
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  ENV: dev
  LOG_LEVEL: info
```

This ConfigMap stores:
```
ENV=dev
LOG_LEVEL=info
```
### 🔗 Using ConfigMap in a Pod (Environment Variables)
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-pod
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
```

What happens:

- All keys from ConfigMap become env vars
- ENV and LOG_LEVEL are injected into container

🔍 Verify Environment Variables
```
kubectl exec -it configmap-env-pod -- env
```

You will see:
```
ENV=dev
LOG_LEVEL=info
```
### 📄 Using ConfigMap as Files (Very Common)

ConfigMaps can also be mounted as files.
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-file-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```
### 🖼️ Diagram: ConfigMap → Files
```
ConfigMap
---------
ENV=dev
LOG_LEVEL=info
     |
     |  mounted as files
     v
Container File System
---------------------
/config/ENV
/config/LOG_LEVEL
```

Each key becomes a file.

### ⚠️ Important Rules

- ConfigMap is for non-sensitive data
- Do NOT store passwords or tokens
- ConfigMap data is plain text
- ConfigMap does NOT restart Pods automatically
- Pods must be restarted to pick up changes (basic case)

| Feature              | Environment Variables | ConfigMap        |
|----------------------|-----------------------|------------------|
| Scope                | Per Pod               | Reusable         |
| Reuse                | ❌ No                 | ✅ Yes          |
| Central management   | ❌ No                 | ✅ Yes          |
| Sensitive data       | ❌ No                 | ❌ No           |


## 🎯 Interview One-Liner

**ConfigMaps provide centralized and reusable configuration, whereas environment variables are defined per Pod and are not reusable.**