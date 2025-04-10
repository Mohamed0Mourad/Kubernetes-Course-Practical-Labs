# 🚪 Kubernetes Ingress Lab: Path & Host-Based Routing

![Kubernetes](https://img.shields.io/badge/Kubernetes-Ingress-blue)
![Nginx](https://img.shields.io/badge/Controller-NGINX-orange)
![Routing](https://img.shields.io/badge/Routing-Host%20%26%20Path-green)

This lab demonstrates Kubernetes Ingress for both path-based and host-based routing using Nginx Ingress Controller.

## 🛠️ Prerequisites

- Kind (Kubernetes in Docker) installed
- kubectl configured
- Basic understanding of Kubernetes concepts

## 🚀 Cluster Setup with Ingress

### 1. Create Kind Cluster with Ingress Support
```bash
# Delete existing cluster if any
kind delete cluster

# Create new cluster with ingress configuration
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF

# Install Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# Wait for controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

## 📦 Application Deployment

### 1. Main Website Service (www)
```yaml
# www.yaml
apiVersion: v1
kind: Pod
metadata:
  name: www
  labels:
    app: www
spec:
  containers:
  - name: www
    image: nginx
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html
    configMap:
      name: www-content

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: www-content
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Main Website</title>
    </head>
    <body>
      <h1>Welcome to www.example.com</h1>
    </body>
    </html>

---
apiVersion: v1
kind: Service
metadata:
  name: www
spec:
  selector:
    app: www
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 2. API Service
```yaml
# api.yaml
apiVersion: v1
kind: Pod
metadata:
  name: api
  labels:
    app: api
spec:
  containers:
  - name: api
    image: ealen/echo-server:latest
    ports:
    - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 3. Admin Service
```yaml
# admin.yaml
apiVersion: v1
kind: Pod
metadata:
  name: admin
  labels:
    app: admin
spec:
  containers:
  - name: admin
    image: ealen/echo-server:latest
    env:
    - name: ECHO_SERVER_BASE_PATH
      value: "/admin"
    ports:
    - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: admin
spec:
  selector:
    app: admin
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 4. Ingress Configuration
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
spec:
  ingressClassName: nginx
  rules:
  - host: www.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: www
            port:
              number: 80
      - path: /admin
        pathType: Exact
        backend:
          service:
            name: admin
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 80
```

### Apply All Configurations
```bash
kubectl apply -f www.yaml -f api.yaml -f admin.yaml -f ingress.yaml
```

## 🔍 Verification

### Check Resources
```bash
kubectl get pods,svc,ingress
```

### Update Local Hosts File
Add these entries to `/etc/hosts`:
```
127.0.0.1 api.example.com
127.0.0.1 www.example.com
```

## 🧪 Testing the Routes

### Test Routes
1. **Main Website**: http://www.example.com
2. **Admin Portal**: http://www.example.com/admin
3. **API Endpoint**: http://api.example.com
4. **API Subpath**: http://api.example.com/something (should work due to Prefix)
5. **Admin Subpath**: http://www.example.com/admin/something (should NOT work due to Exact match)

## 📊 Architecture Overview

```
┌───────────────────────────────────────────────────────┐
│                   Kubernetes Cluster                  │
│ ┌─────────────────────────────────────────────────┐ │
│ │                 Nginx Ingress                   │ │
│ │ ┌───────────────┐ ┌─────────────┐ ┌───────────┐ │ │
│ │ │ Host-Based    │ │ Path-Based  │ │ TLS       │ │ │
│ │ │ Routing       │ │ Routing     │ │ Termination│ │ │
│ │ └───────┬───────┘ └──────┬──────┘ └───────────┘ │ │
│ └─────────┼────────────────┼──────────────────────┘ │
│           │                │                        │
│ ┌─────────▼──────┐ ┌───────▼────────┐               │
│ │ api.example.com│ │www.example.com │               │
│ │ (api service)  │ │ / (www)        │               │
│ └────────────────┘ │ /admin (admin) │               │
│                    └────────────────┘               │
└───────────────────────────────────────────────────────┘
```

## 🧠 Key Concepts Demonstrated

- **Host-based routing**: Different services for different domains
- **Path-based routing**: Different paths route to different services
- **Path types**:
  - `Exact`: Matches only the exact path
  - `Prefix`: Matches the path and all subpaths
- **Ingress Controller**: Nginx as the ingress implementation

## 🧹 Cleanup
```bash
kind delete cluster
```
