
# ⚙️ Kubernetes Lab: ReplicaSet Deep Dive

![Kubernetes](https://img.shields.io/badge/Kubernetes-ReplicaSet-blue)
![Scaling](https://img.shields.io/badge/Scaling-Horizontal-green)
![Management](https://img.shields.io/badge/Pod--Management-Automated-orange)

This lab demonstrates how to define, deploy, and manage a Kubernetes **ReplicaSet**. It also shows how to scale, inspect, and safely delete ReplicaSets while preserving pods.

---

## 🛠️ Prerequisites

- A Kubernetes cluster (Kind, Minikube, or any setup)
- `kubectl` installed and configured
- Basic knowledge of Kubernetes workloads

---

## 📦 Step 1: Create a ReplicaSet

### 1️⃣ Create `replicaset.yaml` file

```yaml
# replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: web
        image: nginx
```

---

### 2️⃣ Apply the ReplicaSet to the cluster

```bash
kubectl apply -f replicaset.yaml
```

**Output:**
```
replicaset.apps/myapp created
```

---

## 🔍 Step 2: Verify ReplicaSet and Pods

```bash
kubectl get rs
```

**Output:**
```
NAME    DESIRED   CURRENT   READY   AGE
myapp   3         3         3       10s
```

```bash
kubectl get pods
```

**Output:**
```
NAME                    READY   STATUS    RESTARTS   AGE
myapp-bb5d867cc-4zn7t   1/1     Running   0          10s
myapp-bb5d867cc-hvkgp   1/1     Running   0          10s
myapp-bb5d867cc-pq9f8   1/1     Running   0          10s
```

Notice the generated pod names — Kubernetes appends a unique hash to the ReplicaSet name.

---

## 🔄 Step 3: Test Pod Recreation

Delete one pod:

```bash
kubectl delete pod myapp-bb5d867cc-4zn7t
```

**Output:**
```
pod "myapp-bb5d867cc-4zn7t" deleted
```

Check again:

```bash
kubectl get pods
```

**Output:**
```
NAME                    READY   STATUS    RESTARTS   AGE
myapp-bb5d867cc-hvkgp   1/1     Running   0          2m
myapp-bb5d867cc-pq9f8   1/1     Running   0          2m
myapp-bb5d867cc-w6ks9   1/1     Running   0          5s
```

✅ A new pod `myapp-bb5d867cc-w6ks9` was created automatically by the ReplicaSet!

---

## 📖 Step 4: Describe the ReplicaSet

```bash
kubectl describe rs myapp
```

**Sample Output Snippet:**
```
Name:         myapp
Namespace:    default
Selector:     app=myapp
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
```

---

## 🔗 Step 5: Pod–ReplicaSet Relationship

### 🔎 Find which ReplicaSet owns a specific pod

```bash
kubectl get pod myapp-bb5d867cc-pq9f8 -o=jsonpath='{.metadata.ownerReferences[0].name}'
```

**Output:**
```
myapp-bb5d867cc
```

---

### 🔍 Find pods managed by a specific ReplicaSet

```bash
kubectl get pods -l app=myapp
```

**Output:**
```
NAME                    READY   STATUS    RESTARTS   AGE
myapp-bb5d867cc-hvkgp   1/1     Running   0          5m
myapp-bb5d867cc-pq9f8   1/1     Running   0          5m
myapp-bb5d867cc-w6ks9   1/1     Running   0          3m
```

---

## 📈 Step 6: Scale the ReplicaSet

### Imperative scaling

```bash
kubectl scale rs myapp --replicas=4
```

**Output:**
```
replicaset.apps/myapp scaled
```

```bash
kubectl get pods
```

**Output:**
```
NAME                    READY   STATUS    RESTARTS   AGE
myapp-bb5d867cc-hvkgp   1/1     Running   0          5m
myapp-bb5d867cc-pq9f8   1/1     Running   0          5m
myapp-bb5d867cc-w6ks9   1/1     Running   0          3m
myapp-bb5d867cc-xj9lm   1/1     Running   0          10s
```

✅ You now have 4 pods running!

---

### Declarative scaling

Update `replicas` value in `replicaset.yaml`:

```yaml
spec:
  replicas: 4
```

Apply the file again:

```bash
kubectl apply -f replicaset.yaml
```

---

## 🤖 Step 7: Autoscaling Concepts

Kubernetes supports:

- **Vertical Scaling**: Increase CPU/memory of existing pods.
- **Horizontal Scaling**: Increase the number of pods.

To use **Horizontal Pod Autoscaler (HPA)**:

1. Install `metrics-server`
2. Create an HPA based on CPU or memory

---

## 🧹 Step 8: Delete the ReplicaSet

### Delete ReplicaSet AND pods (default behavior)

```bash
kubectl delete rs myapp
```

**Output:**
```
replicaset.apps "myapp" deleted
```

All pods will be terminated.

---

### Delete ReplicaSet ONLY (preserve pods)

```bash
kubectl delete rs myapp --cascade=false
```

**Output:**
```
replicaset.apps "myapp" deleted
```

✅ Pods will stay running but will become **orphaned** (no controller will recreate them if they fail):

```bash
kubectl get pods
```

---

