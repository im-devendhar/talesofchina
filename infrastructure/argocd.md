############################
# Providers for EKS cluster
############################

data "aws_eks_cluster" "main" {
  name = var.eks_cluster_name
}

data "aws_eks_cluster_auth" "main" {
  name = var.eks_cluster_name
}

provider "kubernetes" {
  host                   = data.aws_eks_cluster.main.endpoint
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.main.certificate_authority[0].data)
  token                  = data.aws_eks_cluster_auth.main.token
}

provider "helm" {
  kubernetes {
    host                   = data.aws_eks_cluster.main.endpoint
    cluster_ca_certificate = base64decode(data.aws_eks_cluster.main.certificate_authority[0].data)
    token                  = data.aws_eks_cluster_auth.main.token
  }
}

############################
# Namespace
############################

resource "kubernetes_namespace" "argocd" {
  metadata {
    name = var.argocd_namespace
    labels = {
      "app.kubernetes.io/name" = "argocd"
    }
  }
}

############################
# Argo CD via Helm chart
############################

resource "helm_release" "argocd" {
  name       = "argocd"
  namespace  = kubernetes_namespace.argocd.metadata[0].name
  repository = "https://argoproj.github.io/argo-helm"
  chart      = "argo-cd"

  # Pin a version tested for your setup; update intentionally via PRs.
  # You can look up versions here: https://artifacthub.io/packages/helm/argo/argo-cd
  version = var.argocd_chart_version

  # Merge default and env-specific overrides
  values = [
    file("${path.module}/values/argocd-values.yaml")
  ]

  # Ensure namespace exists before installing helm chart
  depends_on = [kubernetes_namespace.argocd]
}

############################
# (Optional) Outputs
############################

output "argocd_namespace" {
  value       = kubernetes_namespace.argocd.metadata[0].name
  description = "Namespace where Argo CD is installed"
}

# If service type is LoadBalancer, this will show the DNS after it’s allocated.
output "argocd_server_service_hostname" {
  value       = try(
    kubernetes_service.argocd_server.status[0].load_balancer[0].ingress[0].hostname,
    "set service.type=LoadBalancer in values to get an external hostname"
  )
  description = "External hostname of Argo CD server if using LoadBalancer"
}

############################
# (Optional) Data source for LB hostname
# Only if you want to read it via Kubernetes provider
############################

data "kubernetes_service" "argocd_server" {
  metadata {
    name      = "argocd-server"
    namespace = kubernetes_namespace.argocd.metadata[0].name
  }

  depends_on = [helm_release.argocd]
}

# Re-expose through output using data source above
output "argocd_server_hostname_via_data" {
  value       = try(data.kubernetes_service.argocd_server.status[0].load_balancer[0].ingress[0].hostname, null)
  description = "External hostname of Argo CD server (via data.kubernetes_service)"
}