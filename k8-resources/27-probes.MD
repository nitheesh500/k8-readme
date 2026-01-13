# 🩺 Kubernetes Probes (Liveness, Readiness, Startup)
A Complete 0 → Hero Guide

This README explains **Kubernetes probes in depth**, including **why they exist, how they work internally, when containers restart, real timelines, examples, best practices, and common mistakes**.

This is written for:
- Beginners who want clarity
- Engineers preparing for interviews
- Production Kubernetes users

---

## 📌 Why Kubernetes Probes Exist (The Real Problem)

Kubernetes can only see:
- Whether a container **process is running**
- Whether the container **has crashed**

But Kubernetes **cannot know**:
- If your app is **stuck**
- If your app is **starting slowly**
- If your app is **alive but cannot handle traffic**

### Real-world failures without probes:
- App deadlocked → users get 500s forever
- App starts slowly → traffic hits too early
- DB temporarily down → app should stop receiving traffic, not restart

👉 **Probes allow Kubernetes to ASK your app questions instead of guessing.**

---

## 🧠 What Is a Probe?

A **probe** is a health check run by Kubernetes against a container.

Kubernetes uses probes to decide:
- Should I **restart** the container?
- Should I **send traffic** to this Pod?
- Should I **wait longer** for startup?

---

## 🧪 Types of Kubernetes Probes

Kubernetes provides **three probes**, each with a **different responsibility**.

| Probe Type | Question Kubernetes Asks | Result of Failure |
|-----------|--------------------------|------------------|
| Startup   | “Have you finished starting?” | Restart (after threshold) |
| Liveness  | “Are you still alive?” | Restart container |
| Readiness | “Can you accept traffic?” | Remove from Service |

🚨 These probes are **NOT interchangeable**.

---

## 🚀 Startup Probe (Deep Explanation)

### What problem does Startup Probe solve?

Some applications:
- Take a long time to boot
- Run migrations
- Warm caches
- Load large configs

Without startup probe:
- Liveness probe runs too early
- Container gets killed before it finishes starting

---

### What Startup Probe Does

- Runs **only during application startup**
- Blocks **liveness and readiness probes**
- Gives the app **time to initialize**
- Restarts the container **only if startup NEVER completes**

---

### 🚨 Important Truth (Common Confusion)

❌ Startup probe does NOT restart on first failure  
✅ Startup probe restarts ONLY after exceeding `failureThreshold`

---

### Example Startup Probe

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  periodSeconds: 10
  failureThreshold: 30
```

Meaning:

- Kubernetes checks every 10 seconds
- Allows 30 failures
- App has 300 seconds (5 minutes) to start
- Restart happens only if startup never succeeds

### Startup Probe Timeline
```
Container starts
↓
startupProbe FAIL (normal)
↓
startupProbe FAIL (normal)
↓
startupProbe SUCCESS
↓
startupProbe stops
↓
readiness + liveness start
```

If startup NEVER succeeds → container restarts.

---
## ❤️ Liveness Probe (Deep Explanation)
**What problem does Liveness Probe solve?**

Sometimes apps:

- Deadlock
- Hang
- Stop responding
- But the process is still running

Without liveness probe:

- Kubernetes thinks app is healthy
- Users see downtime forever

### What Liveness Probe Does

- Runs after startup completes
- Checks app continuously
- If it fails repeatedly → container is restarted

### Example Liveness Probe
```
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 10
  failureThreshold: 3
```

Meaning:

- 3 failures × 10 seconds
- Container restarted after ~30 seconds of failure

🚨 Liveness Probe Rule (VERY IMPORTANT)

❌ Do NOT check DB or external services  
❌ Do NOT make it “smart”  

Why?

- Temporary DB outage ≠ app is dead
- You’ll cause restart loops

✅ Liveness probe should be simple  
(e.g., event loop running, process responsive)

---
## 🚦 Readiness Probe (Deep Explanation)
**What problem does Readiness Probe solve?**

An app may be:

- Running
- Healthy
- But temporarily unable to serve traffic

Examples:

- DB connection lost
- Cache warming
- App in maintenance mode
- Rolling deployment
---
### What Readiness Probe Does

- Controls traffic routing
- If it fails:
  - Pod stays running
  - Pod is removed from Service endpoints
- NO restart happens

### Example Readiness Probe
```
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 5
  failureThreshold: 1
```

### Traffic Flow with Readiness Probe
```
Client
 ↓
Service
 ↓
Only Pods with readiness = TRUE
```

Readiness is essential for:

- Zero-downtime deployments
- Graceful shutdowns
- Dependency failures

### 🔁 How Probes Work Together (Lifecycle)
```
Container starts
↓
Startup Probe runs
↓
Startup SUCCESS
↓
Readiness Probe runs
↓
Traffic allowed
↓
Liveness Probe runs continuously
```
---
### 🔥 What Happens When Probes Fail
| Probe Type      | Failure Effect                      |
|-----------------|-------------------------------------|
| Startup Probe   | Restart container (after threshold) |
| Liveness Probe  | Restart container                   |
| Readiness Probe | Stop traffic only                   |