 
This explains both:

*   **kubectl installation (one‑time)**
*   **AWS Load Balancer Controller manual installation (one‑time)**

***

#  **Installing kubectl and AWS Load Balancer Controller (One-Time Setup)**

Before deploying applications using Argo CD + Ingress on EKS, the following components must be installed **once** on your system and cluster:

*   `kubectl` (to interact with the cluster)
*   AWS Load Balancer Controller (required for ALB-based Ingress)

These steps are executed **one time only**, after EKS and node groups are created.

***

## **1. Install kubectl (One-Time)**

Install kubectl on your local machine / CloudShell:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl apt-transport-https

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubectl

kubectl version --client

```


eksctl is required for:

Associating IAM OIDC provider
Creating service accounts for AWS Load Balancer Controller
Managing cluster-level resources
```bash
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz"
tar -xzf eksctl_$(uname -s)_amd64.tar.gz
sudo mv eksctl /usr/local/bin/
eksctl version
```
Configure kubectl to connect to the EKS cluster:
```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name <your-eks-cluster-name>
```

This allows your system to communicate with the EKS cluster.

***

## **2. Install AWS Load Balancer Controller (One-Time)**

The AWS Load Balancer Controller is required for managing Ingress objects and creating ALBs automatically.

### **Step 1: Associate IAM OIDC Provider**

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster eks-cluster \
  --region us-east-1 \
  --approve
```

### **Step 2: Download IAM Policy**

```bash
curl -o iam-policy.json \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

### **Step 3: Create IAM Policy**

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam-policy.json
```

Copy the **Policy ARN** from the output.

### **Step 4: Create ServiceAccount with IRSA**

```bash
eksctl create iamserviceaccount \
  --cluster eks-cluster \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::943818143857:policy/AWSLoadBalancerControllerIAMPolicy \
  --region us-east-1 \
  --override-existing-serviceaccounts \
  --approve
```

### **Step 5: Install CRDs**

```bash
kubectl apply -f https://raw.githubusercontent.com/aws/eks-charts/master/stable/aws-load-balancer-controller/crds/crds.yaml
```

### **Step 6: Install AWS Load Balancer Controller using Helm**

Install helm
```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

```bash
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=eks-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=$(aws eks describe-cluster \
      --name eks-cluster \
      --query "cluster.resourcesVpcConfig.vpcId" \
      --output text)
```

### **Step 7: Install Argo Cd**
```bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    kubectl rollout status deployment/argocd-server -n argocd --timeout=300s
```
### **Step 7: Verify Installation**

```bash
kubectl -n kube-system get deployment aws-load-balancer-controller
```

Expected:

    aws-load-balancer-controller   1/1   Running

***

## **Why This Must Be Done Before GitHub Actions**

GitHub Actions deploys your application and Ingress via Argo CD—but ALB creation requires the Load Balancer Controller to be already installed in the cluster.

Without this controller:

*   Ingress will not work
*   ALB will not be created
*   Argo CD sync will show errors
*   Application won’t be accessible

This one-time setup ensures your cluster is ready for GitOps deployments.

***

## Summary

| Component      | Installed Where?           | When?      | Required For             |
| -------------- | -------------------------- | ---------- | ------------------------ |
| kubectl        | Local machine / CloudShell | One-time   | Cluster access           |
| ALB Controller | EKS Cluster                | One-time   | Ingress + ALB creation   |
| GitHub Actions | CI pipeline                | Every push | Build + Sync via Argo CD |

After completing the above steps, your cluster is fully ready for:

*   Argo CD auto-sync
*   Ingress creation
*   ALB provisioning
*   GitHub Actions GitOps pipeline

***









