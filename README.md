# ⚙️ Kubernetes EKS GitOps

> Production-grade EKS cluster with ArgoCD GitOps, Helm-based deployments, Prometheus/Grafana monitoring, RBAC namespace isolation, and GitHub Actions CI/CD pipeline. Handles 1000+ requests/min with zero-downtime rolling deployments.

[![Kubernetes](https://img.shields.io/badge/Kubernetes-EKS-326CE5?style=flat-square&logo=kubernetes)](https://kubernetes.io)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?style=flat-square&logo=argo)](https://argoproj.github.io/cd)
[![Helm](https://img.shields.io/badge/Helm-Charts-0F1689?style=flat-square&logo=helm)](https://helm.sh)
[![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC?style=flat-square&logo=terraform)](https://terraform.io)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## 🏗️ Architecture

```
GitHub Actions CI ─────────────────────────────────────────────┐
  (build → test → push to ECR → update Helm values)            │
                                                               │
ArgoCD (GitOps) ─── watches this repo ──────────────────────── ┤
  (detects drift → syncs EKS to desired state)                 │
                                                               ▼
┌─────────────────────────────────────────────────────────┐
│                 AWS EKS Cluster                         │
│                                                         │
│  Namespace: production                                  │
│  ┌────────────────────────────────────────┐             │
│  │  Deployment (3 replicas, rolling)      │             │
│  │  HPA: CPU>70% → scale to max 10        │             │
│  │  Service → ALB Ingress                 │             │
│  └────────────────────────────────────────┘             │
│                                                         │
│  Namespace: monitoring                                  │
│  ┌────────────────────────────────────────┐             │
│  │  kube-prometheus-stack (Helm)          │             │
│  │  Grafana dashboards                    │             │
│  │  AlertManager → Slack                  │             │
│  └────────────────────────────────────────┘             │
└─────────────────────────────────────────────────────────┘
```

## 📁 Repository Structure

```
kubernetes-eks-gitops/
├── terraform/
│   ├── eks-cluster/       # EKS cluster + node groups
│   ├── vpc/               # VPC for EKS
│   └── ecr/               # ECR repository
├── helm/
│   ├── app/               # Application Helm chart
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/     # Deployment, Service, Ingress, HPA
│   └── argocd-apps/       # ArgoCD Application manifests
├── argocd/
│   ├── install.yaml       # ArgoCD installation manifest
│   └── projects/          # ArgoCD project definitions
├── monitoring/
│   └── dashboards/        # Custom Grafana dashboard JSONs
├── rbac/
│   └── namespaces.yaml    # Namespace isolation + RBAC
├── .github/workflows/
│   ├── build-push.yml     # Docker build + ECR push
│   └── update-helm.yml    # Update image tag in values.yaml
└── README.md
```

## 🚀 Quick Start

```bash
# 1. Provision EKS cluster
cd terraform/eks-cluster && terraform init && terraform apply

# 2. Configure kubectl
aws eks update-kubeconfig --name my-cluster --region eu-west-1

# 3. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f argocd/install.yaml

# 4. Deploy application via ArgoCD
kubectl apply -f argocd/projects/
# ArgoCD will automatically sync the Helm chart to the cluster

# 5. Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## ✨ Features

- **GitOps** — ArgoCD watches this repo; any Git commit triggers reconciliation
- **Zero-downtime deploys** — Rolling update strategy with readiness probes
- **Auto-scaling** — HPA scales pods 3→10 based on CPU/memory metrics
- **Full observability** — Prometheus scraping all pods, Grafana dashboards, AlertManager
- **RBAC isolation** — Separate namespaces for production/staging/monitoring
- **Image scanning** — Trivy scans every image before ECR push

## 📖 Lessons Learned

ArgoCD's sync-wave feature was essential for ordering deployments correctly — databases must be ready before apps. The biggest operational learning: HPA and Cluster Autoscaler need to be tuned together, otherwise pods scale faster than nodes and sit Pending. Setting a PodDisruptionBudget prevented the rolling update from taking down all replicas simultaneously during a node drain.

## 🛠️ Tech Stack

`AWS EKS` `ArgoCD` `Helm` `Prometheus` `Grafana` `AlertManager` `Terraform` `GitHub Actions` `ECR` `HPA` `RBAC`

---

*Part of [Sreekar KV's cloud portfolio](https://sreekar424.github.io)*
