
### **Lab 2: Managing Sensitive Data Using Kubernetes Secrets** 🔒💻

#### **Objective:**  
Create and use a **Kubernetes Secret** to securely store a **database password** and inject it into a containerized application. Secrets help store sensitive information like passwords, tokens, and certificates securely in Kubernetes. 🔑

#### **Prerequisites:**
- A running Kubernetes cluster (Minikube, KinD, or a cloud provider-managed cluster) 🌐.
- `kubectl` CLI installed and connected to the cluster 🖥️.

---

#### **Steps:**

##### **Step 1: Create the Secret** 🛡️

First, create a **Secret** to store a sensitive password for a database. You can use `kubectl` to create the Secret and store it in an encoded format:

```bash
kubectl create secret generic db-secret --from-literal=DB_PASSWORD="supersecretpassword"
```

You can verify the **Secret** with the following command:

```bash
kubectl get secret db-secret -o yaml
```

> Note: The password will be **base64-encoded** in the output, which ensures it is stored securely. ⚙️

---

##### **Step 2: Create a Deployment to Use the Secret** 🚢🔐

Now, modify the existing deployment to inject the **Secret** into the container as an environment variable. This helps to securely pass the database password to the application.

Create or modify the `deployment.yaml` file as follows:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secret-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-container
        image: nginx
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
```

In this configuration:
- **Secret Reference**: The **DB_PASSWORD** value is injected into the container from the **db-secret** Secret.
- The **password** remains securely stored in the Secret, and **Kubernetes** ensures it is passed to the application as an environment variable. 🔐

---

##### **Step 3: Deploy the Application** 🚀

Apply the `deployment.yaml` file to your Kubernetes cluster:

```bash
kubectl apply -f deployment.yaml
```

This will deploy the application while securely injecting the database password into the container as an environment variable. 🎉

---

##### **Step 4: Verify the Secret in the Pod** 🔍

- **Access the Running Pod**: First, get the Pod name and verify that the secret was injected correctly as an environment variable.

```bash
kubectl get pods
kubectl exec -it POD_NAME -- env
```

You should see output like:

```bash
DB_PASSWORD=supersecretpassword
```

- **Check that the Secret is not Exposed in Plain Text**:  
  When retrieving the Pod’s definition using `kubectl`, the **actual secret value** is **not exposed**. It will only show the reference to the **Secret** in the YAML.

```bash
kubectl get pod $POD_NAME -o yaml
```

The output will show something like this:

```yaml
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      key: DB_PASSWORD
      name: db-secret
```

This ensures that sensitive information is **not exposed** in plain text. 🔒

---

##### **Step 5: Clean Up** 🧹

After completing the lab, clean up the resources by running:

```bash
kubectl delete deployment secret-demo
kubectl delete secret db-secret
```

This will ensure no leftover resources are consuming your cluster’s resources. 🔄

---

### **Summary:**
In this lab, you’ve learned how to use **Kubernetes Secrets** to securely manage sensitive data, such as database passwords. By injecting secrets into containers as environment variables, Kubernetes helps keep sensitive information safe and ensures that it is not exposed in plain text. 🛡️
