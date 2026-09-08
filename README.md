# 🚀 Local DevSecOps GitOps Kubernetes Lab

A hands-on **DevOps / DevSecOps interview project** demonstrating container deployment, Kubernetes, Helm, Git, GitHub, Argo CD and GitOps — built completely on a local Windows machine without using AWS, Azure or GCP compute infrastructure.

## 🎯 Project Objective

Build and operate a realistic application deployment workflow where a change made in Git is automatically reconciled into Kubernetes through Argo CD and Helm.

The project demonstrates not only deployment, but also **scaling, rollback, configuration management and troubleshooting**.

---

## 🏗️ Architecture

```text
Developer
   │
   │ Git commit / push
   ▼
GitHub (main)
   │
   │ Argo CD watches repository
   ▼
Argo CD
   │
   │ Helm rendering + GitOps reconciliation
   ▼
Kind Kubernetes Cluster
   │
   └── devsecops namespace
        │
        ├── Helm Release: app-release
        │
        ├── Deployment: app-release-app-chart
        │      ├── NGINX Pod 1
        │      ├── NGINX Pod 2
        │      └── NGINX Pod 3
        │
        ├── Service: app-release-app-chart
        │
        └── ConfigMap: app-release-app-chart-html
               └── index.html
                      │
                      ▼
              NGINX Web Server
                      │
                      ▼
              kubectl port-forward
                      │
                      ▼
              Browser :8080
```

---

## 💻 Technology Stack

| Technology | Purpose |
|---|---|
| 🪟 Windows | Local development environment |
| 🐳 Docker Desktop | Container runtime |
| ☸️ Kind | Local Kubernetes cluster |
| 🔧 kubectl | Kubernetes administration and troubleshooting |
| 📦 Helm | Kubernetes application packaging and release management |
| 🌿 Git | Version control |
| 🐙 GitHub | Remote Git repository / source of truth |
| 🔄 Argo CD | GitOps continuous delivery and reconciliation |
| 🌐 NGINX | Web application/server |

> ☁️ **No AWS EKS, Azure AKS, GCP GKE or cloud compute infrastructure is required for this lab.** GitHub is used as the remote Git hosting service.

---

## 📁 Repository Structure

```text
k8s-argo-demo/
│
├── app-chart/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── .helmignore
│   │
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── serviceaccount.yaml
│       ├── configmap.yaml
│       ├── hpa.yaml
│       ├── httproute.yaml
│       ├── NOTES.txt
│       ├── _helpers.tpl
│       └── tests/
│
├── deployment/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── namespace.yaml
│
├── .argocd-app.yaml
└── README.md
```

---

## ☸️ Kubernetes Implementation

### Namespace

```text
devsecops
```

### Application

```text
NGINX: nginx:1.16.0
```

### Deployment

```text
app-release-app-chart
```

The Helm values were changed from **1 replica to 3 replicas**, demonstrating Git-driven scaling.

### Service

A Kubernetes `ClusterIP` service exposes the NGINX application inside the cluster.

### Local Browser Access

```powershell
kubectl port-forward svc/app-release-app-chart -n devsecops 8080:80
```

Then open:

```text
http://localhost:8080
```

---

## 📦 Helm Implementation

The application is packaged as a Helm chart under:

```text
app-chart/
```

Important Helm concepts demonstrated:

- 📦 Helm chart structure
- ⚙️ `values.yaml`
- 🔢 Replica configuration
- 🧩 Helm templates
- ✅ `helm lint`
- 🔍 `helm template`
- 🚀 Helm install/upgrade
- 🕐 Helm release history
- ↩️ Helm rollback
- 🗂️ ConfigMap-based application configuration

### Helm Rollback Demonstration

The release was rolled back to an earlier revision:

```powershell
helm rollback app-release 1 -n devsecops
```

This demonstrated practical release versioning and rollback capability.

---

## 🔄 GitOps with Argo CD

Argo CD was installed inside the local Kubernetes cluster and configured to watch the GitHub repository.

Source:

```text
Repository: MojjadaCode-Git/k8s-argo-demo
Branch: main
Path: app-chart
```

Destination:

```text
Kubernetes namespace: devsecops
```

The Argo CD Application uses automated synchronization with:

- 🔄 Automated sync
- 🧹 Prune
- 🩹 Self-heal
- 🔍 Git revision tracking
- ♻️ Kubernetes reconciliation

### GitOps Flow

```text
Code/Configuration Change
        ↓
Git Commit
        ↓
GitHub main
        ↓
Argo CD detects new revision
        ↓
Helm renders Kubernetes manifests
        ↓
Argo CD reconciles desired state
        ↓
Kubernetes resources updated
        ↓
Pods running the new configuration
```

---

## 🗂️ ConfigMap-Based Application Configuration

The custom web page is stored in:

```text
app-chart/templates/configmap.yaml
```

The ConfigMap contains the application's `index.html` and is mounted into NGINX at:

```text
/usr/share/nginx/html/index.html
```

The final page displays:

```text
Welcome to DevSecOps Interview Session
Deployment Completed Successfully!
Git | Helm | Argo CD | Kubernetes
Automated GitOps Deployment
Thank You So Much!
```

The page also contains CSS animations for the background and heading.

---

## 🛠️ Real Troubleshooting Scenario

A practical troubleshooting scenario was encountered during the project.

### Problem

The browser initially displayed the default NGINX page instead of the custom application page.

### Investigation

Checked the ConfigMap:

```powershell
kubectl get configmap app-release-app-chart-html -n devsecops
```

Checked Argo CD:

```powershell
kubectl get application app-release -n argocd
```

Checked the Git revision being used by Argo CD:

```powershell
kubectl get application app-release -n argocd -o jsonpath="{.status.sync.revision}"
```

### Root Cause

Argo CD was initially referencing an older Git revision, so the newly added ConfigMap was not present in the Kubernetes namespace.

### Resolution

Argo CD was hard-refreshed:

```powershell
kubectl -n argocd annotate application app-release argocd.argoproj.io/refresh=hard --overwrite
```

Argo CD then picked up the latest Git revision, created the ConfigMap and recreated the application Pods.

A second practical issue involved the ConfigMap being mounted using `subPath`. After a ConfigMap content change, the existing Pod continued serving the old mounted file, so a rolling restart was performed:

```powershell
kubectl rollout restart deployment app-release-app-chart -n devsecops
```

After the rollout, the new Pods loaded the updated `index.html` and the browser displayed the new content.

---

## 🔍 Useful Validation Commands

### Check Pods

```powershell
kubectl get pods -n devsecops
```

### Check Services

```powershell
kubectl get svc -n devsecops
```

### Check ConfigMap

```powershell
kubectl get configmap -n devsecops
```

### Check Argo CD Application

```powershell
kubectl get application app-release -n argocd
```

### Check Argo CD Revision

```powershell
kubectl get application app-release -n argocd -o jsonpath="{.status.sync.revision}"
```

### Inspect the HTML served by the running Pod

```powershell
kubectl exec -n devsecops deploy/app-release-app-chart -- cat /usr/share/nginx/html/index.html
```

### Validate Helm Chart

```powershell
helm lint .\app-chart
helm template app-release .\app-chart -n devsecops
```

### Check Helm History

```powershell
helm history app-release -n devsecops
```

---

## 📈 What This Project Demonstrates

### DevOps Skills

- ☸️ Kubernetes deployment and service management
- 📦 Helm packaging and release management
- 🌿 Git version control
- 🐙 GitHub repository management
- 🔄 CI/CD-style GitOps workflow
- 📈 Application scaling
- ↩️ Release rollback
- 🛠️ Production-style troubleshooting

### DevSecOps / GitOps Concepts

- 🔐 Configuration managed as code
- 🔄 Git as the desired-state source
- 🤖 Automated reconciliation
- 🩹 Self-healing deployment model
- 🧹 Automated pruning
- 🔍 Revision-based troubleshooting
- 📋 Declarative Kubernetes manifests

---

## ☁️ Local Lab vs Production

This project intentionally uses Kind to simulate the Kubernetes layer locally.

```text
LOCAL LAB
Docker Desktop
     ↓
Kind Kubernetes
     ↓
Helm
     ↓
Argo CD
     ↓
NGINX

PRODUCTION EQUIVALENT
Cloud infrastructure
     ↓
AWS EKS / Azure AKS
     ↓
Helm
     ↓
Argo CD
     ↓
Microservices / Applications
```

The **GitOps, Helm and Kubernetes concepts remain the same**; the main difference is that the Kubernetes infrastructure in this demonstration runs locally rather than in a cloud-managed cluster.

---

## 🎤 Interview Talking Point

> "I built a local DevSecOps GitOps lab using Docker Desktop and Kind. I packaged an NGINX application with Helm, stored the chart in GitHub, configured Argo CD for automated synchronization, demonstrated scaling from one to three replicas, performed a Helm rollback, used a ConfigMap for application configuration, and troubleshot an Argo CD revision and ConfigMap propagation issue. Finally, I validated the application through a Kubernetes service exposed locally using kubectl port-forward."

---

## ✅ Final Result

The project successfully demonstrates an end-to-end flow:

```text
GitHub
  ↓
Argo CD
  ↓
Helm
  ↓
Kubernetes
  ↓
ConfigMap + Deployment + Service
  ↓
NGINX Pods ×3
  ↓
localhost:8080
  ↓
🎉 Working Application
```

**Built as a practical DevOps / DevSecOps interview demonstration. 🚀**
