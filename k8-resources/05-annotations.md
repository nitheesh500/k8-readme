# Kubernetes Annotations (Beginner Guide)

This guide explains **Annotations** in a simple and practical way.

We will follow this order:
1. Scenario (why annotations exist)
2. The problem
3. The solution (annotations)
4. Simple YAML example
5. Labels vs Annotations (very important)
6. Real-world usage

---

## 📌 Scenario (Why do we need Annotations?)

Imagine this situation:

You have a Pod running in Kubernetes.
- DevOps wants to add **extra information**
- Tools like monitoring, ingress, or CI/CD need **metadata**
- This information is **not used for selecting Pods**

Question:
> Where do we store extra information that Kubernetes should not use for filtering?

---

## ❓ The Problem (Why Labels Are Not Enough)

We already learned **labels**.

Labels are:
- Used for grouping
- Used by Services and Deployments
- Used in selectors

If we put **too much data** in labels:
- Selectors become messy
- Labels lose their purpose

Example (bad idea):
```text
build-number=2024-09-15-commit-abcdef
```

❌ Labels are NOT meant for this.

>✅ The Solution: **Annotations**

**Annotations** are key–value pairs used to **store extra information**.

Annotations are used for:
- Documentation
- Tooling
- Metadata for external systems

Annotations are:
- NOT used for selection
- NOT used by Services
- NOT meant for filtering

Think of annotations as:

>📝 Notes attached to a resource

### 🧠 Simple Mental Model
```
Pod
├── Labels        → Used for grouping & selection
└── Annotations   → Used for extra information
```
### 📄 Adding Annotations to a Pod

Annotations are added under metadata, just like labels.
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  annotations:
    description: "This pod runs nginx"
    owner: "devops-team"
spec:
  containers:
  - name: nginx
    image: nginx
```

This annotation:
- Is stored with the Pod
- Is readable by tools and humans
- Does NOT affect Pod behavior

### 🔍 Viewing Annotations

Describe the Pod:
```
kubectl describe pod nginx-pod
```

You will see:
```
Annotations:
  description: This pod runs nginx
  owner: devops-team
```
### 🆚 Labels vs Annotations (VERY IMPORTANT)
| Feature            | Labels                      | Annotations                 |
|--------------------|-----------------------------|-----------------------------|
| Purpose            | Identify and group objects  | Store extra information     |
| Used by selectors  | ✅ Yes                      | ❌ No                      |
| Used by Services   | ✅ Yes                      | ❌ No                      |
| Size               | Small                       | Can be large                |
| Filtering          | ✅ Yes                      | ❌ No                      |

### 🧠 Final Mental Model (Important)
```
Kubernetes
  |
  |-- stores annotations
  |
External Tools
  |
  |-- read annotations
  |-- take action
```
###   📌 Golden Rule

>If you want to select → use labels  
>If you want to describe → use annotations

### 🛠️ Real-World Usage Examples

Annotations are commonly used by:
- Ingress controllers
- Monitoring tools
- CI/CD pipelines
- Cloud provider integrations

### ⚠️ Important Beginner Rules

- Annotations do NOT affect scheduling
- Annotations do NOT affect networking
- Kubernetes does NOT act on annotations by default
- Tools decide how to use annotations
---
### 🔹 Example 1: Ingress Controller (NGINX)

Ingress controllers use annotations heavily.

```yaml
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
```
What this means:

- Kubernetes stores this annotation
- The NGINX Ingress Controller reads it
- It knows this Ingress belongs to nginx

📌 Kubernetes does nothing with this by itself.

### 🔹 Example 2: Monitoring Tool (Prometheus)

Prometheus uses annotations to discover metrics.
```
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
```
Meaning:
- Prometheus reads this annotation
- It starts scraping metrics from port 8080

Kubernetes only stores this information

---
## Examples: Metadata for External Systems

Annotations are often used to store information for humans or external platforms.

### 🔹 Example 3: CI/CD Metadata
```
metadata:
  annotations:
    build-number: "1245"
    git-commit: "a1b2c3d4"
```

Used by:
- CI/CD pipelines
- Release tracking
- Debugging deployments

❌ Not used for selection  
✅ Used for traceability

### 🔹 Example 4: Cloud Provider Metadata
```
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

Meaning:
- Kubernetes stores the annotation
- AWS cloud controller reads it
- Creates a Network Load Balancer

Again:
>Kubernetes stores it, external system acts on it.