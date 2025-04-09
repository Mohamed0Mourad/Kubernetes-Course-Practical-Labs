
# Kubernetes Multi-Container Pod with Volumes and Probes

![Kubernetes](https://img.shields.io/badge/Kubernetes-Pod-blue)
![NFS](https://img.shields.io/badge/Storage-NFS%20%26%20emptyDir-orange)
![Liveness](https://img.shields.io/badge/Probes-Liveness%20%26%20Logging-green)

This configuration demonstrates a Kubernetes pod with:
- Main Nginx container with liveness probe
- Sidecar logging container
- Both emptyDir and NFS volume types

## 🛠️ Prerequisites

- Kubernetes cluster
- NFS server (can be local machine)
- `kubectl` configured

## 🚀 NFS Server Setup

```bash
sudo apt update
sudo apt install nfs-kernel-server -y
sudo mkdir -p /mnt/shared
sudo chown nobody:nogroup /mnt/shared
sudo chmod 777 /mnt/shared

# Edit exports file
sudo vim /etc/exports
```

Add this line to `/etc/exports`:
```
/mnt/shared *(rw,no_root_squash,insecure,sync,no_subtree_check)
```

Start NFS service:
```bash
sudo exportfs -a
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

## 📋 Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo
spec:
  volumes:
    - name: emptydir-volume
      emptyDir: {}
    - name: nfs-volume
      nfs:
        server: 192.168.2.138 # Replace with your NFS server IP
        path: /mnt/shared
  containers:
    - name: main-container
      image: nginx:alpine
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
      volumeMounts:
        - name: emptydir-volume
          mountPath: /usr/share/nginx/html
        - name: nfs-volume
          mountPath: /mnt/nfs
      ports:
        - containerPort: 80
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 30
        periodSeconds: 10
    - name: sidecar-container
      image: busybox
      command: ["/bin/sh", "-c", "while true; do echo $(date) '- Sidecar Logging'; sleep 5; done"]
      resources:
        requests:
          memory: "16Mi"
          cpu: "110m"
        limits:
          memory: "32Mi"
          cpu: "200m"
      volumeMounts:
        - name: emptydir-volume
          mountPath: /mnt/shared
```

## 🔍 Verification

1. Deploy the pod:
```bash
kubectl apply -f pod.yaml
```

2. Check pod status:
```bash
kubectl get pods
```

3. Access main container:
```bash
kubectl exec -it demo -c main-container -- sh
# Create test file in NFS volume:
echo "Test" > /mnt/nfs/test.txt
```

4. Verify on NFS server:
```bash
cat /mnt/shared/test.txt
```

5. View sidecar logs:
```bash
kubectl logs -f demo -c sidecar-container
```

## 📊 Architecture Overview

```
┌─────────────────────────────────┐
│           Kubernetes Pod        │
│  ┌───────────────────────────┐  │
│  │       main-container      │  │
│  │  - Nginx:alpine           │  │
│  │  - Liveness probe         │  │
│  │  - emptyDir & NFS volumes │  │
│  └──────────────┬────────────┘  │
│                 │               │
│  ┌──────────────▼────────────┐  │
│  │    sidecar-container      │  │
│  │  - Busybox                │  │
│  │  - Continuous logging     │  │
│  │  - emptyDir volume        │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

## 🧠 Key Features

- **Multi-container pod** pattern
- **emptyDir** for inter-container communication
- **NFS** for persistent storage
- **Resource limits** for both containers
- **Liveness probe** for health monitoring
- **Sidecar pattern** for logging

## 🚨 Cleanup

```bash
kubectl delete pod demo
```
