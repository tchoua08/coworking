# Coworking Analytics Service

This project deploys a Flask analytics API for the coworking service on Amazon EKS.
The API reads PostgreSQL data and exposes reports at `/api/reports/daily_usage` and `/api/reports/user_visits`.
The application code is packaged by `analytics/Dockerfile` into a Docker image tagged with semantic version `1.0.0`.
AWS CodeBuild uses `buildspec.yml` to build the image and push it to Amazon ECR repository `coworking`.
The Kubernetes deployment files are stored in `deployment/`.
Non-sensitive runtime values are stored in `deployment/configmap.yaml`.
The database password is stored separately in `deployment/secret.yaml`.
PostgreSQL runs from `deployment/postgresql-deployment.yaml` and is exposed inside the cluster by `postgresql-service`.
The analytics API runs from `deployment/coworking.yaml` and is exposed by a LoadBalancer service on port `5153`.
The API deployment includes liveness and readiness probes for `/health_check` and `/readiness_check`.
CPU and memory requests are defined so Kubernetes can schedule the workloads predictably.
To release a new version, update `IMAGE_TAG` in `buildspec.yml`, build with CodeBuild, update the image tag in `deployment/coworking.yaml`, and apply the manifests again.
Before testing the API, seed PostgreSQL with the SQL files in `db/`.
Use `kubectl get svc`, `kubectl get pods`, `kubectl describe svc postgresql-service`, and `kubectl describe deployment coworking` to validate the deployment.
CloudWatch Container Insights should be used to confirm the application logs and pod health after the pods are running.

## Standout Suggestions

The API uses modest CPU and memory requests because it is a lightweight reporting service with a small project dataset.
For this workload, a `t3.small` EKS worker node is a reasonable starting point because it provides enough burst capacity for development and demonstration traffic.
To reduce costs, keep the node count low, remove unused ECR image tags, and delete the EKS cluster when the project evaluation is complete.
