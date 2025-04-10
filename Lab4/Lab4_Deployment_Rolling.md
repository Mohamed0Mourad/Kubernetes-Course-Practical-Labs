

### **Rolling Out and Rolling Back a Kubernetes Deployment 🔄🔙**

Kubernetes provides powerful commands to manage the rollout and rollback of deployments. You can pause, resume, and even rollback your deployments based on changes or issues.

---

### **Step 1: Change Replica Count 🧑‍💻**

For demonstration purposes, change the replica count to a high number (e.g., 10).

#### Update the ReplicaCount for Deployment:
```bash
kubectl scale deployment sample-deployment --replicas=10
```
**Output:**
```bash
deployment.apps/sample-deployment scaled ✅
```

---

### **Step 2: Pause a Deployment Rollout ⏸️**

To pause the deployment rollout (e.g., for maintenance or manual intervention), you can use the following command:

#### Pause the Rollout:
```bash
kubectl rollout pause deployments sample-deployment
```
**Output:**
```bash
deployment.apps/sample-deployment paused ⏸️
```

---

### **Step 3: Resume a Deployment Rollout ▶️**

After making the necessary updates or interventions, you can resume the rollout to continue deploying changes:

#### Resume the Rollout:
```bash
kubectl rollout resume deployments sample-deployment
```
**Output:**
```bash
deployment.apps/sample-deployment resumed ▶️
```

---

### **Step 4: Rollback a Deployment 🔙**

In case you need to revert to a previous state, Kubernetes allows you to rollback a deployment to its previous replica set.

#### Rollback to the Old ReplicaSet:
```bash
kubectl rollout undo deployment sample-deployment
```
**Output:**
```bash
deployment.apps/sample-deployment rolled back 🔄
```

---

### **Step 5: Viewing the Rollout History 📜**

Kubernetes keeps track of deployment changes in a rollout history, allowing you to view past revisions.

#### Make Several Changes to the Deployment 🛠️:

1. Change the image to `nginx:1.16.1`:
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Update to nginx 1.16.1"
```
**Output:**
```bash
deployment.apps/nginx-deployment image updated ✅
```

2. Change the image to `nginx:1.17.1`:
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.17.1
kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Update to nginx 1.17.1"
```
**Output:**
```bash
deployment.apps/nginx-deployment image updated ✅
```

3. Change the image to `nginx:1.18.0`:
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.18.0
kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Update to nginx 1.18.0"
```
**Output:**
```bash
deployment.apps/nginx-deployment image updated ✅
```

#### View the Rollout History:
```bash
kubectl rollout history sample-deployment
```
**Output:**
```bash
deployment.apps/sample-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         Update to nginx 1.16.1
3         Update to nginx 1.17.1
4         Update to nginx 1.18.0
```

#### View History of a Specific Revision:
```bash
kubectl rollout history deployment/nginx-deployment --revision=3
```
**Output:**
```bash
deployment.apps/nginx-deployment with revision 3
Pod Template:
  Labels:       app=nginx
  Annotations:  kubernetes.io/change-cause=Update to nginx 1.17.1
  Containers:
   nginx:
    Image:      nginx:1.17.1
    Port:       80/TCP
```

#### Rollout to a Specific Version:
To revert to a specific version, use the following command:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```
**Output:**
```bash
deployment.apps/nginx-deployment rolled back to revision 2 🔄
```

#### View the History Again:
```bash
kubectl rollout history sample-deployment
```
**Output:**
```bash
deployment.apps/sample-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         Update to nginx 1.16.1
```

---

### **Step 6: Using the `--record` Option 📝**

You can also record changes using the `--record` option, which helps in tracking the commands that resulted in deployment changes.

#### Create a New Deployment Imperatively:
```bash
kubectl create deployment nginx-deployment --image=nginx:1.14.2
```
**Output:**
```bash
deployment.apps/nginx-deployment created ✅
```

#### Update the Image and Record Changes:
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
```
**Output:**
```bash
deployment.apps/nginx-deployment image updated ✅
```

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.17.1 --record
```
**Output:**
```bash
deployment.apps/nginx-deployment image updated ✅
```

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.18.0 --record
```
**Output:**
```bash
deployment.apps/nginx-deployment image updated ✅
```

#### View the Rollout History with Recorded Changes:
```bash
kubectl rollout history deployment/nginx-deployment
```
**Output:**
```bash
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
3         kubectl set image deployment/nginx-deployment nginx=nginx:1.17.1 --record
4         kubectl set image deployment/nginx-deployment nginx=nginx:1.18.0 --record
```

---

### **Step 7: Deployment Strategies 🔄**

Kubernetes provides different deployment strategies. For this example, we will demonstrate the `Recreate` strategy.

#### Delete All Existing Deployments and Pods 🗑️:
```bash
kubectl delete deployments,pods --all
```
**Output:**
```bash
deployment.apps "sample-deployment" deleted ✅
pod "sample-deployment-pod" deleted ✅
```

#### Modify the Deployment to Use the `Recreate` Strategy:
```yaml
# deployment.yaml (modified)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recreate-demo
spec:
  replicas: 3
  strategy:
    type: Recreate  # Recreate strategy
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2  # Initial version
        ports:
        - containerPort: 80
```

#### Apply the Manifest to the Cluster:
```bash
kubectl apply -f deployment.yaml
```
**Output:**
```bash
deployment.apps/recreate-demo created ✅
```

#### Verify the Pods are Running:
```bash
kubectl get deployments
kubectl get pods
```
**Output:**
```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
recreate-demo      3/3     3            3           1m

NAME                                READY   STATUS    RESTARTS   AGE
recreate-demo-xxxxxxx-xxxxx         1/1     Running   0          1m
recreate-demo-xxxxxxx-xxxxx         1/1     Running   0          1m
recreate-demo-xxxxxxx-xxxxx         1/1     Running   0          1m
```

#### Change the Deployment Image:
```bash
kubectl set image deployment/recreate-demo nginx=nginx:1.16.1
```
**Output:**
```bash
deployment.apps/recreate-demo image updated ✅
```

#### View the Deployment Status:
```bash
kubectl rollout status recreate-demo
```
**Output:**
```bash
deployment "recreate-demo" successfully rolled out ✅
```

#### Verify the Pods are Updated:
```bash
kubectl get pods
```
**Output:**
```bash
NAME                                READY   STATUS    RESTARTS   AGE
recreate-demo-xxxxxxx-xxxxx         1/1     Running   0          2m
recreate-demo-xxxxxxx-xxxxx         1/1     Running   0          2m
recreate-demo-xxxxxxx-xxxxx         1/1     Running   0          2m
```

#### Describe the Deployment:
```bash
kubectl describe deployment recreate-demo
```
**Output:**
```bash
Name:                   recreate-demo
Namespace:              default
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:          Recreate
RollingUpdateStrategy: maxUnavailable=1 maxSurge=1
Pod Template:
  Labels:               app
