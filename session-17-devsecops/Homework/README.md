# Session 17 Homework: Complete CI/CD and DevSecOps

## DevSecOps Demo Project

Build a complete CI/CD pipeline that continuously tests, scans, packages, and deploys a small application. Security checks must control delivery: a failed required check must prevent the image from being pushed or deployed.

## Expected Flow

```text
Code
	|
	v
Build
	|
	v
Unit Test
	|
	v
SAST
	|
	v
SCA
	|
	v
Secret Scan
	|
	v
Docker Build
	|
	v
Container Image Scan
	|
	v
Security Gate
	|-- fail --> Stop pipeline
	|
	v
Push Image to Container Registry
	|
	v
Deploy to Kubernetes
	|
	v
Verify rollout and application health
```

## Project Requirements

### Application

Create a small web application with:

- A home page or useful API endpoint.
- A health endpoint such as `/health`.
- Automated unit tests for successful and invalid requests.
- A non-root container runtime where practical.

### CI/CD

The GitHub Actions workflow must:

1. Trigger on pull requests and pushes to the default branch.
2. Check out the source code and install dependencies.
3. Build the application.
4. Run the unit test suite.
5. Run SAST.
6. Run software composition analysis (SCA).
7. Scan the repository for secrets.
8. Build and tag a Docker image with the commit SHA.
9. Scan the image for vulnerabilities.
10. Enforce the security gate.
11. Push the image to a registry only after all required gates pass.
12. Deploy the exact image tag to Kubernetes only after a successful push.
13. Wait for the rollout and check the deployed application.

Use job dependencies such as `needs` to make the order and gates explicit. Pull requests may run build and scan jobs, but registry publishing and deployment should be limited to trusted pushes to the default branch.

### Security Tooling

Use and configure at least one tool for each category:

| Category | Examples | Required behavior |
| --- | --- | --- |
| SAST | CodeQL, Bandit, Semgrep | Scan application source and fail on the configured policy threshold. |
| SCA | `pip-audit`, `npm audit`, OWASP Dependency-Check | Scan direct and transitive dependencies. |
| Secret scanning | Gitleaks, GitHub secret scanning | Fail when a supported credential or token is detected. |
| Image scanning | Trivy, Grype | Scan the built image and fail on the chosen severity threshold. |

Document the policy in the repository. For example, an image scan may use:

```bash
trivy image --severity HIGH,CRITICAL --exit-code 1 IMAGE:TAG
```

Do not place real credentials in the repository. Store registry and cluster credentials in GitHub Actions secrets or use short-lived identity-based authentication.

### Kubernetes

Include manifests for:

- A `Deployment` with the published image tag.
- A `Service` for application access.
- Readiness and liveness probes where the application supports them.
- Resource requests and limits.
- A non-root security context where compatible with the image.

The workflow must verify the deployment with a command such as:

```bash
kubectl rollout status deployment/APP_NAME --timeout=120s
```

## Required Deliverables

Submit the following in this `Homework/` directory:

```text
Homework/
├── README.md
├── app/                         # Application source
├── tests/                       # Unit tests
├── Dockerfile
├── .dockerignore
├── requirements.txt             # Or the package manager equivalent
├── .github/workflows/
│   └── devsecops.yml
├── security/
│   ├── bandit.yaml              # Or SAST configuration
│   ├── gitleaks.toml            # Or secret-scanning configuration
│   └── trivy.yaml               # Or documented image-scan policy
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── evidence/
│   └── successful-pipeline-output.md
└── screenshots/
		├── pipeline-success.png
		├── security-scans.png
		└── kubernetes-deployment.png
```

Equivalent filenames and tools are acceptable when the README clearly maps them to the requirements.

## Evidence Requirements

Include evidence of a successful run, not only workflow YAML:

- GitHub Actions run showing the ordered jobs and successful gates.
- Unit-test output and coverage, if configured.
- SAST, SCA, secret-scan, and image-scan results.
- Registry image tag, preferably the commit SHA.
- `kubectl get pods` and `kubectl get service` output.
- Successful rollout status and an application health response.

Screenshots must be readable and should include the workflow run, security results, and deployed Kubernetes resources. Redact usernames, tokens, kubeconfig contents, and other sensitive values.

## Suggested Local Checks

Run the same checks locally where the tools are installed:

```bash
python -m pytest
bandit -r app
pip-audit -r requirements.txt
gitleaks detect --source . --redact
docker build -t devsecops-demo:local .
trivy image --severity HIGH,CRITICAL --exit-code 1 devsecops-demo:local
kubectl apply --dry-run=client -f k8s/
```

Record the successful output in `evidence/successful-pipeline-output.md`. If a tool is unavailable locally, the GitHub Actions run is the source of truth; document the limitation rather than skipping the check.

## Acceptance Checklist

- [ ] Application starts locally and in a container.
- [ ] Unit tests pass.
- [ ] SAST runs and is enforced.
- [ ] SCA runs and is enforced.
- [ ] Secret scanning runs and is enforced.
- [ ] Docker image builds successfully.
- [ ] Container image scanning runs and is enforced.
- [ ] Security failures stop image publication.
- [ ] Image is pushed only after the gates pass.
- [ ] Kubernetes manifests deploy the published image.
- [ ] Rollout and health checks pass.
- [ ] Successful output and screenshots are included.
- [ ] README explains setup, secrets, pipeline stages, security policy, and deployment.
