
# 🔐 Kubernetes Ingress Lab: TLS Termination

![Kubernetes](https://img.shields.io/badge/Kubernetes-Ingress-blue)
![Nginx](https://img.shields.io/badge/Controller-NGINX-orange)
![TLS](https://img.shields.io/badge/TLS-Termination-purple)

This lab demonstrates how to enable TLS termination on the Ingress side using a self-signed certificate in a Kubernetes cluster with the Nginx Ingress Controller.

---

## 🛠️ Prerequisites

- Kind (Kubernetes in Docker) installed  
- `kubectl` configured  
- `openssl` installed  

---

## 🔧 Step 1: Generate a Self-Signed Certificate

We'll use the `openssl` command to create a self-signed certificate and private key. This will generate two files: `tls.crt` and `tls.key`.

```bash
# Create a self-signed certificate and key with OpenSSL
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout tls.key -out tls.crt -subj "/CN=example.com/O=example.com"
```

### 🔍 Explanation:

- `-x509`: Specifies that the certificate should be self-signed  
- `-nodes`: Prevents OpenSSL from encrypting the private key  
- `-days 365`: Valid for 365 days  
- `-newkey rsa:2048`: Creates a new 2048-bit RSA key  
- `-keyout tls.key`: Output file for the private key  
- `-out tls.crt`: Output file for the certificate  
- `-subj "/CN=example.com/O=example.com"`: Sets the CN and O fields  

This will create two files in your current directory:

- `tls.crt`: The certificate file  
- `tls.key`: The private key file  

---

## 🔐 Step 2: Create a Kubernetes Secret

Create a Kubernetes Secret to store the certificate and key.

```bash
kubectl create secret tls example-tls-secret --cert=tls.crt --key=tls.key
```

### 🔍 Explanation:

- `tls`: The type of the secret  
- `example-tls-secret`: The name of the secret  
- `--cert`: Path to the certificate file  
- `--key`: Path to the private key file  

---

## 📝 Step 3: Update the Ingress Object to Use TLS

We'll modify the existing Ingress object to enable TLS termination.

### ✍️ Updated Ingress Configuration (`example-ingress.yaml`)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: nginx

  tls:
  - hosts:
    - www.example.com
    - api.example.com
    secretName: example-tls-secret

  rules:
  - host: www.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: www
            port:
              number: 80
      - path: /admin
        pathType: Exact
        backend:
          service:
            name: admin
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 80
```

### 🔍 Explanation:

- `tls`: Enables TLS for the specified hosts  
- `hosts`: List of domains using TLS  
- `secretName`: References the secret created in Step 2  

---

## 🚀 Step 4: Apply the Updated Ingress Configuration

```bash
kubectl apply -f example-ingress.yaml
```

---

## 🔁 Optional: HTTP to HTTPS Redirection

To force all HTTP traffic to redirect to HTTPS, add the following annotation:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

