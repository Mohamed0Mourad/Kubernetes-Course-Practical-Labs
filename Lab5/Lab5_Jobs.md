
### **Lab Setup: Kubernetes Job Patterns** 🚀

In this lab, we'll focus on two widely used Kubernetes job patterns: **Single-Completion Job** and **Parallel Job with Multiple Completions**. These patterns help simulate various real-world tasks like data migration and distributed data processing. 💡

---

### **Preparation** 🔧

1. **Create a directory for the lab:**
   ```bash
   mkdir jobs
   ```

---

### **Scenario 1: Single-Completion Job (Data Migration)** 📦➡️📁

#### **Use Case:**  
We simulate a **single-completion task** where a container pulls a data file, processes it, and stores it in another location. This represents a simple one-time data migration task.

#### **Job Manifest for Scenario 1:**

Create a file called `single-job.yaml` with the following content:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-migration-job
spec:
  completions: 1
  parallelism: 1
  template:
    spec:
      containers:
      - name: migration
        image: busybox
        command: ["sh", "-c", "echo 'Migrating data...' && sleep 10 && echo 'Data migration complete.'"]
      restartPolicy: Never
```

#### **Explanation:**
- **completions:** `1` ensures the job runs to completion once. ✅
- **parallelism:** `1` ensures only one pod runs at a time. 🧑‍💻
- The task uses a simple **echo** and **sleep** to simulate data migration. 📤

#### **Steps to Run:**

1. Apply the Job manifest:
   ```bash
   kubectl apply -f single-job.yaml
   ```

2. Check the Job status:
   ```bash
   kubectl get jobs
   ```

3. View the logs of the Job pod:
   ```bash
   kubectl get pods
   kubectl logs pod_name
   kubectl logs pod_name -f  # To follow the logs
   ```

   You should see output like:
   ```bash
   Migrating data...
   Data migration complete.
   ```

4. Verify that the Job is completed:
   ```bash
   kubectl get jobs
   ```

   The job should be marked as completed. ✅

---

### **Scenario 2: Parallel Job with Multiple Completions (Distributed Data Processing)** 🌐💻

#### **Use Case:**  
In this scenario, multiple pods process different parts of a dataset in parallel. It simulates a **distributed data processing job** where each pod processes a chunk of data, and the task is complete only when all chunks are processed.

#### **Job Manifest for Scenario 2:**

Create a file called `data-processing-job.yaml` with the following content:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processing-job
spec:
  completions: 5
  parallelism: 3
  template:
    spec:
      containers:
      - name: processor
        image: busybox
        command: ["sh", "-c", "echo 'Processing chunk $(MY_CHUNK)...' && sleep 20 && echo 'Chunk $(MY_CHUNK) processed.'"]
        env:
        - name: MY_CHUNK
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
      restartPolicy: Never
```

#### **Explanation:**
- **completions:** `5` ensures that the job is complete only after 5 pods have finished. ✔️
- **parallelism:** `3` ensures that 3 pods run simultaneously, processing different chunks of data. 🏃‍♂️🏃‍♀️
- **environment variable:** Each pod receives a unique name using `metadata.name`, simulating processing a different "chunk" of data. 📊

#### **Steps to Run:**

1. Apply the Job manifest:
   ```bash
   kubectl apply -f data-processing-job.yaml
   ```

2. View the logs of each Job pod (repeat this for all running/complete pods):
   ```bash
   kubectl get pods  # This will show the pods' status
   kubectl logs pod_name  # View logs for a specific pod
   ```

   You should see output like:
   ```bash
   Processing chunk data-processing-job-xyz...
   Chunk data-processing-job-xyz processed.
   ```

3. Once all pods have completed, verify the final status:
   ```bash
   kubectl get jobs
   ```

   You should see the job marked as completed once all 5 chunks are processed. ✅

---

### **Cleanup** 🧹

After completing the lab, clean up the resources by deleting the Jobs:

```bash
kubectl delete job data-migration-job
kubectl delete job data-processing-job
```

This will ensure that no resources are left running in the cluster. 🗑️

---

By following these steps, you'll have a practical understanding of how to set up and manage **single-completion** and **parallel** Kubernetes jobs, both of which are essential for tasks like data migration and distributed data processing. 🌟
