# 🚀 GitOps-Powered Auto-Deploy Platform on Kubernetes

> A production-grade GitOps CI/CD pipeline with ArgoCD, multi-environment deployments, canary releases, and auto-rollback.

## 📁 Project Structure

```
gitops-k8s-project/
├── app/                          # Node.js Application
│   ├── src/index.js
│   ├── Dockerfile
│   └── package.json
├── k8s/
│   ├── base/                     # Shared K8s manifests
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   └── kustomization.yaml
│   └── overlays/                 # Per-environment configs (Kustomize)
│       ├── dev/
│       ├── staging/
│       └── prod/
├── argocd/
│   └── applications.yaml         # ArgoCD app definitions (all 3 envs)
├── monitoring/
│   ├── prometheus/config.yaml    # Scrape configs + alert rules
│   └── grafana/                  # Dashboard JSONs
└── .github/workflows/
    └── ci-cd.yaml                # Full CI/CD pipeline
```

---

## ⚡ How It Works

```
Code Push → GitHub Actions CI
    ↓
Build & Test App
    ↓
Build Docker Image → Push to DockerHub
    ↓
Update image tag in k8s/overlays/[env]/kustomization.yaml
    ↓
Git commit + push (GitOps!)
    ↓
ArgoCD detects Git change (polls every 3 min)
    ↓
ArgoCD syncs → deploys to Kubernetes
    ↓
Health checks pass → Slack notification ✅
         OR
Health checks fail → Auto-rollback → Slack alert ❌
```

---

## 🛠️ Setup Guide

### Step 1: Prerequisites

```bash
# Install these tools
brew install kubectl      # Kubernetes CLI
brew install minikube     # Local K8s cluster
brew install argocd       # ArgoCD CLI
brew install kustomize    # K8s config manager
```

### Step 2: Start Local Cluster

```bash
# Start Minikube
minikube start --cpus=4 --memory=4096 --driver=docker

# Verify
kubectl get nodes
```

### Step 3: Install ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=120s

# Get admin password
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Open: https://localhost:8080  (admin / <password above>)
```

### Step 4: Create Namespaces

```bash
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod
kubectl create namespace monitoring
```

### Step 5: Configure GitHub Secrets

Go to: GitHub Repo → Settings → Secrets → Actions

Add these secrets:
```
DOCKERHUB_USERNAME    → your Docker Hub username
DOCKERHUB_TOKEN       → Docker Hub access token
SLACK_WEBHOOK_URL     → Slack Incoming Webhook URL (optional)
```

### Step 6: Update Your Repo URL

Edit `argocd/applications.yaml` and replace:
```yaml
repoURL: https://github.com/YOUR_USERNAME/gitops-k8s-project
```

### Step 7: Deploy ArgoCD Applications

```bash
# Replace your image name in base deployment
sed -i 's/your-dockerhub-username/YOUR_ACTUAL_USERNAME/g' k8s/base/deployment.yaml

# Apply ArgoCD apps
kubectl apply -f argocd/applications.yaml

# Check status
argocd app list
```

### Step 8: Push Code & Watch the Magic! 🎉

```bash
git add .
git commit -m "feat: initial deployment"
git push origin main

# ArgoCD will auto-sync within 3 minutes
# Watch live:
argocd app get gitops-demo-dev --watch
```

---

## 🌍 Multi-Environment Details

| Environment | Namespace | Replicas | Auto-Prune | Branch |
|-------------|-----------|----------|------------|--------|
| Dev         | dev       | 1        | ✅ Yes     | develop |
| Staging     | staging   | 2        | ✅ Yes     | develop |
| Prod        | prod      | 3        | ❌ No (safe) | main |

---

## 🔄 Auto-Rollback Setup

ArgoCD's `selfHeal: true` handles drift correction. For app-level rollback:

```bash
# Manual rollback to previous version
argocd app rollback gitops-demo-prod

# Or via kubectl
kubectl rollout undo deployment/gitops-demo-app -n prod

# Check rollout history
kubectl rollout history deployment/gitops-demo-app -n prod
```

---

## 📊 Monitoring Stack

```bash
# Install Prometheus + Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring

# Access Grafana
kubectl port-forward svc/monitoring-grafana 3001:80 -n monitoring
# URL: http://localhost:3001  (admin / prom-operator)

# Apply custom configs
kubectl apply -f monitoring/prometheus/config.yaml
```

---

## 🎯 Hatke Features to Add Next

- [ ] **Canary Deployments** with Argo Rollouts (20% traffic split)
- [ ] **Secrets Management** with Sealed Secrets or Vault
- [ ] **Network Policies** for zero-trust networking
- [ ] **OPA/Gatekeeper** for policy enforcement
- [ ] **Chaos Engineering** with Chaos Monkey

---

## 📝 Useful Commands

```bash
# Check app health
argocd app get gitops-demo-dev

# Force sync
argocd app sync gitops-demo-dev

# View pod logs
kubectl logs -l app=gitops-demo-app -n dev --tail=50 -f

# Port forward to test app
kubectl port-forward svc/gitops-demo-app-svc 8888:80 -n dev
curl http://localhost:8888

# Check deployment status
kubectl rollout status deployment/gitops-demo-app -n prod
```

---

Made with ❤️ for DevOps learning | Uses: Kubernetes + ArgoCD + GitHub Actions + Kustomize + Prometheus
# GitOps Project
