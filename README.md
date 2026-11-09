<h1 align="center"> FastAPI Backend — Multi-Node Kubernetes Three-Tier Application </h1>

<p align="center">
  <img src="https://img.shields.io/badge/Kubernetes-v1.30-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes" />
  <img src="https://img.shields.io/badge/FastAPI-0.111-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI" />
  <img src="https://img.shields.io/badge/PostgreSQL-15-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL" />
  <img src="https://img.shields.io/badge/Docker-Hub-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white" alt="AWS" />
  <img src="https://img.shields.io/badge/GitHub_Actions-CI%2FCD-2088FF?style=for-the-badge&logo=github-actions&logoColor=white" alt="CI/CD" />
</p>

---

## 📌 Architectural Overview

This repository contains the backend REST API service for a production-grade three-tier contact management application. The service is orchestrated inside a multi-node Kubernetes cluster deployed on AWS EC2 Ubuntu instances.

```
       [ Client Web Browser ]
                 │
                 ▼ (HTTP Port 30007)
       [ Next.js Frontend Pods ]
                 │
                 ▼ (REST API Port 30008)
     ┌──────────────────────────────────────────────┐
     │  Kubernetes Service: backend-service         │
     │  (Type: NodePort 30008 -> Port 8000)         │
     └──────────────────────┬───────────────────────┘
                            │
               ┌────────────┴────────────┐
               ▼                         ▼
      [ FastAPI Pod 1 ]         [ FastAPI Pod 2 ]
               │                         │
               └────────────┬────────────┘
                            │ (Port 5432)
                            ▼
     ┌──────────────────────────────────────────────┐
     │  PostgreSQL Pod (ClusterIP: 5432)            │
     │  PersistentVolume: /mnt/postgres-data (host) │
     └──────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack & Key Components

| Layer | Component | Detail |
|---|---|---|
| **Language & Runtime** | Python 3.11 | Slim Alpine container image |
| **Web Framework** | FastAPI + Uvicorn | High-performance asynchronous REST API |
| **ORM & Database Engine** | SQLAlchemy + `psycopg2` | Structured data mapping & automatic schema initialization |
| **Target Database** | PostgreSQL 15 | Persistent state via `PersistentVolumeClaim` |
| **Orchestration** | Kubernetes v1.30 (Flannel CNI) | AWS EC2 Master + 2 Worker architecture |
| **Automated Pipeline** | GitHub Actions | Docker Hub image push & SSH Master rolling updates |

---

## 📂 Repository File Structure

```text
.
├── .github/
│   └── workflows/
│       └── deploy.yml          # Automated CI/CD pipeline (Docker build + SSH rollout)
├── k8s/
│   └── backend-deployment.yaml # Kubernetes Deployment (2 Replicas) & NodePort Service
├── Dockerfile                  # Container build instructions
├── main.py                     # FastAPI routes, CORS configuration, and DB startup events
└── requirements.txt            # Python dependencies (fastapi, uvicorn, sqlalchemy, psycopg2)
```

---

## 🚀 REST API Endpoints & Specification

FastAPI automatically serves interactive Swagger documentation on:
`http://<WORKER_PUBLIC_IP>:30008/docs`

| Method | Endpoint | Description | Request Body / Parameters |
|---|---|---|---|
| `GET` | `/contacts` | Fetch all contact records | *None* |
| `POST` | `/contacts` | Create a new contact | `{"name": str, "email": str, "phone": str, "address": str}` |
| `GET` | `/contacts/{id}` | Fetch a single contact by ID | `id` (path parameter) |
| `PUT` | `/contacts/{id}` | Update an existing contact record | `id` + JSON payload |
| `DELETE` | `/contacts/{id}` | Delete a contact record | `id` (path parameter) |

---

## 🔒 Automated CI/CD Pipeline

The GitHub Actions workflow defined in `.github/workflows/deploy.yml` triggers automatically on every `git push` to `main`:

1. **Build & Push:** Uses `docker/build-push-action` to build a production container image and pushes it to Docker Hub under `salalhamza/fastapi-backend:latest`.
2. **Cluster Authentication:** Connects to the AWS EC2 Master Node via SSH (`appleboy/ssh-action`) using deployment keys.
3. **Rolling Deployment:** Applies updated Kubernetes deployment manifests to the `three-tier-app` namespace and executes:
   ```bash
   kubectl rollout restart deployment/fastapi-backend -n three-tier-app
   kubectl rollout status deployment/fastapi-backend -n three-tier-app --timeout=120s
   ```

---

## ⚙️ Environment & Cluster Secrets

The following GitHub Repository Secrets manage automated continuous deployment:

* `DOCKERHUB_USERNAME`: Docker Hub container registry username.
* `DOCKERHUB_TOKEN`: Docker Hub Personal Access Token (Read & Write permissions).
* `K8S_MASTER_IP`: Public Elastic/IPv4 address of the Kubernetes Control Plane.
* `K8S_SSH_USER`: SSH login user (`ubuntu`).
* `K8S_SSH_KEY`: SSH private key authorized on the Kubernetes Master node.

---

## 💻 Local Standalone Execution

```bash
# 1. Clone repository
git clone https://github.com/SalalHamza/k8s-Fastapi-Backend.git
cd k8s-Fastapi-Backend

# 2. Configure virtual environment
python3 -m venv venv
source venv/bin/activate

# 3. Install packages
pip install -r requirements.txt

# 4. Set environment variable & run server
export DATABASE_URL="postgresql://dbuser:StrongPassword123@localhost:5432/userdb"
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```
