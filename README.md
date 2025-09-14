# talesofchina


---

# 🚀 Deploying Tales of China App on Amazon EKS with ALB Ingress

This guide explains how to deploy the **Tales of China** application on **Amazon EKS** and expose it publicly using an **AWS ALB Ingress**.

By the end, you will be able to access your app at a URL like:

```
http://k8s-talesofchina-ingress-123456.elb.amazonaws.com
```

---

## 🔹 Prerequisites

* AWS account with IAM permissions for EKS, EC2, VPC, IAM, and ALB.
* [kubectl](https://kubernetes.io/docs/tasks/tools/) installed.
* [eksctl](https://eksctl.io/) installed.
* [Docker](https://docs.docker.com/get-docker/) installed and configured.
* [AWS CLI](https://aws.amazon.com/cli/) configured with credentials.
* A Docker Hub account.

---

## 🔹 Step 1: Build & Push Docker Image

```bash
# Clone your project
git clone https://github.com/your-username/talesofchina.git
cd talesofchina

# Build Docker image
docker build -t talesofchina:latest .

# Tag image for Docker Hub
docker tag talesofchina:latest your-dockerhub-username/talesofchina:latest

# Push to Docker Hub
docker push your-dockerhub-username/talesofchina:latest
```

---

## 🔹 Step 2: Create EKS Cluster

```bash
eksctl create cluster --name talesofchina-cluster --region us-east-1 --nodes 2
```

This creates:

* VPC, subnets, security groups
* EKS control plane
* 2 worker nodes

---

## 🔹 Step 3: Deploy Kubernetes Manifests

### **Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: talesofchina-deployment
  namespace: talesofchina
spec:
  replicas: 2
  selector:
    matchLabels:
      app: talesofchina
  template:
    metadata:
      labels:
        app: talesofchina
    spec:
      containers:
      - name: talesofchina
        image: your-dockerhub-username/talesofchina:latest
        ports:
        - containerPort: 80
```

### **Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: talesofchina-service
  namespace: talesofchina
spec:
  type: NodePort
  selector:
    app: talesofchina
  ports:
  - port: 80
    targetPort: 80
```

### **Ingress**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: talesofchina-ingress
  namespace: talesofchina
  annotations:
    kubernetes.io/ingress.class: alb
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: talesofchina-service
            port:
              number: 80
```

---

## 🔹 Step 4: Apply the Manifests

```bash
# Create namespace
kubectl create namespace talesofchina

# Apply manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

---

## 🔹 Step 5: Get ALB DNS

```bash
kubectl get ingress -n talesofchina
```

Output example:

```
NAME                  CLASS    HOSTS   ADDRESS                                                            PORTS   AGE
talesofchina-ingress  <none>   *       k8s-talesofchina-ingress-123456.elb.amazonaws.com   80      2m
```

---

## 🔹 Step 6: Access the Application

Copy the ALB DNS (`ADDRESS` field) and open it in a browser:

```
http://k8s-talesofchina-ingress-123456.elb.amazonaws.com
```

✅ The application is now live and accessible publicly from anywhere.

---

## 🔹 Notes

* Ensure the ALB Security Group allows inbound traffic on **port 80 (HTTP)** and **443 (HTTPS)** if you add TLS.
* If you want a custom domain (e.g., `talesofchina.com`), use **Route53 + ACM Certificate Manager** with the same ALB.
* To delete resources:

  ```bash
  eksctl delete cluster --name talesofchina-cluster --region us-east-1
  ```

---


