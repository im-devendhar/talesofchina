

## **Argo CD Application Setup**

***
# Install Argo CD

```bash
kubectl delete namespace argocd --ignore-not-found
kubectl create namespace argocd

kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl rollout status deployment/argocd-server \
  -n argocd --timeout=300s
```
# Expose Argo CD UI Using LoadBalancer
```bash

kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "LoadBalancer"}}'
```
# Get Argo CD UI URL:
```bash
kubectl get svc -n argocd argocd-server
```
# Get Initial Argo CD Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 --decode
```

# Login at

http://<ARGOCD-LOADBALANCER-DNS>
Username: admin
Password: <decoded-password>

# Create and Apply Argo CD Application
```bash
kubectl apply -f argocd/application.yml -n argocd
```
Check status:
```bash
kubectl -n argocd get applications
kubectl -n argocd describe application talesofchina
```
The Argo CD Application manifest (`argocd/application.yml`) defines:

*   Which Git repository Argo CD should watch
*   Which directory contains Kubernetes manifests
*   Which namespace to deploy into
*   Synchronization rules (auto-sync, prune, self-heal)
*   Whether the namespace should be created automatically


This creates a Kubernetes resource:

    kind: Application
    metadata:
      name: talesofchina
      namespace: argocd

Once created, Argo CD will:

### ✔️ Continuously watch your GitHub repo

### ✔️ Automatically sync changes

### ✔️ Create the target namespace (`talesofchina`)

### ✔️ Deploy all manifests from `k8s-manifests/`

### ✔️ Delete resources that are no longer in Git (prune)

### ✔️ Correct manual changes in the cluster (self-heal)

You **do NOT** need to apply `application.yml` again.  
You only apply it once so Argo CD knows where your application lives.

***

***


