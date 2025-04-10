
## 🧪 **Lab Setup: MySQL StatefulSet in KinD**

In this lab, we’ll set up a **highly available MySQL cluster** using a **StatefulSet** in a **KinD (Kubernetes in Docker)** environment. This involves creating persistent volumes, a headless service for stable DNS, and initializing a database using a ConfigMap.

---

### 🛠️ **Step 3: Create a Headless Service**

A **headless service** gives each pod in the StatefulSet a **unique network identity**, which is essential for MySQL clustering or replication.

📄 **`mysql-headless-service.yaml`**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  ports:
    - port: 3306
  clusterIP: None  # 👈 This makes it headless!
  selector:
    app: mysql
```

🚀 **Apply the service**:

```bash
kubectl apply -f mysql-headless-service.yaml
```

✅ *Now, each MySQL pod can be accessed by a DNS name like `mysql-0.mysql`, `mysql-1.mysql`, etc.*

---

### 📦 **Step 4: Create a MySQL ConfigMap**

We'll create a **ConfigMap** to initialize the database with a SQL script that:
- Creates a new database `mydb` 🗃️
- Adds a user `user` with password `password` 🔑
- Grants that user full privileges on `mydb`

📄 **`mysql-configmap.yaml`**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  init.sql: |
    CREATE DATABASE IF NOT EXISTS mydb;
    CREATE USER 'user'@'%' IDENTIFIED BY 'password';
    GRANT ALL PRIVILEGES ON mydb.* TO 'user'@'%';
    FLUSH PRIVILEGES;
```

🚀 **Apply the ConfigMap**:

```bash
kubectl apply -f mysql-configmap.yaml
```

📌 *This will be mounted into each pod to automatically initialize the database at startup.*

---

### 🧱 **Step 5: Create a MySQL StatefulSet**

This is the core of our setup. We’ll run **3 replicas** of MySQL, each with:
- Its own persistent volume (PVC)
- Initialization from the ConfigMap
- A root password via environment variable

📄 **`mysql-statefulset.yaml`**:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"  # 👈 refers to the headless service
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rootpassword"
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
        - name: mysql-init
          mountPath: /docker-entrypoint-initdb.d
      volumes:
      - name: mysql-init
        configMap:
          name: mysql-config
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
      storageClassName: standard
```

🚀 **Apply the StatefulSet**:

```bash
kubectl apply -f mysql-statefulset.yaml
```

📌 *Each pod (mysql-0, mysql-1, mysql-2) will have its own volume like `mysql-data-mysql-0`.*

---

### 📊 **Step 6: Check the StatefulSet and Pods**

Monitor the pod creation progress:

```bash
kubectl get pods
```

Repeat the command until you see all 3 pods in **Running** state.

🗃️ **Check the PVCs**:

```bash
kubectl get pvc
```

Expected output:

```
NAME                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-data-mysql-0   Bound    pvc-xxxx...                                5Gi        RWO            standard        5m
mysql-data-mysql-1   Bound    pvc-yyyy...                                5Gi        RWO            standard        3m
mysql-data-mysql-2   Bound    pvc-zzzz...                                5Gi        RWO            standard        1m
```

---

### 🧩 **Step 7: Accessing MySQL**

Login to one of the pods:

```bash
kubectl exec -it mysql-0 -- mysql -u root -p
```

🔑 Use password: `rootpassword`

Inside MySQL:

```sql
SHOW DATABASES;
USE mydb;
SHOW TABLES;
```

✅ You should see that the `mydb` database exists. Now let’s test logging in with the created user:

```bash
kubectl exec -it mysql-0 -- mysql -u user -ppassword
```

Inside MySQL as `user`, create a table:

```sql
USE mydb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

SELECT * FROM users;
```

🎉 You now have a working MySQL cluster with persistent storage and pre-initialized data!

