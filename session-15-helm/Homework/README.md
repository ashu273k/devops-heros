# Session 15: Helm Homework

This homework focuses on hands-on practice with Helm commands, release management, and a mini project deployment in Kubernetes.

---

## Objective

The goal is to practice and document the complete lifecycle of a Helm chart:

- create a chart
- install a release
- list and inspect the release
- upgrade configuration
- verify the deployment
- roll back to a previous version
- uninstall the release

---

## Task 1: Helm Commands Practice

### 1. helm repo update

Command:

```bash
helm repo update
```

Purpose:
- Refreshes the local chart repository metadata.

Observed output:

```text
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```

---

### 2. helm install

Command:

```bash
helm install my-nginx bitnami/nginx
```

Purpose:
- Deploy an application from a Helm chart into the cluster.

Observed output:

```text
NAME: my-nginx
LAST DEPLOYED: Thu Oct  8 18:49:34 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART: nginx
CHART VERSION: 25.2.1
APP VERSION: 1.31.6
```

---

### 3. helm list

Command:

```bash
helm list
```

Purpose:
- Shows all deployed Helm releases in the current namespace.

Observed output:

```text
NAME            NAMESPACE   REVISION STATUS   CHART          APP VERSION
my-nginx        default     1        deployed nginx         1.31.6
```

---

### 4. helm status

Command:

```bash
helm status my-nginx
```

Purpose:
- Shows the current status, revision, namespace, and deployment timestamp.

Observed output:

```text
NAME: my-nginx
LAST DEPLOYED: Thu Oct  8 18:49:34 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
```

---

### 5. helm get

Commands:

```bash
helm get manifest my-nginx
helm get values my-nginx
```

Purpose:
- Displays the rendered Kubernetes manifests or chart values for a release.

This helped verify what Helm actually deployed and what configuration was being used.

---

### 6. helm upgrade

Command:

```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=3
```

Purpose:
- Updates a running release with new values or a new chart version.

Observed output:

```text
Release "my-nginx" has been upgraded.
NAME: my-nginx
LAST DEPLOYED: Thu Oct  8 18:49:34 2026
NAMESPACE: default
STATUS: deployed
REVISION: 2
```

---

### 7. helm history

Command:

```bash
helm history my-nginx
```

Purpose:
- Shows revision history for the release.

Typical output:

```text
REVISION  UPDATED                   STATUS     CHART         APP VERSION
1         Thu Oct 8 ...            deployed   nginx         1.31.6
2         Thu Oct 8 ...            deployed   nginx         1.31.6
```

---

### 8. helm rollback

Command:

```bash
helm rollback my-nginx 1
```

Purpose:
- Reverts the release to a previous revision.

Observed output:

```text
Rollback was a success.
Happy Helming!
```

---

### 9. helm uninstall

Command:

```bash
helm uninstall my-nginx
```

Purpose:
- Removes the release and its associated resources from Kubernetes.

Observed output:

```text
release "my-nginx" uninstalled
```

---

### 10. helm repo

Commands:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
```

Purpose:
- Adds and manages external Helm repositories.

---

### 11. helm search

Command:

```bash
helm search repo nginx
```

Purpose:
- Searches a repo for available charts.

Observed output:

```text
NAME                 CHART VERSION APP VERSION DESCRIPTION
bitnami/nginx        25.2.1        1.31.6      NGINX Open Source
```

---

### 12. helm create

Command:

```bash
helm create simple-chart
```

Purpose:
- Creates a starter Helm chart structure.

Generated structure:

```text
simple-chart/
  Chart.yaml
  values.yaml
  templates/
    deployment.yaml
    service.yaml
    helpers.tpl
```

---

### 13. helm template

Command:

```bash
helm template my-release simple-chart
```

Purpose:
- Renders the chart locally without installing it to Kubernetes.

This is useful to validate templates before deploying.

---

## Task 2: Complete Helm Rollback Workflow

The full rollback workflow was practiced as follows:

```bash
helm install my-release simple-chart
helm upgrade my-release simple-chart --set replicaCount=3
helm history my-release
helm rollback my-release 1
helm status my-release
kubectl get pods
kubectl get services
```

Flow:

1. Install the release
2. Upgrade the release
3. Verify the deployment
4. Upgrade again
5. Verify again
6. Roll back to previous revision
7. Verify rollback success

This demonstrates Helm’s main advantage: safe, versioned changes and quick recovery.

---

## Task 3: Mini Project

The mini project was implemented using a custom chart named `notes-chart`.

### Chart contents

```text
mini-project/
  notes-chart/
    Chart.yaml
    values.yaml
    templates/
      configmap.yaml
      deployment.yaml
      service.yaml
```

### Commands used

```bash
helm lint notes-chart
helm template notes-dev notes-chart
helm install notes-dev notes-chart
kubectl get pods
kubectl get services
kubectl get configmaps
helm upgrade notes-dev notes-chart -f notes-chart/values-prod.yaml
helm history notes-dev
helm rollback notes-dev 1
helm uninstall notes-dev
```

### What the chart does

- Creates a ConfigMap
- Deploys a containerized app using nginx
- Exposes the app via a NodePort service
- Supports upgrade and rollback

---

## Screenshots

### 1. Helm basic commands

![Helm basic commands](./01-helm-basic.png)

### 2. Helm chart creation

![Helm chart creation](./02-helm-chart.png)

### 3. Chart structure

![Chart structure](./03-chart-structure.png)

### 4. Mini project

![Mini project](./mini-project.png)

---

## Learning Summary

Helm is a package manager for Kubernetes that makes deployment repeatable and manageable. It allows teams to:

- create reusable application packages
- manage configuration with `values.yaml`
- install, upgrade, and rollback safely
- keep deployment history and recover quickly from failed changes

This homework successfully demonstrated the practical use of Helm in a real Kubernetes environment.
