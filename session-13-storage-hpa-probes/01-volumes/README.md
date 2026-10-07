# Kubernetes Volumes

Kubernetes volumes provide storage for containers. Unlike a container's
filesystem, some volumes can survive container restarts or Pod replacement.

## 1. emptyDir

`emptyDir` creates an empty directory when a Pod starts.

It is shared by containers in the same Pod and exists for the lifetime of that
Pod. When the Pod is deleted, the data is deleted.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
    - name: app
      image: nginx:1.27
      volumeMounts:
        - name: temporary-storage
          mountPath: /data
  volumes:
    - name: temporary-storage
      emptyDir: {}
```

Test it:

```bash
kubectl apply -f ../01-volumes/emptydir-pod.yaml
kubectl exec emptydir-demo -- sh -c \
  'echo "Temporary data" > /data/message.txt'
kubectl exec emptydir-demo -- cat /data/message.txt
```

Delete and recreate the Pod:

```bash
kubectl delete pod emptydir-demo
kubectl apply -f ../01-volumes/emptydir-pod.yaml
kubectl exec emptydir-demo -- cat /data/message.txt
```

The file no longer exists because `emptyDir` storage belongs to the deleted
Pod.

## 2. hostPath

`hostPath` mounts a directory from the Kubernetes node into a Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
    - name: app
      image: nginx:1.27
      volumeMounts:
        - name: node-storage
          mountPath: /data
  volumes:
    - name: node-storage
      hostPath:
        path: /tmp/student-data
        type: DirectoryOrCreate
```

`hostPath` is useful for local testing and node-level applications. It is
usually avoided for production application storage because the data is tied to
one node.

## 3. PersistentVolume

A PersistentVolume, or PV, is storage made available to the Kubernetes
cluster. It can be created manually by an administrator or automatically by a
StorageClass.

Example:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: student-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/student-persistent-data
```

Check PersistentVolumes:

```bash
kubectl get pv
kubectl describe pv student-pv
```

## 4. PersistentVolumeClaim

A PersistentVolumeClaim, or PVC, is a request for storage made by a user or
application.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: student-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

Apply and inspect the claim:

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
kubectl describe pvc student-pvc
```

A PVC must be bound to a suitable PV before a Pod can use it.

A Pod mounts the PVC like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo
spec:
  containers:
    - name: app
      image: nginx:1.27
      volumeMounts:
        - name: application-data
          mountPath: /data
  volumes:
    - name: application-data
      persistentVolumeClaim:
        claimName: student-pvc
```

## 5. StorageClass

A StorageClass defines how storage should be provisioned.

```bash
kubectl get storageclass
kubectl describe storageclass
```

Typical Minikube output includes a `standard` StorageClass. Its provisioner
creates storage using the cluster's local storage mechanism.

A PVC can request a specific StorageClass:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

## 6. Dynamic Provisioning

Dynamic provisioning automatically creates a PV when a PVC requests storage.

Apply the claim:

```bash
kubectl apply -f ../03-storageclass/pvc.yaml
kubectl get pvc
kubectl get pv
```

Expected result:

```text
NAME          STATUS   VOLUME                                     CAPACITY
dynamic-pvc   Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   500Mi
```

The storage flow is:

```text
PVC -> StorageClass -> provisioner -> dynamically-created PV
```

This avoids manually creating a PV for every application.

## 7. Access Modes

Common access modes include:

- `ReadWriteOnce` (`RWO`): mounted as read-write by one node.
- `ReadOnlyMany` (`ROX`): mounted read-only by many nodes.
- `ReadWriteMany` (`RWX`): mounted read-write by many nodes.

The supported access modes depend on the storage provider.

## 8. Comparison

| Volume | Lifetime | Typical Use |
|---|---|---|
| `emptyDir` | Pod lifetime | Temporary files and shared scratch space |
| `hostPath` | Node lifetime | Local testing and node-level workloads |
| PV | Cluster-managed | Persistent application storage |
| PVC | Application request | Requesting storage without knowing its implementation |
| StorageClass | Cluster configuration | Defining dynamic storage provisioning |

## 9. Important Commands

```bash
kubectl get pods
kubectl get pv
kubectl get pvc
kubectl get storageclass
kubectl describe pvc <claim-name>
kubectl describe pv <volume-name>
kubectl exec -it <pod-name> -- sh
```

## 10. Key Learnings

- `emptyDir` is temporary and disappears when the Pod is deleted.
- `hostPath` uses storage from a specific Kubernetes node.
- A PV represents available persistent storage.
- A PVC requests storage for an application.
- A StorageClass defines the storage provisioning method.
- Dynamic provisioning automatically creates PVs for PVCs.
- Persistent storage allows application data to survive Pod replacement.
