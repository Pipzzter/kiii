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

Cluster (local, via [k3d](https://k3d.io)):

```bash
k3d cluster create --config k8s/k3d-config.yaml
kubectl apply -k k8s/
kubectl get all -n ciid-project
```

`k8s/k3d-config.yaml` pins the API server to `127.0.0.1` (k3d's default of
`host.docker.internal` doesn't resolve reliably on every machine/network) and publishes port 80
for the Ingress, so no extra flags or manual kubeconfig edits are needed.

To tear down and rebuild from scratch:

```bash
k3d cluster delete kiii
k3d cluster create --config k8s/k3d-config.yaml
kubectl apply -k k8s/
```

The backend pods may restart once or twice right after applying (they can start slightly before
the database's DNS entry is ready) — that's expected and resolves itself within ~30s.

Before applying, `k8s/07-backend-deployment.yaml` and `k8s/09-frontend-deployment.yaml` must point
`image:` at the Docker Hub username the images were actually pushed to.

Manifests:
- `00-namespace.yaml` — dedicated `ciid-project` namespace
- `01/02/03/04` — db ConfigMap, Secret, headless Service, StatefulSet (PostgreSQL)
- `05/06/07/08` — backend ConfigMap, Secret, Deployment, Service
- `09/10` — frontend Deployment, Service
- `11-ingress.yaml` — routes `/api` to backend, `/` to frontend

To reach it through the Ingress from a browser, add `127.0.0.1 ciid.local` to your hosts file
(`C:\Windows\System32\drivers\etc\hosts` on Windows, needs admin rights), then open
`http://ciid.local`.
