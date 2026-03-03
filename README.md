

---

# End-to-End DevOps GitOps POC on AWS EKS

## Project Overview

This Proof of Concept demonstrates a complete DevOps workflow using Infrastructure as Code, CI/CD, GitOps, and Kubernetes on AWS.

The goal of this project is to:

* Provision infrastructure using Terraform
* Build and push Docker images using GitHub Actions
* Deploy applications to EKS using GitOps (Argo CD)
* Automatically provision ALB using AWS Load Balancer Controller
* Validate automatic sync and zero-downtime updates

---

# Architecture Overview

1. EC2 instance used as DevOps machine
2. Terraform provisions:

   * VPC
   * Subnets
   * Internet Gateway
   * EKS Cluster
   * Managed Node Groups
3. GitHub Actions builds Docker image and pushes to DockerHub
4. GitHub Actions updates Kubernetes deployment image tag
5. Argo CD detects change and syncs cluster
6. AWS Load Balancer Controller provisions ALB via Ingress
7. Application becomes publicly accessible

---

# Tools Used and Explanation

---

## 1. Amazon EC2

Amazon EC2 is used as the DevOps control machine.

Purpose in this POC:

* Install Terraform
* Install kubectl
* Install eksctl
* Install Helm
* Connect to EKS cluster
* Install AWS Load Balancer Controller
* Install Argo CD

It acts as the administrative system to manage AWS and Kubernetes resources.

---

## 2. Terraform

Terraform is an Infrastructure as Code (IaC) tool.

Purpose:

* Automates infrastructure provisioning
* Creates VPC, subnets, route tables
* Creates EKS cluster
* Creates managed node groups

Why Terraform?

* Reproducible infrastructure
* Version-controlled infrastructure
* Eliminates manual AWS console work

You used Terraform to build:

* VPC
* EKS Cluster
* Node Groups
* Worker Nodes

Command used:

```
terraform init
terraform plan
terraform apply
```

After execution, full infrastructure was created automatically.

---

## 3. Amazon EKS (Elastic Kubernetes Service)

EKS is AWS’s managed Kubernetes service.

Purpose:

* Runs containerized applications
* Manages Kubernetes control plane
* Provides high availability

In this project:

* Terraform created EKS cluster
* Managed Node Groups were attached
* Worker nodes were launched automatically

---

## 4. Managed Node Groups

Managed Node Groups provide EC2 worker nodes for EKS.

Purpose:

* Run application pods
* Auto-scaling capability
* Automatic node lifecycle management

Why managed?

* AWS handles upgrades
* AWS handles scaling
* Reduced operational overhead

---

## 5. kubectl

kubectl is the Kubernetes CLI tool.

Purpose:

* Communicate with EKS cluster
* Deploy resources
* Check pods, services, deployments

Configured using:

```
aws eks update-kubeconfig --region us-east-1 --name <cluster-name>
```

After configuration:

```
kubectl get nodes
kubectl get pods
```

---

## 6. eksctl

eksctl is a CLI tool for managing EKS clusters.

Purpose in this POC:

* Associate IAM OIDC provider
* Create IAM Service Account for ALB controller

It simplifies EKS cluster-level operations.

---

## 7. Helm

Helm is a Kubernetes package manager.

Purpose:

* Install AWS Load Balancer Controller
* Install other Kubernetes applications easily

You used Helm to install:

* AWS Load Balancer Controller

---

## 8. AWS Load Balancer Controller

This controller enables Kubernetes Ingress to create AWS ALB automatically.

Without this:

* Ingress will not work
* No ALB will be created

What it does:

* Watches Kubernetes Ingress resources
* Automatically provisions ALB
* Configures listeners and target groups
* Connects ALB to Kubernetes services

Installed using:

* IAM OIDC association
* IAM policy
* Service account with IRSA
* Helm chart installation

---

## 9. Docker

Docker is used to containerize the application.

Purpose:

* Package application into container
* Ensure environment consistency
* Push image to DockerHub

Dockerfile defines:

* Base image
* Application artifact
* Entry point

---

## 10. DockerHub

DockerHub stores container images.

Purpose:

* Store versioned application images
* Allow Kubernetes to pull images

GitHub Actions pushes images to:

```
your-dockerhub-username/app-name:version
```

---

## 11. GitHub Actions (CI Pipeline)

GitHub Actions is used for Continuous Integration.

What it does:

1. Trigger on push to main branch
2. Build Docker image
3. Push image to DockerHub
4. Update image version in deployment.yaml
5. Commit updated manifest

This automates application build and version updates.

---

## 12. Kubernetes Manifests

You created:

* namespace.yaml
* deployment.yaml
* service.yaml
* ingress.yaml
* argocd application.yaml

Purpose of each:

Namespace:
Creates logical separation.

Deployment:
Defines pods and container image.

Service:
Exposes pods internally.

Ingress:
Exposes application externally via ALB.

Argo CD Application:
Connects GitHub repo to EKS cluster.

---

## 13. Argo CD (GitOps)

Argo CD is a GitOps continuous delivery tool.

Purpose:

* Monitor Git repository
* Detect manifest changes
* Automatically sync changes to cluster

How it works:

1. You push updated deployment.yaml
2. Argo CD detects change
3. Argo CD syncs cluster automatically
4. New pods are created
5. Old pods are terminated

This ensures:

* Desired state always matches Git repository
* Fully automated deployment

---

# Complete Workflow Summary

Step 1: EC2 instance created
Step 2: Terraform written for infrastructure
Step 3: GitHub Actions pipeline created
Step 4: Kubernetes manifests written
Step 5: Terraform applied → Infrastructure built
Step 6: AWS Load Balancer Controller installed
Step 7: Argo CD installed and UI accessed
Step 8: Application deployed using GitOps
Step 9: Code modified and pushed
Step 10: GitHub Actions updated image
Step 11: Argo CD auto-synced
Step 12: Application updated successfully

---

# What This POC Demonstrates

* Infrastructure as Code
* CI/CD Automation
* GitOps Model
* Kubernetes Ingress with ALB
* Zero-downtime deployment
* Fully automated DevOps pipeline

---

# Key DevOps Concepts Implemented

Infrastructure as Code
Continuous Integration
Continuous Delivery
GitOps
Containerization
Kubernetes Orchestration
Cloud-native deployment

---

# Final Outcome

You have successfully built:

A production-style DevOps pipeline on AWS using:

Terraform + EKS + GitHub Actions + Docker + Argo CD + ALB Controller

This architecture is similar to what real companies use in production environments.

---

