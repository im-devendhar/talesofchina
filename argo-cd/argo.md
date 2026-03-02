

## **Argo CD Application Setup**

***

# 📌 Argo CD Application (One-Time Setup)

The Argo CD Application manifest (`argocd/application.yml`) defines:

*   Which Git repository Argo CD should watch
*   Which directory contains Kubernetes manifests
*   Which namespace to deploy into
*   Synchronization rules (auto-sync, prune, self-heal)
*   Whether the namespace should be created automatically

Argo CD **does not automatically read this file from the Git repository**.  
It must be **applied once** to the Kubernetes cluster.

### **📍 One-time command to create the Argo CD Application**

```bash
kubectl apply -f argocd/application.yml -n argocd
```

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

# Why this is done only once

The Argo CD Application is a Kubernetes object.  
Once created, Argo CD stores it inside the cluster and uses it continuously.

Applying it again on every pipeline run is:

*   Not required
*   Not recommended
*   Can cause unnecessary overrides

***

# Optional (if using GitHub Actions)

If you prefer to apply it from GitHub Actions instead of local machine, you can temporarily add:

```yaml
- name: Create Argo CD application
  run: |
    kubectl apply -f argocd/application.yml -n argocd
```

Run the pipeline once → Argo CD Application gets created.  
Then remove this step.

***


