
---

# Optional: Monitoring Integration (Prometheus + Grafana)

## Overview

Monitoring is a critical component in production-grade Kubernetes environments.

In this POC, monitoring can be optionally integrated using:

* Prometheus (Metrics Collection)
* Grafana (Visualization & Dashboards)

This provides:

* Application-level metrics
* Kubernetes cluster metrics
* Node-level metrics
* Resource usage insights
* Alerting capability (optional)

---

# 1. Prometheus

## What is Prometheus?

Prometheus is an open-source monitoring and alerting toolkit designed for Kubernetes environments.

It:

* Scrapes metrics from applications and Kubernetes components
* Stores time-series data
* Supports powerful querying using PromQL
* Integrates with Alertmanager for alerts

In Kubernetes, Prometheus collects:

* Pod CPU and memory usage
* Node resource utilization
* API server metrics
* Deployment health
* Custom application metrics (if exposed)

---

## Installing Prometheus (Using Helm)

The recommended approach is to use the kube-prometheus-stack Helm chart.

Add Helm repository:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Install kube-prometheus-stack:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

Verify installation:

```bash
kubectl get pods -n monitoring
```

This installs:

* Prometheus
* Grafana
* Node Exporter
* kube-state-metrics
* Alertmanager

---

# 2. Grafana

## What is Grafana?

Grafana is a visualization tool used to display monitoring data collected by Prometheus.

It provides:

* Real-time dashboards
* Kubernetes cluster dashboards
* Node-level monitoring
* Application performance visualization

Grafana connects directly to Prometheus as a data source.

---

## Accessing Grafana

Port-forward Grafana service:

```bash
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
```

Access in browser:

```
http://localhost:3000
```

Default credentials:

Username:

```
admin
```

Password:

```bash
kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
```

---

# What You Can Monitor

After installation, pre-configured dashboards include:

* Kubernetes Cluster Overview
* Node Resource Usage
* Pod Resource Consumption
* Deployment Health
* API Server Metrics

You can also:

* Create custom dashboards
* Set alert rules
* Monitor specific namespaces
* Track CI/CD deployments impact

---

# Optional: Exposing Grafana via Ingress (ALB)

If required, Grafana can be exposed publicly using an Ingress resource and AWS Load Balancer Controller.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
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
                name: monitoring-grafana
                port:
                  number: 80
```

This will provision a new ALB automatically.

---

# Why Monitoring is Important in This POC

Without monitoring:

* You cannot see pod crashes easily
* You cannot detect high CPU usage
* You cannot track memory leaks
* You cannot identify scaling needs

With Prometheus + Grafana:

* You gain observability into the cluster
* You can troubleshoot faster
* You can validate deployment stability
* You can prepare this POC for production readiness

---

# DevOps Maturity Upgrade

Adding monitoring upgrades this project from:

Basic CI/CD + GitOps

to

Production-Ready DevOps Architecture

with observability, metrics, and operational visibility.

---

# Final Architecture After Monitoring Integration

EC2
→ Terraform provisions infrastructure
→ EKS runs Kubernetes
→ GitHub Actions builds images
→ Argo CD deploys application
→ ALB exposes application
→ Prometheus collects metrics
→ Grafana visualizes system health

---


