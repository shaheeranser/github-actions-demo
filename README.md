# github-actions-demo

A demonstration of an automated CI/CD pipeline built with GitHub Actions. The pipeline tests, builds, and deploys a containerized microservices application to Azure Kubernetes Service (AKS) — including an ephemeral staging environment created per pull request.

## Application

A small e-commerce sample made of independent FastAPI services, each backed by its own PostgreSQL database, communicating through RabbitMQ:

- **product_service** (8000) — product catalogue; stores images via Azure Storage
- **order_service** (8001) — orders; communicates with product_service
- **customer_service** (8002) — customer records
- **frontend** — static HTML/JS served by Nginx (3000)

`docker-compose.yml` runs the full stack locally.

## Pipelines (`.github/workflows/`)

- **`ci-pipeline.yml`** — on pushes to `testing`: spins up PostgreSQL service containers and runs each service's pytest suite, then builds and pushes Docker images to Azure Container Registry.
- **`staging.yml`** — on pull requests to `main`: creates a temporary `staging-<run_id>` namespace on AKS, deploys the stack from `k8s/`, waits for pods and LoadBalancer IPs, runs curl smoke tests, collects logs on failure, and tears the namespace down.
- **`deployment.yml`** — manual (`workflow_dispatch`): deploys infrastructure and services to AKS, resolves backend LoadBalancer IPs, injects them into the frontend, then builds and deploys the frontend.

Secrets required: `AZURE_CREDENTIALS`, `AZURE_CONTAINER_REGISTRY`, `AKS_RESOURCE_GROUP`, `AKS_CLUSTER_NAME`.

## Kubernetes

Manifests in `k8s/` cover the services, databases, RabbitMQ, configmaps, and the frontend. Copy `k8s/secrets.example.yaml` and fill in real values before deploying.

## Local development

```bash
docker compose up --build
```

Frontend → http://localhost:3000 · Product API → http://localhost:8000 · Order API → http://localhost:8001 · Customer API → http://localhost:8002 · RabbitMQ UI → http://localhost:15672

## Notes

- Secrets are supplied through GitHub Actions secrets and Kubernetes secrets; no credentials are committed.
