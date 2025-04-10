# 🌐 Kubernetes ClusterIP & NodePort Services Lab

![Kubernetes](https://img.shields.io/badge/Kubernetes-Services-blue)
![ClusterIP](https://img.shields.io/badge/Service-ClusterIP-green)
![NodePort](https://img.shields.io/badge/Service-NodePort-orange)

This lab demonstrates Kubernetes Service types (ClusterIP and NodePort) with practical examples.

## 🚀 Quick Start

### 1. Create Two Nginx Pods
```yaml
# pods.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app01
  labels:
    app: web-app
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80

---
apiVersion: v1
kind: Pod
metadata:
  name: web-app02
  labels:
    app: web-app
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
```

Apply the manifest:
```bash
kubectl apply -f pods.yaml
kubectl get pods
```

### 2. Customize Each Pod
```bash
# For web-app01
kubectl exec -it web-app01 -- bash -c "apt update && apt install -y vim && echo '<h1>I am Pod 01</h1>' > /usr/share/nginx/html/index.html"

# For web-app02
kubectl exec -it web-app02 -- bash -c "apt update && apt install -y vim && echo '<h1>I am Pod 02</h1>' > /usr/share/nginx/html/index.html"
```

## 🔍 ClusterIP Service

### Create ClusterIP Service
```yaml
# clusterip-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Apply and verify:
```bash
kubectl apply -f clusterip-service.yaml
kubectl get svc
```

### Test ClusterIP Service
```bash
# Create temporary Ubuntu pod
kubectl run ubuntu --image=ubuntu -it --rm -- /bin/bash -c "apt update && apt install -y curl && curl my-clusterip-service.default.svc.cluster.local"

# Alternative test (from your local machine)
kubectl port-forward service/my-clusterip-service 8080:80
# Then open http://localhost:8080 in browser
```

## 🌐 NodePort Service

### Create NodePort Service
```yaml
# nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  selector:
    app: web-app
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
```

Apply and verify:
```bash
kubectl apply -f nodeport-service.yaml
kubectl get svc
kubectl get nodes -o wide
```

### Test NodePort Service
Access via any node's IP at port 30007:
```
http://<node-ip>:30007
```

## 📊 Architecture Overview

```
┌───────────────────────────────────────────────────────┐
│                   Kubernetes Cluster                  │
│ ┌───────────────────┐    ┌───────────────────┐      │
│ │      Node 1        │    │      Node 2        │      │
│ │ ┌───────────────┐  │    │ ┌───────────────┐  │      │
│ │ │  web-app01    │  │    │ │  web-app02    │  │      │
│ │ │  (nginx)      │◄─┼────┼─┤  (nginx)      │  │      │
│ │ └───────────────┘  │    │ └───────────────┘  │      │
│ │                    │    │                    │      │
│ │ ClusterIP Service  │    │ NodePort Service   │      │
│ │ (Internal access)  │    │ (External access)  │      │
│ └───────────────────┘    └───────────────────┘      │
└───────────────────────────────────────────────────────┘
```

## 🧠 Key Concepts Demonstrated

- **ClusterIP**: Internal service discovery
- **NodePort**: External access to services
- **Label Selectors**: How services find pods
- **Port Forwarding**: Local access to cluster services
- **Multi-Pod Load Balancing**: Requests distributed between pods

## 🧹 Cleanup
```bash
kubectl delete pod web-app01 web-app02
kubectl delete svc my-clusterip-service my-nodeport-service
```
