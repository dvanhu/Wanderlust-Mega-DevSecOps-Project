# Wanderlust — Mega DevSecOps Project 
[![CI/CD](https://img.shields.io/badge/CI%2FCD-Jenkins-D24939?style=flat-square&logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![GitOps](https://img.shields.io/badge/GitOps-ArgoCD-EF7B4D?style=flat-square&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Containerization-Docker-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![SonarQube](https://img.shields.io/badge/Code%20Quality-SonarQube-4E9BCD?style=flat-square&logo=sonarqube&logoColor=white)](https://www.sonarsource.com/products/sonarqube/)
[![OWASP](https://img.shields.io/badge/Security-OWASP%20Dependency%20Check-000000?style=flat-square&logo=owasp&logoColor=white)](https://owasp.org/www-project-dependency-check/)
[![Trivy](https://img.shields.io/badge/Scanner-Trivy-1904DA?style=flat-square&logo=aquasecurity&logoColor=white)](https://trivy.dev/)
[![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Observability-Grafana-F46800?style=flat-square&logo=grafana&logoColor=white)](https://grafana.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

A production-grade DevSecOps implementation for **WanderLust**, a MERN-stack travel blog application. This project wraps a fully automated CI/CD pipeline around the application, enforcing security at every stage — from code commit to live Kubernetes deployment — with GitOps-driven delivery and real-time observability.

> **Application credit:** The WanderLust MERN application was originally developed by [krishnaacharyaa](https://github.com/krishnaacharyaa/wanderlust). This repository is focused entirely on the DevSecOps engineering layer built on top of it.

---

## Pipeline Architecture

![DevSecOps GitOps Architecture](https://github.com/user-attachments/assets/371660a8-3aef-4425-99ae-7ae22482c5dd)

The pipeline is divided into two automated jobs:

**Jenkins CI Job** — triggered on every push to `main`
- Pulls source code from GitHub
- Runs OWASP Dependency Check against all third-party libraries
- Performs SonarQube static analysis and enforces the quality gate
- Executes Trivy filesystem scan
- Builds Docker images and pushes them to DockerHub

**Jenkins CD Job** — triggered automatically after CI passes
- Updates the Docker image tag in the Kubernetes manifests repository
- ArgoCD detects the manifest change and auto-syncs to the EKS cluster
- Prometheus and Grafana provide live cluster monitoring
- Gmail SMTP sends pipeline completion notifications

---

## Tech Stack

| Category | Tools |
|---|---|
| Application | React.js, Node.js, MongoDB, Redis |
| Containerization | Docker, DockerHub |
| CI/CD Orchestration | Jenkins (Master-Worker architecture) |
| Code Quality | SonarQube |
| Security Scanning | OWASP Dependency Check, Trivy |
| Container Orchestration | Kubernetes — AWS EKS |
| GitOps | ArgoCD |
| Monitoring | Prometheus, Grafana |
| Alerting | Alertmanager, Gmail SMTP |

---

## Jenkins CI/CD Pipeline

Built on a Jenkins master-worker node architecture. Every commit triggers a run through the complete security and build lifecycle with no manual intervention.

| Stage | What it does |
|---|---|
| Checkout SCM | Pulls latest source from GitHub |
| Workspace Cleanup | Removes stale artifacts from prior builds |
| Install Dependencies | Runs `npm install` for frontend and backend |
| OWASP Dependency Check | Scans third-party libraries against the NVD CVE database |
| Trivy Filesystem Scan | Checks the repository filesystem for vulnerabilities |
| SonarQube Analysis | Static analysis with quality gate enforcement — pipeline aborts on failure |
| Build Docker Images | Builds images for all four services |
| Trivy Image Scan | Scans built container images before any push |
| Docker Push | Pushes verified images to DockerHub |
| Post Actions | Archives build artifacts and sends email notification |

---

## SonarQube — Quality Gate

SonarQube runs against the full codebase on every pipeline execution. The quality gate must pass before any Docker image is built or pushed.

**Latest analysis results:**

| Metric | Result |
|---|---|
| Quality Gate | Passed |
| New Bugs | 0 |
| New Vulnerabilities | 0 |
| New Security Hotspots | 0 |
| New Code Smells | 0 |
| Reliability / Security / Maintainability | A / A / A |

---

## ArgoCD — GitOps Continuous Deployment

ArgoCD watches the Kubernetes manifests repository and reconciles the live cluster state with the declared state on every commit. No manual `kubectl apply` is ever needed.

**Deployment state:**

| Property | Value |
|---|---|
| Application Health | Healthy |
| Sync Status | Synced to `main` |
| Auto Sync | Enabled |
| Synced Resources | 10 |
| Healthy Resources | 17 |

**Deployed resources:**

- `mongo-pv` / `mongo-pvc` — Persistent volume and claim for MongoDB
- `backend-service`, `frontend-service`, `mongo-service`, `redis-service` — Kubernetes Services with EndpointSlices
- `backend-deployment`, `frontend-deployment`, `mongo-deployment`, `redis-deployment` — Deployments, each running 1/1 pods

---

## Monitoring — Prometheus and Grafana

Prometheus scrapes pod-level metrics from the cluster. Grafana surfaces them on the **Kubernetes / Compute Resources / Namespace (Pods)** dashboard with 5-second auto-refresh.

**Live network metrics (5-minute window):**

| Pod | Receive Bandwidth | Transmit Bandwidth | Received Packets |
|---|---|---|---|
| backend-deployment | 87.7 B/s | 39.0 B/s | 398 mp/s |
| mongo-deployment | 23.6 B/s | 69.8 B/s | 280 mp/s |
| redis-deployment | 11.7 B/s | 11.7 B/s | 182 mp/s |
| frontend-deployment | 0 B/s | 0 B/s | 0 p/s |

CPU usage is tracked per pod over a rolling 1-hour window with quota, request, and limit overlays — all pods run well within their declared resource boundaries.

---

## Repository Structure

```
Wanderlust-Mega-DevSecOps-Project/
├── backend/                        # Node.js API server
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   ├── services/
│   ├── Dockerfile
│   └── server.js
├── frontend/                       # React.js client
│   ├── src/
│   ├── Dockerfile
│   └── vite.config.ts
├── database/                       # MongoDB Dockerfile
├── kubernetes/                     # Kubernetes manifests
│   ├── backend.yaml
│   ├── frontend.yaml
│   ├── mongodb.yaml
│   ├── redis.yaml
│   ├── persistentVolume.yaml
│   └── persistentVolumeClaim.yaml
├── Assets/
│   └── DevSecOps+GitOps.gif        # Architecture diagram
├── Jenkinsfile                     # Full CI/CD pipeline definition
├── docker-compose.yml              # Local development setup
├── alertmanager-config.yaml        # Alertmanager configuration
└── wanderlust-alert.yaml           # Custom alert rules
```

---

## Key Highlights

- Security gates at every stage — OWASP CVE scanning, Trivy filesystem and image scanning, and SonarQube quality enforcement all run before a single artifact is published
- Zero-touch deployments — ArgoCD reconciles the cluster continuously; updating a manifest tag is all it takes to trigger a full rollout with rollback history intact
- Fully containerized workloads — all four services run as isolated pods on EKS with persistent storage for MongoDB
- Production-grade observability — per-pod CPU, memory, and network bandwidth metrics with configurable alerting via Alertmanager
- Clean separation of CI and CD — the CI pipeline owns build and security; the CD pipeline owns delivery and notification
