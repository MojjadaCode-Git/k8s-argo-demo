# Kubernetes GitOps DevSecOps Lab

> A production-inspired local Kubernetes project demonstrating **Helm, GitOps, Argo CD, configuration management, scaling, rollback, and troubleshooting**.

![Kubernetes](https://img.shields.io/badge/Kubernetes-Local-blue?logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-Chart-0F1689?logo=helm&logoColor=white)
![Argo%20CD](https://img.shields.io/badge/Argo%20CD-GitOps-EF7B4D?logo=argo&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Desktop-2496ED?logo=docker&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-Source%20Control-181717?logo=github&logoColor=white)

## Overview

This project was built as a practical **DevOps / DevSecOps interview demonstration**. It implements an end-to-end GitOps workflow on a local Kubernetes cluster using Docker Desktop and Kind.

The goal is to demonstrate how a configuration change moves from Git to a running Kubernetes workload through **Argo CD reconciliation and Helm rendering**.

**No AWS, Azure, or GCP compute infrastructure is required.** The Kubernetes control plane runs locally through Kind.

---

## Architecture

```text
                         GitOps Workflow

Developer
   |
   | git commit / push
   v
GitHub (main)
   |
   | repository reconciliation
   v
Argo CD
   |
   | Helm rendering + sync
   v
Kind Kubernetes Cluster
   |
   +-----------------------------+
   | devsecops namespace          |
   |                              |
   |  Helm Release: app-release   |
   |          |                   |
   |          v                   |
   |  Deployment: app-chart       |
   |       |       |       |      |
   |      Pod     Pod     Pod     |
   |       |       |       |      |
   |       +-------+-------+      |
   |               |              |
   |          ClusterIP Service   |
   |               |              |
   |          NGINX application   |
   |                              |
   |  ConfigMap -> index.html     |
   +-----------------------------+
                   |
                   | kubectl port-forward
                   v
             Browser :8080
```

### Production mapping

The same Kubernetes and GitOps concepts can be used with a managed cloud cluster:

```text
Local Lab                         Production
---------                         ----------
Kind                              AWS EKS / Azure AKS
Docker Desktop                    Cloud container runtime
Helm                              Helm
Argo CD                           Argo CD
NGINX                             Microservices / applications
GitHub                            GitHub
```

The infrastructure is different; the **declarative Kubernetes, Helm and GitOps workflow remains conceptually the same**.

---

## Technology Stack

| Technology | Role |
|---|---|
| Docker Desktop | Local container runtime |
| Kind | Local Kubernetes cluster |
| Kubernetes | Container orchestration |
| kubectl | Cluster administration and troubleshooting |
| Helm | Application packaging and release management |
| Git | Version control |
| GitHub | Remote repository and desired-state source |
| Argo CD | GitOps continuous delivery and reconciliation |
| NGINX | Web application/server |

---

## Repository Structure

```text
k8s-argo-demo/
├── app-chart/
│   ├── Chart.yaml
│   ├── values.yaml
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

## Kubernetes Implementation

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

The Helm configuration was changed from **1 replica to 3 replicas**, demonstrating Git-driven scaling.

### Service

A `ClusterIP` Service exposes the application inside the Kubernetes cluster.

### Local access

```powershell
kubectl port-forward svc/app-release-app-chart -n devsecops 8080:80
```

Open:

```text
http://localhost:8080
```

---

## Helm

The application is packaged as a Helm chart under `app-chart/`.

The project demonstrates:

- Helm chart structure
- `values.yaml`
- Helm templating
- Replica configuration
- `helm lint`
- `helm template`
- Release history
- Upgrade workflow
- Rollback
- ConfigMap-based configuration

### Rollback demonstration

```powershell
helm rollback app-release 1 -n devsecops
```

This demonstrated practical release versioning and recovery using Helm.

---

## GitOps with Argo CD

Argo CD watches the GitHub repository and reconciles the desired Kubernetes state.

```text
Repository : MojjadaCode-Git/k8s-argo-demo
Branch     : main
Path       : app-chart
Namespace  : devsecops
```

The application is configured for:

- Automated synchronization
- Self-healing
- Pruning of removed resources
- Git revision tracking
- Kubernetes reconciliation

### End-to-end flow

```text
Configuration change
        |
        v
Git commit
        |
        v
GitHub main
        |
        v
Argo CD detects revision
        |
        v
Helm renders manifests
        |
        v
Argo CD reconciles desired state
        |
        v
Kubernetes resources updated
        |
        v
Running application
```

---

## Configuration Management with ConfigMap

The custom web page is defined in:

```text
app-chart/templates/configmap.yaml
```

The ConfigMap contains `index.html` and mounts it into NGINX at:

```text
/usr/share/nginx/html/index.html
```

The page demonstrates that application configuration can be managed through Git and propagated through the GitOps workflow.

---

## Troubleshooting Case Study

One of the most useful parts of this project was troubleshooting a real deployment issue.

### Symptom

The browser continued to display the default NGINX page instead of the custom application page.

### Investigation

Checked the ConfigMap:

```powershell
kubectl get configmap app-release-app-chart-html -n devsecops
```

Checked Argo CD:

```powershell
kubectl get application app-release -n argocd
```

Checked the Git revision used by Argo CD:

```powershell
kubectl get application app-release -n argocd -o jsonpath="{.status.sync.revision}"
```

### Root cause

Argo CD was initially using an older Git revision, so the newly added ConfigMap was not present in Kubernetes.

### Resolution

Argo CD was hard-refreshed:

```powershell
kubectl -n argocd annotate application app-release argocd.argoproj.io/refresh=hard --overwrite
```

Argo CD then reconciled the latest Git revision and created the ConfigMap.

A second issue was identified because the ConfigMap file was mounted using `subPath`. The already-running Pods continued serving the old mounted file, so a rolling restart was performed:

```powershell
kubectl rollout restart deployment app-release-app-chart -n devsecops
```

The new Pods loaded the updated file and the browser displayed the new application page.

This demonstrates a complete troubleshooting process: **symptom → investigation → root cause → corrective action → validation**.

---

## Validation Commands

### Kubernetes

```powershell
kubectl get pods -n devsecops
kubectl get svc -n devsecops
kubectl get configmap -n devsecops
```

### Argo CD

```powershell
kubectl get application app-release -n argocd
kubectl get application app-release -n argocd -o jsonpath="{.status.sync.revision}"
```

### Inspect the running application

```powershell
kubectl exec -n devsecops deploy/app-release-app-chart -- cat /usr/share/nginx/html/index.html
```

### Helm validation

```powershell
helm lint .\app-chart
helm template app-release .\app-chart -n devsecops
helm history app-release -n devsecops
```

---

## Key DevOps Skills Demonstrated

**Kubernetes**
- Deployments
- Pods and replicas
- Services
- ConfigMaps
- Namespaces
- Rolling restarts
- Troubleshooting

**Helm**
- Chart structure
- Values and templates
- Release management
- Rollback
- Manifest validation

**GitOps**
- Git as source of truth
- Argo CD synchronization
- Reconciliation
- Self-healing
- Automated pruning
- Revision-based troubleshooting

**DevSecOps mindset**
- Infrastructure/configuration as code
- Declarative deployments
- Repeatable releases
- Version-controlled changes
- Controlled recovery and troubleshooting

---

## Interview Summary

> I built a local DevSecOps GitOps lab using Docker Desktop and Kind. I packaged an NGINX application with Helm, stored the chart in GitHub, configured Argo CD for automated synchronization, demonstrated scaling from one to three replicas, performed a Helm rollback, managed application configuration with a ConfigMap, and troubleshot an Argo CD revision and ConfigMap propagation issue. Finally, I validated the application through a Kubernetes Service exposed locally using kubectl port-forward.

---

## Result

The final deployment successfully demonstrates:

```text
GitHub
   |
   v
Argo CD
   |
   v
Helm
   |
   v
Kubernetes
   |
   +--> ConfigMap
   +--> Deployment (3 replicas)
   +--> Service
   |
   v
NGINX
   |
   v
localhost:8080
```

**Status: Completed successfully.**

Built as a practical DevOps / DevSecOps interview project.