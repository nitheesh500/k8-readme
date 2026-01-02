# Kubernetes Namespace

This guide explains **Namespaces** in the simplest possible way.

We will follow this order:
1. Scenario (why namespaces exist)
2. The problem
3. The solution (namespace)
4. Simple examples
5. How it helps in real projects

---

## 📌 Scenario (Why do we need Namespaces?)

Imagine this situation:

You have **one Kubernetes cluster**, and:
- Developers are deploying apps
- QA team is testing apps
- DevOps team is managing system components

All of them use the **same cluster**.

Question:
> How do we avoid confusion, name conflicts, and accidents?

---

## ❓ The Problem (Without Namespace)

Without namespaces:
- Everyone creates resources in the **same place**
- Two teams might create a Pod with the same name
- Deleting resources becomes risky
- Hard to separate environments (dev, test, prod)

Example problem:
```
Team A creates: nginx-pod
Team B creates: nginx-pod
```
❌ Name conflict

✅ The Solution: **Namespace**

A Namespace is a logical boundary inside a Kubernetes cluster.

Think of it as:
- A folder inside a filesystem
- A room inside a building

Each namespace:
- Is isolated
- Can have resources with the same names
- Helps organize workloads

🧠 Simple Mental Model
```
Kubernetes Cluster
├── default
│   └── app pods
├── dev
│   └── dev pods
├── test
│   └── test pods
└── prod
    └── prod pods
```

Same resource names are allowed in different namespaces.

## 📄 What is a Namespace?

A Namespace:
- Is NOT a node
- Is NOT a cluster
- Is NOT a network boundary  

It is only:
> A logical grouping of Kubernetes resources

### 🔹 Default Namespaces (Important)

Kubernetes creates some namespaces automatically:

| Namespace       | Purpose                                         |
|-----------------|-------------------------------------------------|
| default         | Where resources go if no namespace is specified |
| kube-system     | Kubernetes internal components                  |
| kube-public     | Publicly readable resources                     |
| kube-node-lease | Node heartbeat and health information           |

### 📄 Create a Namespace (Simple YAML)
```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Apply it:
```
kubectl apply -f namespace.yaml
```
### 📄 Create a Pod Inside a Namespace
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx
```

This Pod will be created inside the dev namespace, not default.

### 🔍 How to See Namespaces
```
kubectl get namespaces
```
List Pods in a namespace:
```
kubectl get pods -n dev
```