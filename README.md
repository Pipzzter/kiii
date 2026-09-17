# CI/CD Demo App — КИИИ проектна задача

Full-stack апликација (React + FastAPI + PostgreSQL) докеризирана, оркестрирана со Docker
Compose, поставена на CI/CD пајплајн (GitHub Actions → Docker Hub) и со Kubernetes манифести
за деплојмент во посебен namespace.

App source (`backend/`, `frontend/`) originates from
[fastapi/full-stack-fastapi-template](https://github.com/fastapi/full-stack-fastapi-template);
the Dockerfiles, `docker-compose.yml`, CI/CD pipeline and all `k8s/` manifests were written from
scratch for this project.

## Структура

```
backend/    FastAPI REST API (Python)
frontend/   React app (Vite + TypeScript), served by nginx, proxies /api to backend
k8s/        Kubernetes manifests (namespace, ConfigMaps, Secrets, Deployments,
            Services, Ingress, StatefulSet for the db)
.github/workflows/ci-cd.yml   CI/CD: build + push images to Docker Hub on push to main
docker-compose.yml            Local orchestration: frontend + backend + db
```

## Локално со Docker Compose

```bash
docker compose up --build
```

- Frontend: http://localhost:8080
- Backend API docs: http://localhost:8000/docs

`.env` at the repo root has dev-only placeholder values (`changethis`) so this works with zero
setup — replace them for anything beyond a local demo.

## CI/CD

`.github/workflows/ci-cd.yml` builds and pushes `backend` and `frontend` images to Docker Hub on
every push to `main`, tagged `latest` and with the commit SHA. Requires two repo secrets:

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN` (Docker Hub access token, not your password)

## Kubernetes

```bash
kubectl apply -k k8s/
kubectl get all -n ciid-project
```

Before applying, replace `REPLACE_DOCKERHUB_USERNAME` in `k8s/07-backend-deployment.yaml` and
`k8s/09-frontend-deployment.yaml` with the real Docker Hub username the images were pushed to.

Manifests:
- `00-namespace.yaml` — dedicated `ciid-project` namespace
- `01/02/03/04` — db ConfigMap, Secret, headless Service, StatefulSet (PostgreSQL)
- `05/06/07/08` — backend ConfigMap, Secret, Deployment, Service
- `09/10` — frontend Deployment, Service
- `11-ingress.yaml` — routes `/api` to backend, `/` to frontend

To reach it through the Ingress, point a host entry (e.g. `ciid.local`) at the cluster's ingress
controller IP, then open `http://ciid.local`.
