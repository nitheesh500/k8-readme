# Multi-Container Pod 

We will follow this order:
1. Scenario (why we need it)
2. Problem
3. Solution using a Multi-Container Pod
4. Simple YAML example
5. What is happening (easy explanation)

---

## 📌 Scenario (Why Multi-Container Pod?)

Imagine this situation:

You have an application that:
- Writes logs to a file
- You want to **read those logs continuously**

You have two choices:
- Put everything in **one container** ❌ (bad practice)
- Split responsibilities into **two containers** ✅

This is where **Multi-Container Pods** are used.

---

## ❓ The Problem

One container:
- Runs the application
- Writes logs to a file

Another container:
- Reads the same log file
- Prints or ships logs

Question:
> How can two containers **share files and run together**?

---

## ✅ The Solution: Multi-Container Pod

A **Multi-Container Pod** allows:
- Multiple containers
- Shared storage (volume)
- Same lifecycle (start & stop together)

All containers:
- Run inside **one Pod**
- Share the same network
- Can share the same files

---

## 🧠 Simple Mental Model

```
Pod
├── Container 1 (App)
├── Container 2 (Log Reader)
└── Shared Volume
```


---

## 📄 Simple Multi-Container Pod Example

This example uses:
- One container to write logs
- One container to read logs

```
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "while true; do echo Hello from app >> /data/app.log; sleep 5; done"]
    volumeMounts:
    - name: shared-data
      mountPath: /data

  - name: log-reader
    image: busybox
    command: ["sh", "-c", "tail -f /data/app.log"]
    volumeMounts:
    - name: shared-data
      mountPath: /data

  volumes:
  - name: shared-data
    emptyDir: {}
```

## 🔍 What is Happening Here? 
```
Pod: multi-container-pod
│
├── Container 1: app
│   - writes logs to a file
|   - Runs continuously
│
└── Container 2: log-reader
    - reads that same file
    - Prints logs continuously
```


**Volume: emptyDir**
- Shared between containers
- Exists as long as the Pod exists

### 🔄 How Containers Communicate

- They do NOT talk via IP
- They share files using a volume
- This is very common in real systems

Check logs of the second container:
```
kubectl logs multi-container-pod -c log-reader
```
You will see logs written by the first container.

```
+====================================================+
|                  Kubernetes Cluster                |
|                                                    |
|  +----------------------------------------------+  |
|  |                  Node                        |  |
|  |                                              |  |
|  |  +----------------------------------------+  |  |
|  |  |                Pod                     |  |  |
|  |  |                                        |  |  |
|  |  |  +-------------+    +---------------+  |  |  |
|  |  |  | Container A |    | Container B   |  |  |  |
|  |  |  |   (app)     |    | (log-reader)  |  |  |  |
|  |  |  |             |    |               |  |  |  |
|  |  |  |  /data ---- |----| ---- /data    |  |  |  |
|  |  |  |   |         |    |        |      |  |  |  |
|  |  |  | app.log     |    | tail -f       |  |  |  |
|  |  |  +-------------+    +---------------+  |  |  |
|  |  |                                        |  |  |
|  |  |     Shared Volume (emptyDir)           |  |  |
|  |  +----------------------------------------+  |  |
|  |                                              |  |
|  +----------------------------------------------+  |
|                                                    |
+====================================================+
```