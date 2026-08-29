# Kubernetes Hands-On Exercise Series

Welcome to Exercise-1 **Kubernetes (K8s) exercises**!  
These activities will help you understand the basics of how Kubernetes runs and manages containerized applications.  

## Business Problem (Zepto Example)

Imagine you are a **DevOps Engineer at Zepto**.  
The product team just built a lightweight **web app** that shows the **storefront and delivery status page** for customers.  

Your task as the DevOps engineer:  
**Deploy this app on Kubernetes** so that it is always running, portable, and can be scaled later.  
Simulate this using the popular `nginx` container image (think of it as Zepto’s storefront web app).

---

## Exercise 1: Hello Pod

**Goal:** Run your first app inside Kubernetes and access it.

### Project Files
- `pod.yaml` - Kubernetes declarative manifest for the Nginx Pod (`hello-k8s`).
- `service.yaml` - Kubernetes declarative manifest for the NodePort Service (`hello-k8s`).

---

## Method 1: Declarative Deployment (Using YAML Files)

### 1. Start a local Kubernetes cluster with Minikube:
```bash
minikube start
```

### 2. Deploy the Pod using `pod.yaml`:
```bash
kubectl apply -f pod.yaml
```

### 3. Verify the Pod is running:
```bash
kubectl get pods
```

### 4. Deploy the Service using `service.yaml`:
```bash
kubectl apply -f service.yaml
```

### 5. Verify the Service:
```bash
kubectl get svc hello-k8s
```

### 6. Open the app in your browser:
```bash
minikube service hello-k8s
```

---

## Method 2: Imperative Commands (CLI-only)

```bash
# 1. Run Pod
kubectl run hello-k8s --image=nginx --port=80

# 2. Check Pod status
kubectl get pods

# 3. Expose as Service
kubectl expose pod hello-k8s --type=NodePort --port=80

# 4. Open in browser
minikube service hello-k8s
```

---

## System Internal Details

```
[kubectl CLI]
      | (1) REST API JSON to kube-apiserver
      v
+-------------------------------------------------------------+
|                        CONTROL PLANE                        |
|                                                             |
|  +--------------------+         (2) Save Desired State      |
|  |   kube-apiserver   +------------------------------+      |
|  +---------+----------+                              |      |
|            |                                         v      |
|            | (3) Watch Unscheduled Pods          +-------+  |
|            v                                     | etcd  |  |
|  +--------------------+                          +-------+  |
|  |   kube-scheduler   |                                     |
|  +---------+----------+                                     |
+------------|------------------------------------------------+
             | (4) Assigns Pod to Worker Node
             v
+-------------------------------------------------------------+
|                         WORKER NODE                         |
|                                                             |
|  +--------------------+                                     |
|  |      kubelet       | (Node Agent)                        |
|  +---------+----------+                                     |
|            | (5) Request Container Creation                 |
|            v                                                |
|  +--------------------+                                     |
|  |   CRI / containerd | Pulls 'nginx', configures cgroups   |
|  +---------+----------+                                     |
|            | (7) Network Interface Allocation               |
|            v                                                |
|  +--------------------+                                     |
|  |        CNI         | Assigns Pod Cluster IP              |
|  +--------------------+                                     |
|            |                                                |
|            +----------> (8) Updates API Server:             |
|                             "Pod is Running"                |
+-------------------------------------------------------------+
```

1. **Step 1:** `kubectl` CLI connects to `kube-apiserver` and sends a Pod Manifest (in JSON): Pod name `hello-k8s`, container image `nginx`, port `80`.
2. **Step 2:** `kube-apiserver` receives the request and stores the Pod manifest in `etcd`.
3. **Step 3:** `kube-scheduler` watches for states in `etcd` and chooses a worker node that has sufficient CPU and memory.
4. **Step 4:** `kube-scheduler` assigns the pod to the `kubelet` agent running on that worker node.
5. **Step 5:** `kubelet` instructs the Container Runtime (CRI) to pull the `nginx` image from the registry and create the container.
6. **Step 6:** Sets up the filesystem, mounts volumes, and runs the process inside isolated Linux namespaces and cgroups.
7. **Step 7:** CNI (Container Network Interface) assigns a unique IP address to the Pod that is reachable inside the cluster.
8. **Step 8:** `kubelet` updates the API Server: *"Pod hello-k8s is running"*.

---

## Clean Up
```bash
kubectl delete -f service.yaml
kubectl delete -f pod.yaml
```
