# Kubernetes Environment Variables

This guide explains **Environment Variables** in Kubernetes in a simple way.

We will follow this order:
1. Scenario (why env vars are needed)
2. The problem
3. The solution (environment variables)
4. Simple Pod examples
5. Common beginner rules

---

## 📌 Scenario (Why Environment Variables?)

Imagine this situation:

You have the **same application** running in:
- dev
- test
- prod

Only a few things change:
- Environment name
- Database URL
- Feature flags

Question:
> Do we rebuild the container image for every environment?

❌ No. That is a bad practice.

---

## ❓ The Problem

If values are hardcoded inside the application:
- You must rebuild the image for every change
- Same image cannot be reused
- Environment management becomes messy

Example (bad idea):
```
ENV=prod (hardcoded inside code)
```

## 📍 Scope of Environment Variables (IMPORTANT)

This is the most important beginner concept.

> 🔹 **Scope = Container level**

Environment variables are:

- ✅ Available inside one container
- ❌ NOT shared automatically with other containers
- ❌ NOT available outside the Pod

### 🔹 Scope Diagram (Beginner View)
```
Pod
├── Container A
│   └── ENV=dev      ✅ available
│
├── Container B
│   └── ENV=dev      ❌ NOT available
```

📌 Even though both containers are in the same Pod,  
environment variables are **per container**.

### 📄 Simple Pod Example with Environment Variable
```
apiVersion: v1
kind: Pod
metadata:
  name: env-demo-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: ENV
      value: dev
```

What this means:

- ENV=dev is set
- Only inside the app container
- Nowhere else

### 🔍 How to Verify Environment Variables

Exec into the container:
```
kubectl exec -it env-demo-pod -- env
```

Expected output:
```
ENV=dev
```
### 📄 Multiple Environment Variables
```
env:
- name: ENV
  value: dev
- name: LOG_LEVEL
  value: info
```

All of them follow the same scope rules.
---
### ⚠️ Important Beginner Rules

- Environment variables are container-scoped
- They are not shared across containers
- They are not global to the Pod
- Avoid hardcoding secrets

❌ Bad:
```
DB_PASSWORD=admin123
```

✅ Good:
```
ENV=prod
LOG_LEVEL=info
```
Secrets will be handled using Secrets later.




# Dockerfile ENV vs Kubernetes Environment Variables

### **what happens when an environment variable is defined**
- inside a **Dockerfile**
- and again inside a **Kubernetes Pod**

We will follow this order:
1. Scenario
2. The confusion
3. The rule (very important)
4. Examples
5. Diagram
6. Best practices

---

## 📌 Scenario

You build a container image using a Dockerfile.

Inside the Dockerfile, you already defined:
```
ENV ENV=prod
````

Later, while deploying the same image in Kubernetes, you define:
```
env:
- name: ENV
  value: dev
```

### Question:

#### **Which value will the container finally use?**

### ❓ The Confusion

Questions?:

- Will Dockerfile value be used?
- Will Kubernetes value be used?
- Will both exist?
- Will this cause an error?

### ✅ The Rule (VERY IMPORTANT)

**Environment variables defined in Kubernetes OVERRIDE environment variables defined in the Dockerfile.**

### 📦 Case 1: ENV only in Dockerfile
```
FROM nginx
ENV ENV=prod
```
```
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
  - name: app
    image: my-nginx
```
Result inside container
```
ENV=prod
```

✔ Dockerfile value is used
✔ Kubernetes did not override anything

### 📦 Case 2: ENV in Dockerfile AND Kubernetes
```
FROM nginx
ENV ENV=prod
```
```
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
  - name: app
    image: my-nginx
    env:
    - name: ENV
      value: dev
```
Result inside container
```
ENV=dev
```

❗ Dockerfile said prod  
❗ Kubernetes changed it to dev

**👉 Kubernetes overrides Dockerfile**

## 📦 Case 3: Partial Override (Merge Behavior)
```
ENV APP_NAME=myapp
ENV ENV=prod
```
```
env:
- name: ENV
  value: test
```
Result inside container
```
APP_NAME=myapp   (from Dockerfile)
ENV=test         (from Kubernetes)
```

✔ Kubernetes overrides only what it defines  
✔ Other Dockerfile env variables remain

### 🖼️ Diagram: How Override Works 
```
Dockerfile (Image Build)
------------------------
ENV ENV=prod
ENV APP_NAME=myapp
        |
        |  (image created)
        v
Kubernetes Pod (Runtime)
------------------------
ENV ENV=dev
        |
        v
Final Container Environment
---------------------------
ENV=dev
APP_NAME=myapp
```