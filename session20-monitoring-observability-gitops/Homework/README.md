# Session 20 Homework: Monitoring, Observability, and GitOps

Build a small, reproducible platform demo that shows how a team monitors an application, investigates its behavior through observability signals, and delivers Kubernetes changes through GitOps.

## Learning Goals

By completing this homework, you should be able to:

- Measure CPU, memory, request, and application health signals.
- Collect and inspect application and Kubernetes logs.
- Define an alert with a clear threshold and response.
- Explain the difference between monitoring and observability.
- Describe metrics, logs, and traces and when to use each.
- Store Kubernetes desired state in Git.
- Use a GitOps controller to reconcile Git and Kubernetes.
- Demonstrate that a Git change causes a controlled deployment change.

## Task 1: Monitoring Demo

Create a small application and instrument it with monitoring signals. The demo may run locally with Docker Compose or in a local Kubernetes cluster such as `kind` or `minikube`.

### Required Signals

#### Metrics

Expose or collect at least:

- CPU utilization.
- Memory utilization.
- Request count.
- Request error count or error rate.
- Request latency.
- Application health state.

Prometheus-format application metrics at `/metrics` are recommended. Kubernetes resource metrics may be collected with Metrics Server, `kubectl top`, or an equivalent tool.

#### Logs

Generate structured or consistently formatted application logs that include:

- Timestamp.
- Log level.
- Event or message.
- Request or correlation identifier when available.
- Error details for failed requests.

Show how to inspect application and Kubernetes logs with commands such as:

```bash
kubectl logs deployment/APP_NAME -n NAMESPACE
kubectl describe pod POD_NAME -n NAMESPACE
```

#### Alerts

Define at least three alerts with documented thresholds and actions. For example:

| Alert | Example condition | Response |
| --- | --- | --- |
| High CPU | CPU usage above 80% for 5 minutes | Inspect traffic, throttling, and scaling. |
| High memory | Memory usage above 80% for 5 minutes | Inspect leaks, limits, and recent releases. |
| Application unhealthy | Health endpoint fails for 2 consecutive checks | Check pod status, logs, dependencies, and rollout state. |

The exact thresholds may differ, but each alert must state what it detects, how long it must persist, and what a responder should investigate.

### Suggested Monitoring Stack

Use the following tools or document equivalent choices:

- Prometheus for metrics and PromQL queries.
- Grafana for dashboards.
- Alertmanager or the selected monitoring platform for alert routing.
- `kubectl logs`, Loki, or an equivalent log system for logs.
- Kubernetes probes for application health.

The dashboard should show the application health, request rate, error rate, latency, CPU, and memory. Include a screenshot with visible data rather than an empty dashboard.

## Task 2: Observability Documentation

Create a document explaining the three major observability pillars and how they work together.

### Metrics

Metrics are numeric measurements recorded over time. They are efficient for dashboards, alerting, and trend analysis.

Examples:

- CPU and memory utilization.
- Request rate.
- Error rate.
- P50, P95, and P99 latency.
- Pod restarts.
- Queue depth.

Metrics answer questions such as: **How much? How often? Is the system getting worse?**

### Logs

Logs are timestamped events emitted by applications and infrastructure. They provide detailed context about what happened.

Examples:

- A request failed validation.
- A dependency timed out.
- A deployment started or completed.
- A pod was evicted.

Logs answer: **What happened and what context explains it?**

### Traces

Traces follow one request across services. A trace is made of spans, where each span represents work performed by a service, database, queue, or external dependency.

Document:

- Trace IDs and span IDs.
- Parent-child span relationships.
- Service boundaries.
- Latency contribution from each span.
- Error propagation across a request.

OpenTelemetry is recommended for instrumentation, with Jaeger, Tempo, or another compatible backend for visualization.

### Why Observability Is Required

Monitoring can identify a known failure condition, such as high CPU or an unavailable endpoint. Observability helps engineers investigate unfamiliar failures by connecting metrics, logs, and traces with shared labels or correlation IDs.

Document one example incident using the three signals:

```text
Metric: error rate increases
	-> Log: payment dependency timeout appears
	-> Trace: most request time is spent in the payment span
	-> Action: investigate the dependency and roll back or fail over
```

### Common Tools

Compare the tools used in the project with alternatives:

| Signal or function | Example tools |
| --- | --- |
| Metrics | Prometheus, CloudWatch, Datadog, VictoriaMetrics |
| Dashboards | Grafana, CloudWatch dashboards |
| Logs | Loki, Elasticsearch/OpenSearch, CloudWatch Logs |
| Traces | OpenTelemetry, Jaeger, Tempo, Zipkin |
| Alerting | Alertmanager, PagerDuty, Grafana Alerting |

### Kubernetes Observability

Explain how to observe:

- Nodes, pods, deployments, and services.
- Container CPU and memory usage.
- Pod restarts and failed probes.
- Kubernetes events.
- Application logs from replicas.
- Control-plane or cluster signals where available.

Useful commands include:

```bash
kubectl get pods -A
kubectl top pods -n NAMESPACE
kubectl top nodes
kubectl get events -n NAMESPACE --sort-by=.lastTimestamp
kubectl describe deployment APP_NAME -n NAMESPACE
```

## Task 3: GitOps Demo

Use Git as the source of truth for Kubernetes desired state. Argo CD is recommended, but Flux or an equivalent GitOps controller is acceptable.

### GitOps Principles

Document these principles:

- **Git is the source of truth:** desired Kubernetes configuration is versioned in Git.
- **Declarative configuration:** files describe the desired state, not a sequence of imperative commands.
- **Continuous reconciliation:** the controller compares Git with the cluster and acts on drift.
- **Auditable change:** commits, reviews, and history explain who changed desired state and why.
- **Repeatable delivery:** the same revision can be synchronized into another cluster or namespace.

### Required GitOps Workflow

Demonstrate this flow:

```text
Developer changes Kubernetes manifest
							|
							v
				Commit and push to Git
							|
							v
			 GitOps controller detects change
							|
							v
			 Controller reconciles Kubernetes
							|
							v
				Application reaches desired state
```

The Git repository should contain at least:

```text
gitops-repo/
└── app/
		├── namespace.yaml
		├── deployment.yaml
		└── service.yaml
```

The controller application definition may live outside the watched workload path. Configure automated synchronization, pruning, and self-healing where appropriate for the local demonstration.

### GitOps Demonstration Steps

1. Create a local cluster and install the GitOps controller.
2. Commit the application manifests to a Git repository.
3. Register the repository and manifest path with the controller.
4. Verify the application is synchronized and healthy.
5. Change a declarative value in Git, such as the replica count or image tag.
6. Commit and push the change.
7. Show the controller detecting and synchronizing the new revision.
8. Verify the Kubernetes workload matches Git.
9. Make a temporary manual change with `kubectl` and demonstrate self-healing, if enabled.

Do not make direct `kubectl` changes the normal deployment process. The intended change path is Git commit, review, synchronization, and verification.

## Suggested Project Structure

```text
Homework/
├── README.md
├── monitoring-demo/
│   ├── app/
│   ├── docker-compose.yml
│   ├── prometheus.yml
│   ├── grafana/
│   └── alerts/
├── observability/
│   └── README.md
├── gitops-demo/
│   ├── app/
│   │   ├── namespace.yaml
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── argocd-application.yaml
└── screenshots/
		├── monitoring-dashboard.png
		├── alerts.png
		├── kubernetes-observability.png
		└── gitops-sync.png
```

Equivalent layouts are acceptable when the README clearly identifies the monitoring demo, observability documentation, and GitOps demo.

## Recommended Verification Commands

Application and Kubernetes:

```bash
kubectl get nodes
kubectl get all -n NAMESPACE
kubectl get events -n NAMESPACE --sort-by=.lastTimestamp
kubectl logs deployment/APP_NAME -n NAMESPACE
kubectl port-forward service/APP_NAME 8080:80 -n NAMESPACE
```

Metrics and health:

```bash
curl http://localhost:8080/health
curl http://localhost:8080/metrics
```

GitOps:

```bash
kubectl get applications -n argocd
kubectl describe application APP_NAME -n argocd
git log --oneline -- app/
```

Use the commands appropriate to the selected tools and record the actual output in the evidence directory.

## Required Deliverables

- Monitoring demo with metrics, logs, alerts, CPU, memory, and application health.
- Observability documentation covering metrics, logs, traces, tools, and Kubernetes.
- GitOps demo using Git as the desired-state source.
- Kubernetes manifests and GitOps controller configuration.
- Screenshots showing dashboards, alerts, Kubernetes signals, and GitOps synchronization.
- README documenting setup, workflow, queries, alerts, troubleshooting, and cleanup.

## Evidence Requirements

Include readable screenshots or terminal output showing:

1. Monitoring dashboard with application and resource data.
2. A successful health check and metrics endpoint response.
3. Application logs and Kubernetes events.
4. At least one alert in a normal or intentionally triggered state.
5. GitOps application synchronized and healthy.
6. A Git commit changing the deployment and the resulting Kubernetes reconciliation.
7. Self-healing or drift correction, if enabled.

Redact tokens, passwords, private repository credentials, cluster credentials, and other secrets before committing screenshots.

## Cleanup

Stop local monitoring components according to the selected tool:

```bash
docker compose down
```

Remove the GitOps demo resources and local cluster when finished:

```bash
kubectl delete -f gitops-demo/argocd-application.yaml
kind delete cluster --name session20
```

Use the cleanup commands that match the tools and cluster used in your implementation.

## Acceptance Checklist

- [ ] CPU utilization is measured and displayed.
- [ ] Memory utilization is measured and displayed.
- [ ] Application health is checked with a health endpoint or probe.
- [ ] Application and Kubernetes logs can be inspected.
- [ ] At least three alerts are defined with thresholds and responses.
- [ ] Metrics, logs, and traces are explained.
- [ ] Observability tools and Kubernetes observability are documented.
- [ ] Git is used as the source of truth for Kubernetes desired state.
- [ ] A GitOps controller synchronizes the application.
- [ ] A Git change produces a verified Kubernetes change.
- [ ] Screenshots and command output are included.
- [ ] Cleanup steps are documented and tested.
