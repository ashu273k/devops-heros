# Session 21 Final Project: DevOps Capstone

## End-to-End DevOps Project

Build and document a production-style application delivery platform using the tools and practices from the course. The project must take an application from source control through testing, security scanning, container publishing, Kubernetes deployment, Helm packaging, monitoring, and GitOps reconciliation.

## Required Delivery Flow

```text
Application
    |
    v
Git and GitHub
    |
    v
GitHub Actions CI pipeline
    |-- build
    |-- unit test
    |-- SAST / SCA / secret scan
    |-- Docker build
    |-- container scan and security gate
    `-- push image to registry
		  |
		  v
	  Terraform infrastructure
		  |
		  v
	    Kubernetes cluster
		  |
		  v
		  Helm
		  |
		  v
	  Monitoring and logs
		  |
		  v
		 GitOps
```

## Final Repository Structure

The final project must be organized like this:

```text
final-devops-project/
├── application/              # Frontend, backend, tests, and migrations
├── docker/                   # Dockerfiles and Compose configuration
├── kubernetes/               # Raw Kubernetes manifests and bootstrap files
├── helm/                     # Helm chart for the application
├── terraform/                # Cloud infrastructure as code
├── .github/
│   └── workflows/            # CI/CD workflows
├── security/                 # SAST, SCA, secret, and image-scan policy
├── monitoring/               # Metrics, dashboards, alerts, and values files
├── gitops/                   # GitOps application and desired state
├── troubleshooting/          # Broken scenarios and investigation records
└── README.md
```

Equivalent filenames are acceptable when the README clearly maps each directory to a requirement.

## 1. Application

Build a useful application rather than a placeholder container. A typical implementation may contain a frontend, backend API, and database.

The application must include:

- A documented purpose and local setup instructions.
- At least four useful API endpoints, including a health endpoint.
- Persistent data or a clearly documented reason persistence is unnecessary.
- Automated unit or integration tests.
- Liveness and readiness endpoints suitable for Kubernetes probes.
- A Prometheus-compatible `/metrics` endpoint or an equivalent metrics integration.
- Useful application logs with timestamps, levels, and error context.

Document the API, configuration variables, database migrations, and test commands.

## 2. Git and GitHub

Use Git and GitHub as the source-control and collaboration platform.

Requirements:

- Meaningful commit messages.
- A `.gitignore` excluding secrets, virtual environments, build output, dependencies, and Terraform state.
- A repository history that demonstrates incremental work.
- Pull request or branch protection practices documented where available.
- No credentials, private keys, tokens, `.env` files, or cloud account secrets in the repository or its history.

## 3. Docker

Containerize every application component that needs to run in the target environment.

Requirements:

- A Dockerfile for each application image.
- A multi-stage frontend build where applicable.
- Small, pinned base images where practical.
- A non-root runtime user.
- No secrets baked into images.
- Health checks or documented Kubernetes probes.
- Local Compose setup for the application and its dependencies, where useful.

Example commands:

```bash
docker compose up --build
docker compose ps
docker compose down
```

## 4. Kubernetes

Deploy the application to a Kubernetes cluster such as EKS, kind, or minikube. Cloud deployment is required when Terraform provisions the cluster.

Include and explain:

- Namespace.
- Deployment for each application component.
- ClusterIP Service for internal communication.
- ConfigMap for non-sensitive configuration.
- Secret for sensitive configuration. Use a safe example or external secret mechanism; never commit real values.
- Ingress routing external requests to the correct service.
- HorizontalPodAutoscaler with documented CPU or memory targets.
- Liveness and readiness probes.
- Resource requests and limits.
- Persistent storage where the application requires it.

Verify with:

```bash
kubectl get all -n NAMESPACE
kubectl get configmap,secret,ingress,hpa -n NAMESPACE
kubectl describe deployment APP_NAME -n NAMESPACE
kubectl rollout status deployment/APP_NAME -n NAMESPACE
```

## 5. Helm

Package the Kubernetes application as a reusable Helm chart.

The chart must include:

```text
helm/APP_NAME/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-prod.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   └── _helpers.tpl
└── README.md
```

Use values for image repository and tag, replica count, resources, probes, ingress host, and environment configuration. Validate and deploy it with:

```bash
helm lint helm/APP_NAME
helm template APP_NAME helm/APP_NAME
helm upgrade --install APP_NAME helm/APP_NAME \
  --namespace NAMESPACE --create-namespace \
  --values helm/APP_NAME/values-dev.yaml
helm list -n NAMESPACE
```

## 6. Terraform Infrastructure

Use Terraform to provision the required cloud infrastructure. At minimum, document and provision the network and Kubernetes platform needed by the application.

The Terraform project should include:

- Provider and version constraints.
- Variables and an example variables file.
- VPC and subnets across suitable availability zones.
- Route tables and internet or NAT connectivity as required.
- Security groups with least-privilege rules.
- EKS or an equivalent managed Kubernetes cluster when using AWS.
- Worker node group or managed compute capacity.
- Outputs for cluster name, region, and connection information.
- State handling and backend assumptions.

Run and record:

```bash
terraform fmt -check -recursive
terraform init
terraform validate
terraform plan
terraform apply
terraform output
terraform destroy
```

Use `terraform plan -out=tfplan` and `terraform apply tfplan` when demonstrating a reviewed plan. Never commit `terraform.tfstate`, plan files, credentials, or real `terraform.tfvars` values. Destroy billable resources after evidence is captured unless the instructor explicitly requests that they remain.

## 7. CI/CD with GitHub Actions

Create one or more workflows under `.github/workflows/`.

The pipeline must:

1. Trigger on pull requests and pushes to the default branch.
2. Install frontend and backend dependencies.
3. Build the application.
4. Run unit and integration tests.
5. Run security checks.
6. Build Docker images.
7. Scan the images.
8. Enforce security gates.
9. Push images to GitHub Container Registry or another documented registry.
10. Tag images with the immutable commit SHA.
11. Deploy or update the Helm release after publication.
12. Wait for rollout completion and verify application health.

Use `needs` to make failed tests and scans block image publication and deployment. Registry publishing and production deployment should be limited to trusted pushes, not arbitrary pull requests.

## 8. DevSecOps

Implement and document all four security checks:

| Check | Examples | Required gate |
| --- | --- | --- |
| SAST | CodeQL, Bandit, Semgrep | Fails according to the configured policy. |
| SCA | `pip-audit`, `npm audit`, Dependency-Check | Fails for policy-defined vulnerable dependencies. |
| Secret scanning | Gitleaks, GitHub secret scanning | Fails on supported exposed credentials. |
| Container scanning | Trivy, Grype | Fails on the selected HIGH/CRITICAL threshold. |

Example image gate:

```bash
trivy image --severity HIGH,CRITICAL --exit-code 1 IMAGE:TAG
```

Record the policy, tool versions, exceptions, and remediation process in `security/`. An exception must explain the risk, owner, expiration date, and why the pipeline can continue.

## 9. Monitoring and GitOps

### Monitoring

Provide monitoring for:

- Application health.
- Request count and error rate.
- Request latency.
- CPU and memory utilization.
- Pod restarts and failed probes.
- Kubernetes events and application logs.

Prometheus and Grafana are recommended. Include dashboard or query definitions in `monitoring/` and document how to access them. The monitoring evidence must show populated data, not only installed pods.

Useful checks include:

```bash
curl http://APPLICATION/health
curl http://APPLICATION/metrics
kubectl logs deployment/APP_NAME -n NAMESPACE
kubectl top pods -n NAMESPACE
kubectl get events -n NAMESPACE --sort-by=.lastTimestamp
```

### GitOps

Use Git as the source of truth for Kubernetes desired state. Argo CD or Flux is recommended.

The GitOps flow must demonstrate:

```text
Manifest change
	|
	v
Commit and push to Git
	|
	v
GitOps controller detects revision
	|
	v
Controller reconciles Kubernetes
	|
	v
Workload reaches desired state
```

Show one Git change, such as an image tag or replica count update, traveling from commit to synchronized Kubernetes resources. If self-healing is enabled, make a temporary manual drift and show the controller restoring the Git-defined state.

## 10. Final Troubleshooting Challenge

Intentionally introduce at least three controlled faults into a copy or branch of the project. Examples include:

- Incorrect container image tag causing `ImagePullBackOff`.
- Readiness probe path or port mismatch.
- Service selector that does not match pod labels.
- Missing ConfigMap key or incorrect Secret reference.
- Invalid database connection string.
- Helm value overriding the wrong image repository.
- HPA target without available resource metrics.
- Ingress path or service port mismatch.
- Terraform security group or subnet dependency error.
- GitOps repository path or revision error.

For every scenario, document:

1. The intended symptom.
2. The commands used to investigate.
3. Relevant logs, events, and resource descriptions.
4. The root cause.
5. The corrective change.
6. Verification after the fix.
7. The preventive lesson or test that would catch it earlier.

Useful investigation commands:

```bash
kubectl get pods -n NAMESPACE
kubectl describe pod POD_NAME -n NAMESPACE
kubectl logs POD_NAME -n NAMESPACE --previous
kubectl get events -n NAMESPACE --sort-by=.lastTimestamp
kubectl get endpoints SERVICE_NAME -n NAMESPACE
kubectl describe ingress INGRESS_NAME -n NAMESPACE
helm history APP_NAME -n NAMESPACE
kubectl get application -n argocd
```

Do not leave the production path broken after completing the exercise. Store broken examples under `troubleshooting/` and restore a verified working deployment.

## Required Screenshots and Evidence

Include readable screenshots or command output for:

- Application running locally or in Kubernetes.
- Passing tests.
- Successful GitHub Actions pipeline.
- Published registry image with an SHA tag.
- Security scan results and gates.
- Terraform plan and AWS infrastructure.
- Helm release and running Kubernetes pods.
- Ingress or application health response.
- Monitoring dashboard with populated metrics.
- Logs and Kubernetes events.
- GitOps synchronized and healthy status.
- At least one troubleshooting investigation and successful fix.
- Terraform destroy or documented cleanup.

Redact credentials, tokens, private keys, kubeconfig contents, and sensitive infrastructure data before committing screenshots.

## Final README Requirements

The project README must contain:

- Project overview.
- Architecture diagram.
- Technologies used.
- Application setup.
- Docker setup.
- Kubernetes deployment.
- Helm deployment.
- Terraform infrastructure.
- CI/CD pipeline.
- DevSecOps implementation.
- Monitoring and logs/metrics.
- GitOps workflow.
- Troubleshooting records.
- Screenshots and evidence links.
- Lessons learned.
- Cleanup instructions.

## Acceptance Checklist

- [ ] Application builds, starts, and has automated tests.
- [ ] GitHub repository and commit history are documented.
- [ ] Docker images build and run as non-root where practical.
- [ ] Kubernetes Deployment, Service, ConfigMap, Secret, Ingress, HPA, probes, and required storage are present.
- [ ] Helm installs and upgrades the application successfully.
- [ ] Terraform provisions the required infrastructure and state workflow is documented.
- [ ] CI builds, tests, scans, publishes, and deploys using GitHub Actions.
- [ ] SAST, SCA, secret scanning, and image scanning are enforced by security gates.
- [ ] Monitoring displays application and Kubernetes signals.
- [ ] GitOps reconciles a committed desired-state change.
- [ ] Multiple troubleshooting faults are investigated and fixed.
- [ ] Screenshots, successful output, lessons learned, and cleanup instructions are included.
