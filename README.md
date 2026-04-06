# Wanderlust — Mega DevSecOps Project

> A production-grade **DevSecOps** implementation for WanderLust, a MERN-stack travel blog application — featuring a fully automated CI/CD pipeline with integrated security scanning, GitOps-driven deployment on Kubernetes (EKS), and real-time observability via Prometheus and Grafana.

![WanderLust App](https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project/raw/main/screenshots/wanderlust-app.png)

---

## About This Project

**WanderLust** is a MERN travel blog where users can explore and share travel stories across categories like Adventure, Nature, City, and Beaches.

> **Application credit:** The WanderLust MERN app was originally developed by [krishnaacharyaa](https://github.com/krishnaacharyaa/wanderlust). This repository is focused entirely on the **DevSecOps engineering layer** built on top of it — CI/CD, security, containerization, orchestration, and monitoring.

---

## Architecture & Pipeline Flow

![DevSecOps Architecture](https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project/raw/main/screenshots/devsecops-architecture.png)

The end-to-end pipeline is split into two jobs:

**Jenkins CI Job**
- Developer pushes code → GitHub triggers Jenkins
- OWASP Dependency Check → SonarQube Code & Quality Gate Analysis
- Trivy Filesystem Scan → Docker Build & Push to DockerHub

**Jenkins CD Job**
- Updates Docker image version in the manifests repo
- ArgoCD detects the change and auto-syncs to Kubernetes (EKS)
- Prometheus + Grafana monitor the live cluster
- Email notifications on pipeline completion

---

## Tech Stack

| Category | Tools |
|---|---|
| **Application** | React.js, Node.js, MongoDB, Redis |
| **Containerization** | Docker, DockerHub |
| **CI/CD** | Jenkins (Master-Worker), ArgoCD |
| **Code Quality** | SonarQube |
| **Security Scanning** | OWASP Dependency Check, Trivy |
| **Orchestration** | Kubernetes (AWS EKS) |
| **GitOps** | ArgoCD |
| **Monitoring** | Prometheus, Grafana |
| **Cloud** | AWS EC2, AWS EKS |
| **Notifications** | Email (Gmail SMTP) |

---

## Live Application

Both the frontend and backend are containerized and deployed to the EKS cluster.

### Frontend — WanderLust Travel Blog
![Frontend](https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project/raw/main/screenshots/frontend-running.png)

### Backend — Health Check
![Backend](https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project/raw/main/screenshots/backend-health.png)

---

## Jenkins CI/CD Pipeline

A complete DevSecOps pipeline was built on Jenkins with a **master-worker node architecture**. Every commit to the `main` branch triggers an automated run through the full security and build lifecycle.

### Pipeline Stages

| Stage | Description |
|---|---|
| Checkout SCM | Pull latest source from GitHub |
| Workspace Cleanup | Remove stale artifacts from previous builds |
| Checkout Code | Verified source checkout |
| Install Dependencies | `npm install` for frontend & backend |
| OWASP Dependency Check | Scan third-party libraries for known CVEs |
| Trivy Filesystem Scan | Filesystem-level vulnerability scan |
| SonarQube Analysis | Static code analysis + quality gate enforcement |
| Build Docker Images | Multi-service Docker image builds |
| Trivy Image Scan | Container image vulnerability assessment |
| Docker Push | Push verified images to DockerHub |
| Post Actions | Archive artifacts + send email notifications |

### Pipeline Run — All Stages Passed

![Jenkins Pipeline](https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project/raw/main/screenshots/jenkins-pipeline.png)

---

## SonarQube — Code Quality Gate

SonarQube is integrated into the pipeline and enforces a quality gate **before any image is built or pushed**. A failed quality gate blocks the pipeline from proceeding.

**Results on latest analysis:**

| Metric | Result |
|---|---|
| Quality Gate | **Passed** |
| New Bugs | 0 |
| New Vulnerabilities | 0 |
| New Security Hotspots | 0 |
| New Code Smells | 0 |
| Reliability / Security / Maintainability | **A / A / A** |

![SonarQube](https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project/raw/main/screenshots/sonarqube.png)

---

## 🔄 ArgoCD — GitOps Continuous Deployment

ArgoCD watches the Kubernetes manifests repository and **auto-syncs** any changes directly to the EKS cluster — no manual `kubectl apply` required.

**Deployment Status:**

| Property | Value |
|---|---|
| App Health | Healthy |
| Sync Status | Synced to `main` |
| Auto Sync | Enabled |
| Synced Resources | 10 |
| Healthy Resources | 17 |

### Application Resource Tree

![ArgoCD Services](https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project/raw/main/screenshots/argocd-1.png)

![ArgoCD Deployments](https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project/raw/main/screenshots/argocd-2.png)

**Deployed resources include:**
- `mongo-pv` / `mongo-pvc` — Persistent storage for MongoDB
- `backend-service`, `frontend-service`, `mongo-service`, `redis-service` — Kubernetes Services with EndpointSlices
- `backend-deployment`, `frontend-deployment`, `mongo-deployment`, `redis-deployment` — Deployments (1/1 running)

---

## Monitoring — Prometheus & Grafana

Prometheus scrapes metrics from the cluster and Grafana visualizes them in real time on the **Kubernetes / Compute Resources / Namespace (Pods)** dashboard.

### Network Usage (Last 5 Minutes)

![Grafana Network](https://github.com/dvanhu/Wanderlust-Mega-DevSecOps-Project/raw/main/screenshots/grafana-network.png)

| Pod | Receive BW | Transmit BW | Received Packets |
|---|---|---|---|
| backend-deployment | 87.7 B/s | 39.0 B/s | 398 mp/s |
| mongo-deployment | 23.6 B/s | 69.8 B/s | 280 mp/s |
| redis-deployment | 11.7 B/s | 11.7 B/s | 182 mp/s |
| frontend-deployment | 0 B/s | 0 B/s | 0 p/s |

### CPU Usage (Last 1 Hour)

![Grafana CPU]

CPU usage tracked across all pods over 1 hour with per-pod breakdowns for quota, requests, and limits — confirming all pods are running well within their resource boundaries.

---

## Repository Structure

```
Wanderlust-Mega-DevSecOps-Project/
├── frontend/               # React.js frontend
├── backend/                # Node.js backend
├── k8s/                    # Kubernetes manifests
│   ├── frontend-deployment.yaml
│   ├── backend-deployment.yaml
│   ├── mongo-deployment.yaml
│   ├── redis-deployment.yaml
│   └── services/
├── Jenkinsfile             # Full CI/CD pipeline definition
├── docker-compose.yml      # Local development setup
└── screenshots/            # Pipeline & deployment screenshots
```

---

## Key Highlights

- **Security-first pipeline** — every commit is scanned by OWASP, Trivy (filesystem + image), and SonarQube before any artifact is published
- **Zero-touch deployments** — ArgoCD auto-syncs the cluster the moment manifests are updated, with full rollback history available
- **Fully containerized** — all 4 services (frontend, backend, MongoDB, Redis) run as isolated pods inside EKS
- **Production-grade observability** — real-time CPU, memory, and network metrics per pod via Grafana dashboards
- **Automated notifications** — email alerts on pipeline success or failure via Gmail SMTP
