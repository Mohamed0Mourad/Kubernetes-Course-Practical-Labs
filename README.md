

# 📦 Kubernetes Course – Practical Labs  
**Course Title**: [Kubernetes from Beginner to Master (Arabic)](https://www.udemy.com/course/kubernetes-from-beginner-to-master-arabic)  
**Instructor**: Eng. Ahmed Elfakharany  

---

## 📘 About This Repository

This repository contains hands-on **practical labs** and YAML manifests based on the excellent course:  
📚 **Kubernetes from Beginner to Master – in Arabic** by **Eng. Ahmed Elfakharany**, available on [Udemy](https://www.udemy.com/course/kubernetes-from-beginner-to-master-arabic).

These labs will help you reinforce your understanding of Kubernetes by applying the core concepts through real-world scenarios using Minikube, KinD, or any cloud-based Kubernetes cluster.

---

## 🎯 What You Will Learn

Throughout this course, you will:

✅ Master the fundamentals of Kubernetes architecture and components (Pods, Nodes, Controllers, API Server, etc.)  
✅ Deploy, manage, and scale containerized workloads using **Deployments**, **StatefulSets**, and **Jobs**  
✅ Understand service types like **ClusterIP**, **NodePort**, and how to expose apps to the internet  
✅ Configure and manage storage using **PersistentVolumes (PVs)** and **PersistentVolumeClaims (PVCs)**  
✅ Secure your applications and cluster access using **Role-Based Access Control (RBAC)**  
✅ Use **ConfigMaps** and **Secrets** to externalize and secure application configurations  
✅ Explore Stateful apps with **MySQL StatefulSet** deployment  
✅ Test access permissions with `kubectl auth can-i` and simulate real access scenarios  
✅ Clean up Kubernetes resources safely to maintain a tidy cluster

---

## 📁 Labs Included

| ✅ Lab | Description |
|-------|-------------|
| ConfigMaps | Externalize configuration and inject it into containers via env variables and mounted files |
| Secrets | Securely inject sensitive data like database passwords into Pods |
| RBAC | Demonstrate Role-Based Access Control by creating roles and testing user access |
| MySQL StatefulSet | Set up a MySQL cluster using StatefulSet and persistent storage |
| Services | Use ClusterIP and NodePort to expose applications |
| PVC & PV | Understand how to persist container data across Pod restarts |

---

## 🚀 Requirements

- Kubernetes cluster (Minikube, KinD, or cloud-managed like GKE, EKS, or AKS)  
- `kubectl` CLI installed and connected  
- Basic knowledge of YAML and Linux commands  
- [Course enrollment](https://www.udemy.com/course/kubernetes-from-beginner-to-master-arabic) for best results 🎓

---

```
