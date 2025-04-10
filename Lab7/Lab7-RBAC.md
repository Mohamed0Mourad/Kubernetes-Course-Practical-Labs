### **Lab: Demonstrating Kubernetes Role-Based Access Control (RBAC)** 🔐🛠️

In this lab, you'll explore **Kubernetes Role-Based Access Control (RBAC)**, a system that allows you to define and control access to various Kubernetes resources based on roles assigned to users, service accounts, or groups. By the end of this lab, you'll be able to create roles, bind them to users or service accounts, and test the access permissions.

#### **Prerequisites:**
- A running Kubernetes cluster (Minikube, KinD, or a managed Kubernetes service). 🌐
- `kubectl` installed and configured to interact with your Kubernetes cluster. 🖥️

---

### **Steps:**

#### **Step 1: Set Up a Namespace for the Lab** 🏷️

Namespaces help organize resources within a cluster and isolate environments. To keep everything isolated for this lab, create a new namespace:

```bash
kubectl create namespace rbac-lab
```

This namespace will contain all the resources related to this lab. ✨

---

#### **Step 2: Create a Service Account for Testing** 👤

In Kubernetes, a **ServiceAccount** is used to simulate a user or application accessing the Kubernetes API. For this lab, we'll create a **dev-user** service account within the `rbac-lab` namespace:

```bash
kubectl create serviceaccount dev-user --namespace rbac-lab
```

To view the service account:

```bash
kubectl get serviceaccount dev-user --namespace rbac-lab
```

---

#### **Step 3: Create a Role with Limited Permissions** 🎭

Roles define the permissions that a user (or service account) has for specific resources. In this step, we'll create a **Role** that allows the `dev-user` to view Pods (but not create or delete them) within the `rbac-lab` namespace.

Create a YAML file, `view-pods-role.yaml`, with the following content:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: rbac-lab
  name: view-pods
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

Apply the role to the cluster:

```bash
kubectl apply -f view-pods-role.yaml
```

To verify the role was created:

```bash
kubectl get role view-pods --namespace rbac-lab
```

---

#### **Step 4: Bind the Role to the Service Account** 🔗

To allow the `dev-user` service account to use the `view-pods` Role, create a **RoleBinding**. This binding associates the role with the service account.

Create a YAML file, `view-pods-rolebinding.yaml`, with the following content:

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: rbac-lab
  name: view-pods-binding
subjects:
  - kind: ServiceAccount
    name: dev-user
    namespace: rbac-lab
roleRef:
  kind: Role
  name: view-pods
  apiGroup: rbac.authorization.k8s.io
```

Apply the RoleBinding:

```bash
kubectl apply -f view-pods-rolebinding.yaml
```

---

#### **Step 5: Test Access with the `can-i` Command** 🔍

Kubernetes provides a useful command, `kubectl auth can-i`, to check if a user or service account has permission to perform a specific action.

Check if `dev-user` can list Pods:

```bash
kubectl auth can-i list pods --namespace rbac-lab --as=system:serviceaccount:rbac-lab:dev-user
```

You should see a response of **yes**, since `dev-user` has permission to list Pods.

Check if `dev-user` can create Pods:

```bash
kubectl auth can-i create pods --namespace rbac-lab --as=system:serviceaccount:rbac-lab:dev-user
```

You should see **no**, as the current role does not allow create permissions.

---

#### **Step 6: Add Additional Permissions** ➕

Let's modify the Role to allow `dev-user` to create and delete Pods. Update the role by adding the `create` and `delete` verbs.

Modify the `view-pods-role.yaml` file as follows:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: rbac-lab
  name: view-pods
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch", "create", "delete"]
```

Apply the changes:

```bash
kubectl apply -f view-pods-role.yaml
```

Now, test if `dev-user` can create Pods:

```bash
kubectl auth can-i create pods --namespace rbac-lab --as=system:serviceaccount:rbac-lab:dev-user
```

The response should now be **yes**, as the user has permission to create Pods.

---

#### **Step 7: Create a Pod as `dev-user`** ⚙️

Now that `dev-user` has permission to create Pods, let's try creating one. Create a simple Pod manifest file, `pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: rbac-lab
spec:
  containers:
  - name: nginx
    image: nginx
```

Try creating the Pod as `dev-user`:

```bash
kubectl create -f pod.yaml --as=system:serviceaccount:rbac-lab:dev-user
```

Verify that the Pod was created:

```bash
kubectl get pods --namespace rbac-lab
```

---

#### **Step 8: Clean Up** 🧹

To avoid cluttering your cluster, delete the resources you created:

```bash
kubectl delete namespace rbac-lab
```


### **Difference Between `kubectl apply` and `kubectl create`** 🆚

- **`kubectl create`:**  
  This command is used to create a resource from scratch. If the resource already exists, the command will fail. It's ideal for creating new resources, like namespaces or service accounts. ⚙️

- **`kubectl apply`:**  
  This command is used to create or update a resource. If the resource doesn’t exist, it will be created; if it already exists, it will be updated based on the YAML configuration. This is useful when you want to make incremental changes to a resource. 📝

In most production environments, **`kubectl apply`** is preferred, as it ensures that resources are maintained and updated according to their YAML definitions.
