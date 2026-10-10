# Mini Project: Notes App with Helm

This mini project demonstrates a complete Helm workflow for deploying a lightweight Notes application using a custom chart.

---

## Project Structure

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

The chart deploys a simple nginx-based application and exposes it through a NodePort service.

---

## Chart Files

### Chart.yaml

```yaml
apiVersion: v2
name: notes-chart
description: A simple Notes application Helm chart
type: application
version: 0.1.0
appVersion: "1.0"
```

### values.yaml

```yaml
replicaCount: 1

image:
  repository: nginx
  tag: "1.24"

service:
  port: 80
  nodePort: 30090

app:
  name: notes-app
  environment: development
```

### templates/configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  APP_NAME: {{ .Values.app.name | quote }}
  ENVIRONMENT: {{ .Values.app.environment | quote }}
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deploy
  labels:
    app: {{ .Release.Name }}
    environment: {{ .Values.app.environment }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: notes
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
          envFrom:
            - configMapRef:
                name: {{ .Release.Name }}-config
```

### templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-svc
spec:
  type: NodePort
  selector:
    app: {{ .Release.Name }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.port }}
      nodePort: {{ .Values.service.nodePort }}
```

---

## Lint and Render

```bash
helm lint notes-chart
helm template notes-dev notes-chart
```

Observed output:

```text
==> Linting notes-chart
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed
```

This confirmed that the chart syntax was valid and templates rendered successfully.

---

## Install the Release

```bash
helm install notes-dev notes-chart
```

Observed output:

```text
NAME: notes-dev
LAST DEPLOYED: Sat Oct 14 14:39:20 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
```

Verify with Kubernetes:

```bash
kubectl get pods
kubectl get services
kubectl get configmaps
```

Observed output:

```text
NAME                            READY   STATUS    RESTARTS   AGE
notes-dev-deploy-xxxxxxx        1/1     Running   0          20s
project-broken-pod              0/1     ImagePullBackOff  0   18h
troubleshooting-app-...         1/1     Running   0          18h
```

```text
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
kubernetes        ClusterIP  10.96.0.1       <none>        443/TCP     18h
notes-dev-svc     NodePort   10.111.18.147   <none>        80:30090/TCP  18s
```

---

## Upgrade the Release

To simulate a production-style update, change the app configuration values and upgrade the chart.

```bash
helm upgrade notes-dev notes-chart -f notes-chart/values-prod.yaml
```

The lab used a production override file to update:

- replica count
- app environment
- image tag
- service configuration

Observed output:

```text
Release "notes-dev" has been upgraded.
LAST DEPLOYED: Sat Oct 14 14:48:02 2026
NAMESPACE: default
STATUS: deployed
REVISION: 2
```

Verify again:

```bash
kubectl get pods
kubectl get configmaps
```

---

## Release History

```bash
helm history notes-dev
```

Typical output:

```text
REVISION  UPDATED                   STATUS      CHART          APP VERSION
1         ...                       superseded  notes-chart    1.0
2         ...                       deployed    notes-chart    1.0
```

This confirms Helm tracks each change as a revision.

---

## Rollback

```bash
helm rollback notes-dev 1
```

Expected result:

```text
Rollback was a success.
Happy Helming!
```

Then verify:

```bash
helm status notes-dev
kubectl get pods
```

This confirms the previous revision is restored successfully.

---

## Uninstall

```bash
helm uninstall notes-dev
```

Observed output:

```text
release "notes-dev" uninstalled
```

---

## Screenshot Evidence

![Mini project screenshot](../Homework/mini-project.png)

---

## Learning Outcome

This mini project demonstrates the complete Helm workflow:

1. Create chart
2. Lint and validate templates
3. Install release
4. Upgrade release with values
5. Check history
6. Roll back to previous revision
7. Uninstall cleanly

The main benefit is that Helm makes application deployment consistent, repeatable, and easy to recover from when a change fails.

```

Expected output:

```text
REVISION   STATUS      DESCRIPTION
1          superseded  Install complete
2          deployed    Upgrade complete
```

---

## Step 13: Simulate a Bad Upgrade

```bash
helm upgrade notes-dev notes-chart --set image.tag=broken-tag-does-not-exist
```

Check pods:

```bash
kubectl get pods
```

Expected output:

```text
NAME                            READY   STATUS             RESTARTS
notes-dev-deploy-xxxx           0/1     ImagePullBackOff   0
```

---

## Step 14: Rollback to Revision 2

```bash
helm rollback notes-dev 2
```

Expected output:

```text
Rollback was a success! Happy Helming!
```

Pods are healthy again:

```bash
kubectl get pods
```

```text
NAME                            READY   STATUS    RESTARTS
notes-dev-deploy-aaaa           1/1     Running   0
notes-dev-deploy-bbbb           1/1     Running   0
notes-dev-deploy-cccc           1/1     Running   0
```

---

## Step 15: Clean Up

```bash
helm uninstall notes-dev
kubectl get pods
kubectl get services
```

All resources are gone.

---

## What You Practiced

```text
[PASS] Created a Helm chart from scratch
[PASS] Used values.yaml and values-prod.yaml
[PASS] Deployed to Kubernetes with helm install
[PASS] Upgraded the release with different values
[PASS] Simulated a bad upgrade (broken image tag)
[PASS] Rolled back to a healthy revision
[PASS] Cleaned up with helm uninstall
```

---

## Reference

* **Helm best practices:** https://helm.sh/docs/chart_best_practices/
* **Helm CLI reference:** https://helm.sh/docs/helm/
