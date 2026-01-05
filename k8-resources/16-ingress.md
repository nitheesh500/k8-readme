# Kubernetes Ingress 

This guide explains **Ingress** in a simple, real-world way.

We will follow this order:
1. Scenario (real production problem)
2. Why Service types are not enough
3. What Ingress is
4. How Ingress works
5. Simple Ingress example
6. Traffic flow diagrams
7. Why Ingress is production-ready
8. Key takeaways

---

## 📌 Scenario (Why Ingress?)

Imagine this real situation:

### 1️⃣ First, understand the BASIC problem (before Ingress)

Imagine you have 3 applications in Kubernetes:

- Frontend
- Backend API
- Auth service

❌ Without Ingress (bad approach)

You create 3 LoadBalancers:
```
Frontend  → LoadBalancer → frontend.example.com
Backend   → LoadBalancer → backend.example.com
Auth      → LoadBalancer → auth.example.com
```
### 🚨 Problems

- 💰 Each LoadBalancer costs money
- 😵 Too many public IPs / DNS records
- 🔐 SSL/TLS setup repeated for each app
- ❌ No URL routing like /api or /auth

👉 This is NOT scalable
### You want:
- One domain (example.com)
- Path-based routing
- HTTPS (SSL)
- Clean URLs

Question:
> Can we do this with ClusterIP or NodePort?

❌ Not properly.

---

## ❓ Problem with Service Types

### ClusterIP
- Internal only
- No external access

### NodePort
- Exposes random ports (30000+)
- No DNS
- No SSL
- Hard to manage

### LoadBalancer
- One LoadBalancer per Service
- Expensive
- No path-based routing

---

## ✅ The Solution: Ingress

**Ingress** is a Kubernetes object that:
- Routes HTTP/HTTPS traffic
- Uses host-based routing
- Uses path-based routing
- Terminates SSL (TLS)
- Exposes **multiple services using ONE LoadBalancer**

```
In short: Ingress allows you to expose MULTIPLE services using ONE external entry point based on URL paths or hostnames.
```
Think of Ingress as:
> A **smart HTTP reverse proxy** for Kubernetes

---

## 🧩 Important: Ingress vs Ingress Controller
```
| Component          |      Role                      |
|--------------------|--------------------------------|
| Ingress            | YAML rules (paths, hosts, TLS) |
| Ingress Controller | Actual traffic handler |
```
Ingress **alone does nothing**.

You must install an **Ingress Controller**:
- NGINX Ingress
- AWS ALB Ingress
- Traefik

## 🔹 Ingress Resource
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
```

### 🔹 Backend Service (ClusterIP)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80
```
### 🔹 Frontend Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: ClusterIP
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```
### What this Ingress rule means (plain English)
```
If request is app.example.com/      → frontend
If request is app.example.com/api  → backend
```

That’s it.
That’s the CORE of Ingress.

## Final mental model 
```
Client
  ↓
DNS
  ↓
Cloud LoadBalancer
  ↓
Ingress Controller
  ↓
ClusterIP Services
  ↓
Pods
```
By default ingress controller is not installed    
## [Ingress controler installation procedure](https://github.com/nitheesh500/k8-ingress/blob/main/README.md)

### 🎤 Interview-ready explanation (LONG & STRONG)

- Ingress is a Kubernetes API object used to manage external HTTP and HTTPS access to services within a cluster. It provides Layer-7 routing features such as host-based and path-based routing and TLS termination. Ingress rules are implemented by an Ingress Controller, which receives traffic from a cloud load balancer and routes requests to appropriate ClusterIP services.
