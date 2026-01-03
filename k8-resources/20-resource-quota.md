# Kubernetes ResourceQuota

This guide explains **ResourceQuota** and **why it comes AFTER resource requests & limits**.

We will follow this order:
1. Scenario (real problem)
2. What requests & limits solved
3. What problem still remains
4. ResourceQuota solution
5. Simple examples
6. Diagrams
7. Key rules

---

## 📌 Scenario (Why ResourceQuota?)

You already learned **resource requests & limits**.

Now imagine this situation:

You have **one Kubernetes cluster** shared by:
- Team A (dev)
- Team B (qa)
- Team C (prod)

Each team deploys Pods **with proper requests & limits**.

Still, problems happen.

Question:
> If every Pod has limits, why do we still need ResourceQuota?

---

## 🔁 Recap: What Requests & Limits Solve

**Requests & limits work at Pod / container level.**

They ensure:
- A single Pod does not overuse CPU or memory
- Node stays stable
- One Pod does not kill others

Example:
```
Pod A limit: 500m CPU
Pod B limit: 500m CPU
```

✔ Pod-level protection
✔ Node-level safety

### ❗ The Remaining Problem (Important)

Even with limits, a team can still:

- Create too many Pods
- Consume all cluster resources
- Block other teams from deploying



🧠 Problem 
```
Cluster Resources
-----------------
CPU: 8 cores
Memory: 16Gi

Team A:
- 100 Pods × 100m CPU = 10 CPU ❌
- Each Pod uses small resources

Team B:
- Cannot schedule Pods
```
No Pod is violating limits, but the **cluster is still exhausted**.  
> Requests & limits cannot stop this.

## ✅ The Solution: ResourceQuota

A ResourceQuota:

- Is applied at namespace level
- Sets hard limits for a namespace
- Controls total usage, not individual Pods
- It enforces hard limits

Think of it as:

> A budget for a namespace

🧠 Simple Mental Model
```
Cluster
└── Namespace (dev)
    ├── ResourceQuota (limits)
    ├── Pod 1
    ├── Pod 2
    └── Pod N
```

Pods must fit within the namespace budget.

### 📄 Simple ResourceQuota Example
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```
This means for the namespace:

- Total CPU requests ≤ 2 cores
- Total CPU limits ≤ 4 cores  
Both rules are enforced independently.

### ❌ What Happens When Quota Is Exceeded?

- Pod is NOT created
- Scheduler rejects it
- Error is returned immediately

Example error:
```
exceeded quota: dev-quota
```



### 🧠 Why Kubernetes Needs BOTH

Without ResourceQuota:

- One team can exhaust cluster
- Control namespace behavior

Without Requests & Limits:

- One Pod can crash node
- Control Pod behavior

# 🏨 Complete Scenario (Rooms + People) — FULL EXPLANATION

## Hotel Details (Fixed Capacity)

- The hotel has 2 rooms
- Each room can hold up to 2 people

So the hotel capacity is:

- Rooms available = 2
- Maximum people allowed inside = 4

👉 This is where “4 people” comes from

🔁 Kubernetes Mapping (IMPORTANT)
```
| Hotel Concept      | Kubernetes Concept          |
|--------------------|-----------------------------|
| Rooms              | `requests.cpu = 2`          |
| People allowed     | `limits.cpu = 4`            |
| Guest              | Pod                         |
| Booking a room     | Pod creation                |
| People using room  | CPU usage at runtime        |
```
👥 Current Situation (Existing Pods)

Two guests are already inside the hotel.
```
| Guest   | Rooms Used | People Inside |
|---------|------------|---------------|
| Guest A | 1          | 1             |
| Guest B | 1          | 1             |
```
Hotel Status Now

- Rooms used = 2 / 2 → ❌ FULL
- People inside = 2 / 4 → ✅ SPACE LEFT

This means:

- You cannot accept new guests
- But existing guests can still bring more people(it means the application can  consume more resources when load increases then the limit might reach to 4.)

### 🚪 New Guest Arrives (New Pod)

New guest says:

>“I need 1 room”

Can we accept?

>❌ NO

Why?

- All rooms are already booked
- Even though there is space for more people

🔁 Now Map This EXACTLY to Kubernetes
ResourceQuota
requests.cpu = 2
limits.cpu   = 4

