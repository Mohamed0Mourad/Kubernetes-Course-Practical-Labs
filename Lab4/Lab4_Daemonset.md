

### Step 1: Create a Multi-Node KinD Cluster 🚀

We need to ensure that our KinD cluster has more than one node, so let's delete any existing cluster and create a new one with multiple nodes.

#### Delete the existing KinD cluster (if any) 🗑️:
```bash
kind delete cluster
```
**Output:**
```
Deleting cluster "kind" ...
```

#### Create the new multi-node cluster YAML file 📄:
```yaml
# cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```
Explanation:  
- The above YAML file creates a KinD cluster with one control-plane node and two worker nodes.

#### Create the KinD cluster using the configuration file 🛠️:
```bash
kind create cluster --config=cluster.yaml
```
**Output:**
```
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.23.0) 🖼
 ✓ Preparing nodes 🛠
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🛸
 ✓ Installing kubelet & kube-proxy 🔧
 ✓ Setting up kubeconfig 🗂
Cluster creation complete. You can now use your cluster with kubectl.
```

#### Confirm that all nodes are available and ready ✅:
```bash
kubectl get nodes
```
**Output:**
```
NAME                STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   1m    v1.23.0
kind-worker          Ready    <none>          1m    v1.23.0
kind-worker2         Ready    <none>          1m    v1.23.0
```
The control-plane node and both worker nodes should be in `Ready` status.

---

### Step 2: Create and Apply a DaemonSet 🔄

A DaemonSet ensures that a copy of a pod runs on all selected nodes. Let’s create and apply a DaemonSet.

#### Create the `daemonset.yaml` file 📄:
```yaml
# daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.17.1-1.0
```
Explanation:  
- This DaemonSet will deploy a pod running Fluentd on all nodes of the cluster.

#### Apply the DaemonSet to the cluster 🌐:
```bash
kubectl apply -f daemonset.yaml
```
**Output:**
```
daemonset.apps/fluentd created
```

#### Verify the pods created by the DaemonSet 🔍:
```bash
kubectl get pods -o wide
```
**Output:**
```
NAME                         READY   STATUS    RESTARTS   AGE   IP              NODE             NOMINATED NODE   READINESS GATES
fluentd-xxxxx-xxxxx           1/1     Running   0          10s   192.168.0.2     kind-worker      <none>           <none>
fluentd-xxxxx-xxxxx           1/1     Running   0          10s   192.168.0.3     kind-worker2     <none>           <none>
fluentd-xxxxx-xxxxx           1/1     Running   0          10s   192.168.0.4     kind-control-plane <none>           <none>
```

---

### Step 3: Limit DaemonSet to Specific Nodes 🎯

Now, let's limit the DaemonSet to only run on nodes with a specific label.

#### Label one of the worker nodes 🏷️:
```bash
kubectl label nodes kind-worker gpu=true
```
**Output:**
```
node/kind-worker labeled
```

#### Modify the `daemonset.yaml` to target only nodes with the `gpu=true` label 🔄:
```yaml
# daemonset.yaml (modified)
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      nodeSelector:
        gpu: "true"
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.17.1-1.0
```
Explanation:  
- The `nodeSelector` is added under `spec` to limit pod scheduling to nodes with the `gpu=true` label.

#### Apply the modified DaemonSet 📤:
```bash
kubectl apply -f daemonset.yaml
```
**Output:**
```
daemonset.apps/fluentd configured
```

#### Verify the pod is now scheduled on the designated node only 🔎:
```bash
kubectl get pods -o wide
```
**Output:**
```
NAME                         READY   STATUS    RESTARTS   AGE   IP              NODE             NOMINATED NODE   READINESS GATES
fluentd-xxxxx-xxxxx           1/1     Running   0          10s   192.168.0.2     kind-worker      <none>           <none>
```
The Fluentd pod should now only be running on `kind-worker`, which has the `gpu=true` label.

---

### Summary 📚

In this lab, we:
1. Created a KinD cluster with multiple nodes 🌐.
2. Deployed a DaemonSet to ensure pods run on each node 🔄.
3. Applied a label to a worker node and modified the DaemonSet to target only nodes with that label 🎯.

This is useful for running DaemonSets with specific requirements (e.g., GPUs, custom resources) on designated nodes within a cluster.
