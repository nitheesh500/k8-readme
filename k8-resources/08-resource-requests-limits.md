# Kubernetes Resource Requests & Limits

This guide explains **Resource Requests and Limits** in a simple, practical way.

We will follow this order:
1. Scenario (why resource limits are needed)
2. The problem
3. The solution (requests & limits)
4. Simple Pod example
5. What happens at runtime
6. Diagrams
7. Key rules

---

## 📌 Scenario (Why Resource Limits?)

Imagine this situation:

You have multiple Pods running on the **same Kubernetes node**.
One Pod suddenly:
- Uses too much CPU
- Consumes all memory

Result:
- Other Pods become slow
- Some Pods crash
- Node becomes unstable

Question:
> How do we prevent one Pod from affecting others?

---

## ❓ The Problem (Without Limits)

Without resource limits:
- Any Pod can use unlimited CPU
- Any Pod can consume all memory
- One bad Pod can crash the entire node

Example problem:
```
Pod A uses all memory
Pod B crashes
```
❌ No control

### ✅ The Solution: Requests and Limits

Kubernetes provides resource requests and limits to control:

- CPU usage
- Memory usage

They help Kubernetes:

- Decide where to place Pods
- Protect nodes from overload

🧠 Simple Mental Model
```
Node Resources
--------------
CPU: 4 cores
Memory: 8Gi
```
Pods ask for resources  
Pods are restricted by limits

---
### 🔹 What is a Resource Request?

A request is:

>The minimum amount of resource a Pod needs

Kubernetes uses requests to:

- Schedule Pods onto nodes

Think of it as:

>“I need at least this much to run”

---
### 🔹 What is a Resource Limit?

A limit is:

>The maximum amount of resource a Pod can use

Kubernetes uses limits to:

- Stop Pods from overusing resources

Think of it as:

> “I must not use more than this”


### 📄 Simple Pod Example (CPU & Memory)
```
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "256Mi"
```
### 🔍 What This Means (Simple)

- cpu: 100m → needs at least 0.1 CPU
- memory: 128Mi → needs at least 128 MB
- cpu limit: 500m → can use max 0.5 CPU
- memory limit: 256Mi → can use max 256 MB

---
# ⚠️ What Happens at Runtime?

## What Happens When CPU Limit Is Exceeded?

- Kubernetes does NOT kill the Pod
- Kubernetes does NOT restart the container
- Kubernetes slows down the container

This slowdown is called CPU throttling.

**CPU Throttling = Kubernetes limits how fast the container can run.**

The app keeps running, but:

- Requests take longer
- Processing becomes slower
- Performance drops

## What Happens When Memory Limit Is Exceeded?

- **❌ The container is KILLED immediately.**

- Status becomes OOMKilled
- Pod may restart automatically (if allowed)


⚠️ Important Beginner Rules

- ✅ Always set requests
- ✅Always set limits
- ✅CPU can be throttled
- ❌Memory cannot be throttled
- ❌ Setting memory limit too low
- ❌ Not testing memory usage
