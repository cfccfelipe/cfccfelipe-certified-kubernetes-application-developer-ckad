# State Persistence - Essential Commands

## PersistentVolume (PV)

```bash
# Get PVs (cluster-scoped)
kubectl get persistentvolumes
kubectl get pv  # Short form

# Describe PV
kubectl describe pv <pv-name>

# Get PV status
kubectl get pv <pv-name> -o jsonpath='{.status.phase}'  # Available, Bound, Released, Failed

# Get PV capacity
kubectl get pv <pv-name> -o jsonpath='{.spec.capacity.storage}'

# Create PV from YAML
kubectl apply -f pv.yaml

# Delete PV
kubectl delete pv <pv-name>

# Get all PVs sorted by capacity
kubectl get pv --sort-by=.spec.capacity.storage
```

## PersistentVolumeClaim (PVC)

```bash
# Get PVCs (namespace-scoped)
kubectl get persistentvolumeclaims
kubectl get pvc  # Short form

# Describe PVC
kubectl describe pvc <pvc-name>

# Get PVC status
kubectl get pvc <pvc-name> -o jsonpath='{.status.phase}'  # Pending, Bound, Lost

# Check which PV is bound
kubectl get pvc <pvc-name> -o jsonpath='{.spec.volumeName}'

# Create PVC
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

# Delete PVC
kubectl delete pvc <pvc-name>

# Check PVC usage by pods
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.volumes[*].persistentVolumeClaim.claimName}{"\n"}{end}'
```

## StorageClass

```bash
# Get StorageClasses (cluster-scoped)
kubectl get storageclasses
kubectl get sc  # Short form

# Describe StorageClass
kubectl describe sc <sc-name>

# Get default StorageClass
kubectl get sc -o jsonpath='{.items[?(@.metadata.annotations.storageclass\.kubernetes\.io/is-default-class=="true")].metadata.name}'

# Set default StorageClass
kubectl patch storageclass <sc-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# Create StorageClass (example)
kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
volumeBindingMode: WaitForFirstConsumer
EOF
```

## Using Volumes in Pods

```bash
# Create pod with PVC
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
# Edit pod.yaml to add:
# spec:
#   volumes:
#   - name: storage
#     persistentVolumeClaim:
#       claimName: my-pvc
#   containers:
#   - name: nginx
#     volumeMounts:
#     - name: storage
#       mountPath: /data

kubectl apply -f pod.yaml

# Verify volume is mounted
kubectl exec <pod> -- df -h /data
kubectl exec <pod> -- ls -la /data
```

## EmptyDir Volumes

```bash
# Create pod with emptyDir
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: cache
      mountPath: /cache
  volumes:
  - name: cache
    emptyDir: {}
EOF

# EmptyDir in memory
# volumes:
# - name: cache
#   emptyDir:
#     medium: Memory
#     sizeLimit: 128Mi

# Verify emptyDir
kubectl exec emptydir-pod -- ls -la /cache
kubectl exec emptydir-pod -- touch /cache/test.txt
kubectl delete pod emptydir-pod  # Data lost on pod deletion
```

## ConfigMap and Secret Volumes

```bash
# Mount ConfigMap as volume
kubectl create configmap app-config --from-literal=key=value

kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'cat /config/key && sleep 3600']
    volumeMounts:
    - name: config
      mountPath: /config
  volumes:
  - name: config
    configMap:
      name: app-config
EOF

# Mount Secret as volume
kubectl create secret generic app-secret --from-literal=password=secret123

kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'cat /secrets/password && sleep 3600']
    volumeMounts:
    - name: secrets
      mountPath: /secrets
      readOnly: true
  volumes:
  - name: secrets
    secret:
      secretName: app-secret
      defaultMode: 0400
EOF

# Verify files
kubectl exec configmap-volume-pod -- ls -la /config
kubectl exec secret-volume-pod -- ls -la /secrets
```

## StatefulSets

```bash
# Create StatefulSet
kubectl apply -f statefulset.yaml

# Get StatefulSets
kubectl get statefulsets
kubectl get sts  # Short form

# Describe StatefulSet
kubectl describe sts <sts-name>

# Get pods from StatefulSet
kubectl get pods -l app=<sts-label>

# Scale StatefulSet
kubectl scale sts <sts-name> --replicas=5

# Delete StatefulSet (keep pods)
kubectl delete sts <sts-name> --cascade=orphan

# Delete StatefulSet and pods
kubectl delete sts <sts-name>

# Rollout restart StatefulSet
kubectl rollout restart sts <sts-name>

# Check StatefulSet status
kubectl get sts <sts-name> -o jsonpath='{.status.replicas}{" desired, "}{.status.readyReplicas}{" ready"}'
```

## StatefulSet Pod Management

```bash
# StatefulSet pods have stable names: <sts-name>-0, <sts-name>-1, etc.

# Access specific pod
kubectl exec -it <sts-name>-0 -- sh

# View PVC for StatefulSet pod
kubectl get pvc -l app=<sts-label>

# Each pod has its own PVC: <volumeClaimTemplate-name>-<pod-name>

# Delete pod (will be recreated with same PVC)
kubectl delete pod <sts-name>-0

# Scale down (PVCs remain)
kubectl scale sts <sts-name> --replicas=0

# Scale up (pods reattach to existing PVCs)
kubectl scale sts <sts-name> --replicas=3
```

## Volume Operations

```bash
# Check volume mounts in pod
kubectl get pod <pod> -o jsonpath='{.spec.volumes}'
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].volumeMounts}'

# Verify volume is mounted
kubectl exec <pod> -- df -h
kubectl exec <pod> -- mount | grep <mount-path>

# Check volume usage
kubectl exec <pod> -- du -sh /data

# Write data to volume
kubectl exec <pod> -- sh -c 'echo "test data" > /data/file.txt'

# Read data from volume
kubectl exec <pod> -- cat /data/file.txt

# List files in volume
kubectl exec <pod> -- ls -lah /data
```

## Expanding PVCs

```bash
# Check if StorageClass allows expansion
kubectl get sc <sc-name> -o jsonpath='{.allowVolumeExpansion}'

# Expand PVC (edit size)
kubectl edit pvc <pvc-name>
# Change spec.resources.requests.storage to larger value

# OR patch
kubectl patch pvc <pvc-name> -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'

# Check expansion status
kubectl get pvc <pvc-name>
kubectl describe pvc <pvc-name> | grep -A 5 "Conditions:"

# For some volume types, pod restart needed
kubectl delete pod <pod-using-pvc>
kubectl get pod <new-pod> -o jsonpath='{.spec.volumes[*].persistentVolumeClaim}'
```

## Debugging Storage Issues

```bash
# PVC stuck in Pending
kubectl describe pvc <pvc-name>
kubectl get events --field-selector involvedObject.name=<pvc-name>
kubectl get pv  # Check if matching PV available

# No StorageClass found
kubectl get sc

# PV not binding to PVC
kubectl get pv <pv-name> -o jsonpath='{.spec.capacity.storage}'
kubectl get pvc <pvc-name> -o jsonpath='{.spec.resources.requests.storage}'
kubectl get pv <pv-name> -o jsonpath='{.spec.accessModes}'
kubectl get pvc <pvc-name> -o jsonpath='{.spec.accessModes}'

# Pod can't mount volume
kubectl describe pod <pod> | grep -A 10 "Events:"
kubectl describe pod <pod> | grep -A 5 "Volumes:"

# Volume mount permission issues
kubectl exec <pod> -- ls -la /mount-path
kubectl get pod <pod> -o jsonpath='{.spec.securityContext}'
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].securityContext}'

# Data not persisting
kubectl get pvc <pvc-name> -o jsonpath='{.spec.volumeName}'
kubectl get pv <pv-name> -o jsonpath='{.spec.persistentVolumeReclaimPolicy}'
```

## Testing Persistence

```bash
# Create pod with PVC and write data
kubectl run test --image=busybox --restart=Never -- sh -c 'echo "test" > /data/file.txt && sleep 3600'
# (Need to add PVC volume in YAML)

# Write data
kubectl exec test -- sh -c 'echo "persistent data" > /data/test.txt'

# Verify data
kubectl exec test -- cat /data/test.txt

# Delete pod
kubectl delete pod test

# Create new pod with same PVC
kubectl run test2 --image=busybox --restart=Never -- sleep 3600
# (Use same PVC)

# Verify data persists
kubectl exec test2 -- cat /data/test.txt
```

## Common Patterns

### Pattern 1: Create PVC and use in pod
```bash
# Create PVC
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

# Wait for binding
kubectl get pvc my-pvc -w

# Use in pod
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
# Edit to add volume and volumeMount
kubectl apply -f pod.yaml
```

### Pattern 2: StatefulSet with persistent storage
```bash
# StatefulSet automatically creates PVCs per pod
kubectl apply -f statefulset-with-storage.yaml

# Check PVCs created
kubectl get pvc

# Scale and verify PVCs
kubectl scale sts web --replicas=3
kubectl get pvc -l app=web
```

### Pattern 3: Shared volume between containers
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: shared-vol
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'while true; do date >> /shared/log.txt; sleep 5; done']
    volumeMounts:
    - name: shared
      mountPath: /shared
  - name: reader
    image: busybox
    command: ['sh', '-c', 'tail -f /shared/log.txt']
    volumeMounts:
    - name: shared
      mountPath: /shared
  volumes:
  - name: shared
    emptyDir: {}
EOF

kubectl logs shared-vol -c reader -f
```

## Remember

- **PV** - cluster-scoped storage resource
- **PVC** - namespace-scoped storage request
- **StorageClass** - dynamic provisioning
- **Access Modes:**
  - **ReadWriteOnce (RWO)** - single node read-write
  - **ReadOnlyMany (ROX)** - multiple nodes read-only
  - **ReadWriteMany (RWX)** - multiple nodes read-write
- **Reclaim Policies:**
  - **Retain** - manual cleanup
  - **Delete** - auto-delete
  - **Recycle** - deprecated
- **emptyDir** - pod lifetime only
- **PVC binding** - matches size, access mode, storage class
- **StatefulSet PVCs** - stable, persistent per pod
- **Volume expansion** - StorageClass must allow it
- **Pod must be deleted/restarted** for some expansions
