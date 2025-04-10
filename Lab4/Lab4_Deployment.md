
# 🚀 Kubernetes Lab: Deployment Management

![Kubernetes](https://img.shields.io/badge/Kubernetes-Deployment-blue)
![RollingUpdate](https://img.shields.io/badge/Strategy-RollingUpdate-orange)
![Production](https://img.shields.io/badge/Environment-Production-success)

This lab demonstrates how to create a **Deployment** in Kubernetes, update the application version using a **rolling update**, and monitor the rollout and ReplicaSets.

---

## 📦 Step 1: Define the Deployment Manifest

Create a file called `deployment.yaml`:

```yaml
# deployment.yaml

# API version used to manage deployments
apiVersion: apps/v1

# Define the type of Kubernetes object
kind: Deployment

# Metadata about the deployment
metadata:
  name: sample-deployment         # Name of the Deployment
  labels:
    app: sample-app               # Label to categorize this Deployment

spec:
  replicas: 3                     # Desired number of pods (replica count)

  # Define how to select the pods that this Deployment will manage
  selector:
    matchLabels:
      app: sample-app             # Must match the labels in the pod template

  # Strategy used during rolling updates
  strategy:
    type: RollingUpdate           # RollingUpdate replaces Pods incrementally
    rollingUpdate:
      maxSurge: 1                 # Allows 1 extra pod (above replicas) during update
      maxUnavailable: 1           # Allows 1 pod to be unavailable during update

  minReadySeconds: 10             # Time for a pod to be considered "ready"
  revisionHistoryLimit: 5         # Keeps the last 5 ReplicaSets for rollback
  progressDeadlineSeconds: 600    # Deployment must complete within this time

  # Template for the pod to be created
  template:
    metadata:
      labels:
        app: sample-app           # Must match selector.matchLabels

    spec:
      containers:
      - name: sample-container
        image: nginx:1.21         # NGINX image version 1.21
        ports:
        - containerPort: 80       # Exposes port 80 from the container

        # Define environment variables inside the container
        env:
        - name: ENVIRONMENT
          value: "production"     # Example environment variable

        # Define resource requests and limits
        resources:
          requests:
            cpu: "100m"           # Minimum CPU guaranteed
            memory: "128Mi"       # Minimum memory guaranteed
          limits:
            cpu: "200m"           # Maximum CPU the container can use
            memory: "256Mi"       # Maximum memory the container can use

        # Liveness probe checks if the app is still running
        livenessProbe:
          httpGet:
            path: /               # Root path
            port: 80
          initialDelaySeconds: 15 # Start checking after 15s
          periodSeconds: 20       # Check every 20s

        # Readiness probe checks if the app is ready to receive traffic
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5  # Start checking after 5s
          periodSeconds: 10       # Check every 10s

        # Mount the volume into the container
        volumeMounts:
        - name: sample-volume
          mountPath: /usr/share/nginx/html  # Mount path inside the container

      # Define the volume used by the container
      volumes:
      - name: sample-volume
        emptyDir: {}              # Creates an ephemeral emptyDir volume

```

---

## 🚀 Step 2: Create the Deployment

```bash
kubectl apply -f deployment.yaml
```

**Output:**
```
deployment.apps/sample-deployment created
```

---

## 🔍 Step 3: View the Deployment

```bash
kubectl get deployments
```

**Output:**
```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
sample-deployment   3/3     3            3           10s
```

---

## 📈 Step 4: Monitor the Created Pods

```bash
kubectl get pods -l app=sample-app
```

**Output:**
```
NAME                                 READY   STATUS    RESTARTS   AGE
sample-deployment-7b7bbdc8d7-2c75t   1/1     Running   0          15s
sample-deployment-7b7bbdc8d7-92c2j   1/1     Running   0          15s
sample-deployment-7b7bbdc8d7-v7jfz   1/1     Running   0          15s
```

---

## 🔄 Step 5: Perform a Rolling Update

### 🎯 Update the image from `nginx:1.21` to `nginx:1.27.2`

Modify `deployment.yaml`:

```yaml
image: nginx:1.27.2
```

Then re-apply the file:

```bash
kubectl apply -f deployment.yaml
```

**Output:**
```
deployment.apps/sample-deployment configured
```

---

## 🔎 Step 6: Monitor the Rollout

```bash
kubectl rollout status deployment sample-deployment
```

**Output:**
```
deployment "sample-deployment" successfully rolled out
```

Behind the scenes, Kubernetes created a new ReplicaSet and gradually shifted traffic to the new pods using the rolling update strategy.

---

## 📊 Step 7: View ReplicaSets

```bash
kubectl get replicasets -o wide
```

**Output:**
```
NAME                          DESIRED   CURRENT   READY   AGE     CONTAINERS        IMAGES
sample-deployment-7b7bbdc8d7   0         0         0       2m      sample-container   nginx:1.21
sample-deployment-6cc86df5c9   3         3         3       10s     sample-container   nginx:1.27.2
```

🎉 You can see the old ReplicaSet still exists (scaled down), and a new one was created to handle the new version!

---

### View detailed Deployment info:

```bash
kubectl describe deployment sample-deployment
```

### View history:

```bash
kubectl rollout history deployment sample-deployment
```

**Output:**
```
deployment.apps/sample-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

You can add a change cause with:
```bash
kubectl annotate deployment sample-deployment kubernetes.io/change-cause="Updated nginx image to 1.27.2"
```

---

## 🧹 Cleanup

To delete the Deployment and all its associated pods and ReplicaSets:

```bash
kubectl delete deployment sample-deployment
```

**Output:**
```
deployment.apps "sample-deployment" deleted
```

