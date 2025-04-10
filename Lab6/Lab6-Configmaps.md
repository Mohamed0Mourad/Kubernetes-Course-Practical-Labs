
### **Lab 1: Using Kubernetes ConfigMap to Externalize Application Configuration** 📂⚙️

#### **Objective:**
Create a **ConfigMap** to externalize the configuration of a web application and inject it into the container as both environment variables and a mounted file. This approach allows you to manage configuration separately from the application code, making it easier to update and manage settings in a Kubernetes environment.

#### **Prerequisites:**
- A running Kubernetes cluster (Minikube, KinD, or a cloud provider-managed cluster) 🌐.
- `kubectl` CLI installed and connected to the cluster 🖥️.

---

#### **Steps:**

##### **Step 1: Create the ConfigMap** 📝

First, create a **ConfigMap** that stores configuration values for a sample web application. These values include a database connection string, log level, and maximum connections.

Run the following command to create the ConfigMap:

```bash
kubectl create configmap app-config \
  --from-literal=DATABASE_URL="mysql://db:3306/mydb" \
  --from-literal=LOG_LEVEL="INFO" \
  --from-literal=MAX_CONNECTIONS="100"
```

You can verify the ConfigMap with the following command:

```bash
kubectl get configmap app-config -o yaml
```

This will show the details of the `app-config` ConfigMap. 📑

---

##### **Step 2: Create a Deployment to Use the ConfigMap** 🚢

Create a YAML file for a **Deployment** that uses the `app-config` ConfigMap. We will inject the ConfigMap data into the container as both environment variables and a mounted configuration file.

Create the following file called `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: configmap-demo
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
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DATABASE_URL
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
        - name: MAX_CONNECTIONS
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: MAX_CONNECTIONS
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

In this YAML:
- **Environment Variables**: The `DATABASE_URL`, `LOG_LEVEL`, and `MAX_CONNECTIONS` values are injected from the **ConfigMap** as environment variables.
- **Mounted File**: The ConfigMap data is mounted as a file inside the container at `/etc/config`. 📁

---

##### **Step 3: Deploy the Application** 🚀

Apply the `deployment.yaml` file to your cluster:

```bash
kubectl apply -f deployment.yaml
```

This will deploy the application with the **ConfigMap** values injected into the environment and as a file. 🎉

---

##### **Step 4: Verify the Environment Variables and Mounted File** 🔍

- **Check Environment Variables**: Access the running Pod and check the environment variables injected from the ConfigMap:

  ```bash
  kubectl get pods
  kubectl exec -it POD_NAME -- env
  ```

  You should see output like:

  ```bash
  # --- other variables
  DATABASE_URL=mysql://db:3306/mydb
  LOG_LEVEL=INFO
  MAX_CONNECTIONS=100
  # -- other variables
  ```

- **Check the Mounted File**: Verify that the ConfigMap data is mounted as a file inside the container:

  ```bash
  kubectl exec -it POD_NAME -- cat /etc/config/DATABASE_URL
  ```

  This should output:

  ```bash
  mysql://db:3306/mydb
  ```

---

##### **Step 5: Clean Up** 🧹

After completing the lab, clean up the resources by running:

```bash
kubectl delete deployment configmap-demo
kubectl delete configmap app-config
```

This ensures no leftover resources are consuming your cluster’s resources. 🔄

---

### **Summary:**
In this lab, you’ve learned how to use **Kubernetes ConfigMaps** to externalize application configuration and inject it both as environment variables and as a mounted file. This approach simplifies the management of configuration, especially in a cloud-native environment. 🌱
