# Coworking Analytics Deployment

This project packages the coworking analytics Flask API as a Docker image and deploys it to Kubernetes on Amazon EKS.
The application reads activity data from PostgreSQL and exposes reports at `/api/reports/daily_usage` and `/api/reports/user_visits`.
The Docker image is built from `analytics/Dockerfile`, tagged with semantic version `1.0.0`, and published to Amazon ECR by CodeBuild using `buildspec.yml`.
Runtime configuration is split between `deployment/configmap.yaml` for non-sensitive values and `deployment/secret.yaml` for the database password.
PostgreSQL is deployed with `deployment/postgresql-deployment.yaml`, `deployment/postgresql-service.yaml`, and persistent storage definitions in `deployment/postgresql-pvc.yaml` and `deployment/postgresql-pv.yaml`.
The analytics API is deployed by `deployment/coworking.yaml` as a LoadBalancer service plus one Kubernetes deployment.
After changing application code, publish a new release by incrementing `IMAGE_TAG` in `buildspec.yml`, running the CodeBuild project, and updating the image tag in `deployment/coworking.yaml`.
Apply database resources before the analytics deployment so the readiness probe can connect to the `tokens` table.
Seed the database through a temporary `kubectl port-forward svc/postgresql-service 5433:5432` session and run the SQL files in `db/`.
Use `kubectl get pods`, `kubectl get svc`, `kubectl describe svc postgresql-service`, and `kubectl describe deployment coworking` to verify the deployment.
CloudWatch Container Insights should be enabled on the EKS cluster so pod logs and health signals are visible during operation.
For this workload, `t3.small` worker nodes are sufficient because the API is lightweight and PostgreSQL is only used for a small project dataset.
The Kubernetes manifests include conservative CPU and memory requests so the scheduler can place pods predictably.
To reduce cost, run a single-node development cluster, avoid oversized worker instances, delete the EKS cluster after evaluation, and keep only required ECR image tags.

## Standout Suggestions

The deployment requests 100m CPU and 128Mi memory for the API, with 500m CPU and 512Mi memory limits to prevent a single pod from consuming the node.
The PostgreSQL pod requests 100m CPU and 256Mi memory, which is reasonable for the small seeded dataset and can be increased if query volume grows.
A cost-conscious production setup would start with `t3.small` or a similar burstable instance and move to larger nodes only after CloudWatch metrics show sustained pressure.
