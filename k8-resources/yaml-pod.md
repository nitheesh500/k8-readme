# YAML Basics → Kubernetes Pod

## 1. What is YAML?

YAML stands for **YAML Ain’t Markup Language**.

Kubernetes uses YAML files to **define resources** such as:
- Pods
- Deployments
- Services

YAML is:
- Human readable
- Indentation based
- Written using key–value pairs

---

## 2. Basic YAML Rules (Must Know)

### 2.1 Indentation matters

```yaml
parent:
  child: value
```
### 2.2 Key : Value format
```
name: nginx-pod
```

### 2.3 Lists use -
```
containers:
- name: nginx
  image: nginx
```

### 2.4 Use spaces, not tabs

## 3. Kubernetes YAML File Structure

Every Kubernetes YAML file follows this same structure:
```
apiVersion:
kind:
metadata:
spec:
```

What each field means (simple)

| Field       | Meaning                         |
|------------|----------------------------------|
| apiVersion | Kubernetes API version           |
| kind       | Type of Kubernetes object        |
| metadata   | Name and labels                  |
| spec       | How the object should run        |


## 4️⃣ Solving the Scenario: Create a Pod

Now let’s solve our original problem:
```
“I want to run nginx using Kubernetes”
```

The smallest Kubernetes object that can run a container is a Pod.

## 5️⃣ Minimal Pod Example

This is the simplest working Pod.
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx

```

What this does:  
- Creates a Pod named nginx-pod
- Runs one container
- Uses the nginx image

## 6️⃣ What is inside spec 

For now, remember only this:
```
spec:
  containers:
```

- spec tells Kubernetes how the Pod should run
- containers tells Kubernetes what to run

A Pod cannot exist without containers.

## 7️⃣ What Happens Next?

If you apply this file:
```
kubectl apply -f pod.yaml
```

Kubernetes will:
- Create the Pod
- Pull the nginx image
- Start the container

---

## 8️⃣ Extending the Scenario: Application Needs a Port

Let’s continue our scenario.

We already ran **nginx** inside a Pod.  
Now ask a simple question:

> “My application runs on port 80.  

> How does Kubernetes know this?”

To answer this, we add **ports** to the Pod.

---

## 9️⃣ Adding Ports to the Pod

Ports are defined **inside the container section**.

```yaml
ports:
- containerPort: 80
```
This means:

- The container listens on port 80
- Kubernetes is now aware of this port

⚠️ Important  Rule:  
- **Defining a port does NOT mean the app is accessible from browser**
---
## 🔟 Pod Example with Port (Updated)

Now our Pod YAML looks like this:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

What changed:

- Only ports was added
- Everything else remains the same  
---
## 1️⃣1️⃣ Can We Access the Application Now?
Short answer

❌ NO — not from your browser

Why not?  
Because:  
- Pod is internal to the cluster
- No Service is created
- Kubernetes has not exposed it outside

containerPort only tells:

> “The app listens on this port”

It does not mean:

> “Make this app public”
---
## 1️⃣2️⃣ Where Can We Access It From?
| From where                 | Accessible |
|----------------------------|------------|
| Inside the same Pod        | ✅ Yes     |
| From another Pod           | ✅ Yes     |
| From laptop / browser      | ❌ No      |
---
## 1️⃣3️⃣ How Can We Test It?
Access from inside the Pod
```
kubectl exec -it nginx-pod -- curl localhost:80
```

✔️ This works because:
- The container can access itself using localhost 
---