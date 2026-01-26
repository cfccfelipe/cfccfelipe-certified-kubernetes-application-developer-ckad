# State Persistence - Practice Questions

## Question 1: EmptyDir Volume

**Scenario:** Create a pod with an emptyDir volume shared between containers.

**Tasks:**
1. Create a pod named `shared-data` with two containers
2. First container writes data to `/data/output.txt` every 5 seconds
3. Second container reads from `/data/output.txt`
4. Both containers mount the same emptyDir volume
5. Verify data sharing works
6. Delete the pod and observe that data is lost

**Solution:** See `solution-01.md`

---

## Question 2: HostPath Volume

**Scenario:** Mount a directory from the host node into a pod.

**Tasks:**
1. Create a pod that mounts `/var/log` from the host
2. Container should read and display host logs
3. Create another pod that writes to host path
4. Verify data persists after pod deletion
5. Document security considerations of hostPath

**Files:** `hostpath-pod.yaml`
**Solution:** See `solution-02.md`

---

## Question 3: PersistentVolume and PersistentVolumeClaim

**Scenario:** Create persistent storage that survives pod deletion.

**Tasks:**
1. Create a PersistentVolume (PV) with:
   - Capacity: `1Gi`
   - Access mode: `ReadWriteOnce`
   - Storage class: `manual`
   - Host path: `/mnt/data`
2. Create a PersistentVolumeClaim (PVC) requesting `500Mi`
3. Create a pod using the PVC
4. Write data to the volume
5. Delete pod and recreate - verify data persists
6. Check PV and PVC status

**Files:** `pv-pvc-setup.yaml`
**Solution:** See `solution-03.md`

---

## Question 4: Dynamic Provisioning with StorageClass

**Scenario:** Use StorageClass for dynamic volume provisioning.

**Tasks:**
1. Check available StorageClasses
2. Create a PVC using the default StorageClass
3. Observe automatic PV creation
4. Create a StatefulSet using dynamic PVCs
5. Scale the StatefulSet and observe PVC creation per pod
6. Verify each pod has its own persistent storage

**Files:** `statefulset-storage.yaml`
**Solution:** See `solution-04.md`

---

## Question 5: ConfigMap and Secret as Volumes

**Scenario:** Mount ConfigMaps and Secrets as volumes instead of environment variables.

**Tasks:**
1. Create a ConfigMap with multiple configuration files
2. Create a Secret with credentials
3. Create a pod that mounts:
   - ConfigMap at `/etc/config`
   - Secret at `/etc/secrets`
4. Verify files are created in the pod
5. Update ConfigMap and observe automatic file updates (may have delay)
6. Set proper file permissions for secrets

**Files:** `config-secret-volumes.yaml`
**Solution:** See `solution-05.md`

---

## Question 6: StatefulSet with Persistent Storage

**Scenario:** Deploy a StatefulSet with stable network identity and persistent storage.

**Tasks:**
1. Create a StatefulSet named `web` with 3 replicas
2. Each pod should have a PVC requesting `1Gi`
3. Create a headless service for stable network identity
4. Write unique data to each pod's volume
5. Delete a pod and verify it comes back with same storage
6. Scale down and scale up - verify storage persists

**Files:** `statefulset-demo.yaml`
**Solution:** See `solution-06.md`

---

## Question 7: Multi-Container Pod with Shared Volume

**Scenario:** Multiple containers need to share and process data.

**Tasks:**
1. Create a pod with three containers:
   - Generator: Writes random data to `/shared/input.txt`
   - Processor: Reads from input, writes to `/shared/output.txt`
   - Logger: Monitors `/shared/output.txt`
2. All share an emptyDir volume
3. Verify the data pipeline works
4. Use different sub-paths for different containers

**Solution:** See `solution-07.md`

---

## Question 8: Volume Expansion

**Scenario:** Expand an existing PVC when storage needs increase.

**Tasks:**
1. Create a PVC with `1Gi` storage (with expandable StorageClass)
2. Create a pod using the PVC and fill it with data
3. Expand the PVC to `2Gi`
4. Verify the expansion succeeded
5. Verify pod can use the additional space

**Solution:** See `solution-08.md`

---

## Question 9: HostPath Volume with DirectoryOrCreate

**Scenario:** Mount host directory into pod, create if doesn't exist.

**Tasks:**
1. Create pod with hostPath volume
2. Set `type: DirectoryOrCreate`
3. Path: `/mnt/data`
4. Pod writes data to host path
5. Verify data persists on host after pod deletion
6. Understand security implications

---

## Question 10: PersistentVolume Reclaim Policy

**Scenario:** Understand PV reclaim behavior.

**Tasks:**
1. Create PV with `persistentVolumeReclaimPolicy: Retain`
2. Bind to PVC and use it
3. Delete PVC and observe PV status (Released)
4. Create PV with `persistentVolumeReclaimPolicy: Delete`
5. Observe automatic cleanup
6. When to use each policy

---

## Question 11: StorageClass with VolumeBindingMode

**Scenario:** Control when volumes are provisioned.

**Tasks:**
1. Create StorageClass with `volumeBindingMode: WaitForFirstConsumer`
2. Create PVC (should stay Pending)
3. Create pod using PVC
4. PVC binds only when pod is scheduled
5. Compare with `volumeBindingMode: Immediate`
6. Understand topology-aware scheduling

---

## Question 12: Projected Volume - Combine Multiple Sources

**Scenario:** Combine ConfigMap, Secret, and ServiceAccount token in one volume.

**Tasks:**
1. Create projected volume with:
   - ConfigMap data
   - Secret data
   - ServiceAccount token
   - Downward API (pod info)
2. All mounted in same directory
3. Each source in subdirectory or as individual files
4. Verify all sources are accessible

---

## Question 13: EmptyDir with Memory Medium

**Scenario:** Create RAM-backed temporary storage.

**Tasks:**
1. Create pod with emptyDir volume
2. Set `medium: Memory`
3. Set `sizeLimit: 128Mi`
4. Data stored in RAM, not disk
5. Faster but volatile
6. Verify with `df -h` inside pod

---

## Question 14: PVC with Specific Storage Class

**Scenario:** Request storage from specific StorageClass.

**Tasks:**
1. List available StorageClasses
2. Create PVC requesting `storageClassName: fast-ssd`
3. Verify PVC binds to correct storage type
4. Understand storage class parameters
5. Check provisioner used

---

## Question 15: StatefulSet Volume Claim Template

**Scenario:** Each pod gets its own PVC automatically.

**Tasks:**
1. Create StatefulSet with volumeClaimTemplates
2. Each pod gets unique PVC: `data-web-0`, `data-web-1`, etc.
3. Scale StatefulSet up - new PVCs created
4. Scale down - PVCs remain (not deleted)
5. Scale up - pods reattach to existing PVCs

---

## Question 16: SubPath Volume Mounts

**Scenario:** Mount specific file or directory from volume.

**Tasks:**
1. ConfigMap has multiple files
2. Mount only specific file using `subPath`
3. Rest of directory remains unchanged
4. Useful for mounting config into existing directories
5. Understand subPath limitations

---

## Question 17: ReadWriteMany (RWX) Volume

**Scenario:** Share volume across multiple pods.

**Tasks:**
1. Create PVC with `accessModes: ReadWriteMany`
2. Requires storage that supports RWX (NFS, CephFS, etc.)
3. Multiple pods write to same volume simultaneously
4. Verify data sharing between pods
5. Understand RWX support varies by storage

---

## Question 18: PersistentVolume Node Affinity

**Scenario:** Bind PV to specific node.

**Tasks:**
1. Create PV with nodeAffinity
2. PV only available on certain nodes
3. Pods using this PV scheduled on those nodes
4. Test by cordoning other nodes
5. Verify pod placement

---

## Question 19: Volume Snapshots

**Scenario:** Create snapshot of PVC for backup.

**Tasks:**
1. Create VolumeSnapshot from existing PVC
2. PVC must be bound and in use
3. Snapshot captures point-in-time data
4. Restore from snapshot to new PVC
5. Verify data is restored

---

## Question 20: ConfigMap and Secret Subpath with Updates

**Scenario:** Understand volume mount update behavior.

**Tasks:**
1. Mount ConfigMap as volume (full mount, not subPath)
2. Update ConfigMap data
3. Changes appear in pod after ~60 seconds
4. Mount with subPath - updates DON'T propagate
5. Document the difference

---

## Question 21: PVC Cloning

**Scenario:** Create new PVC as copy of existing PVC.

**Tasks:**
1. Create source PVC with data
2. Clone it using `dataSource` in new PVC
3. New PVC has copy of data
4. Both PVCs are independent
5. Verify cloning support in StorageClass

---

## Question 22: Local Persistent Volumes

**Scenario:** Use local disk storage (non-replicated).

**Tasks:**
1. Create local PV on specific node
2. PV tied to node - if node fails, data lost
3. High performance, no network overhead
4. Set `volumeBindingMode: WaitForFirstConsumer`
5. Pod scheduled on node with PV

---

## Question 23: Volume Mount Propagation

**Scenario:** Control mount propagation between host and container.

**Tasks:**
1. Set `mountPropagation: Bidirectional`
2. Mounts in container visible on host and vice versa
3. Or `mountPropagation: HostToContainer`
4. Or `mountPropagation: None` (default)
5. Understand privileged containers requirement

---

## Question 24: Ephemeral Volumes

**Scenario:** Use ephemeral volume that follows pod lifecycle.

**Tasks:**
1. Create pod with `ephemeral` volume type
2. Volume created when pod created
3. Volume deleted when pod deleted
4. Useful for scratch space
5. Compare with emptyDir

---

## Question 25: StatefulSet Persistent Storage Deletion

**Scenario:** Control what happens to PVCs when StatefulSet scales down.

**Tasks:**
1. Create StatefulSet with 5 replicas
2. Scale down to 3 replicas
3. PVCs for pods 3-4 remain (not deleted)
4. Scale back up to 5
5. Pods 3-4 reattach to existing PVCs with data intact
6. Manually delete PVCs if needed

---

## Storage Concepts

### Volume Types

| Type | Lifecycle | Use Case |
|------|-----------|----------|
| **emptyDir** | Pod lifetime | Temporary sharing between containers |
| **hostPath** | Host lifetime | Access host filesystem (use carefully) |
| **configMap** | Independent | Configuration files |
| **secret** | Independent | Sensitive data |
| **persistentVolumeClaim** | Independent | Persistent data across pod restarts |
| **projected** | Independent | Combine multiple volume sources |

### PersistentVolume (PV)

Cluster-level storage resource:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

### PersistentVolumeClaim (PVC)

Request for storage:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```

### Access Modes

| Mode | Abbreviation | Description |
|------|--------------|-------------|
| **ReadWriteOnce** | RWO | Single node read-write |
| **ReadOnlyMany** | ROX | Multiple nodes read-only |
| **ReadWriteMany** | RWX | Multiple nodes read-write |
| **ReadWriteOncePod** | RWOP | Single pod read-write (1.27+) |

### Reclaim Policies

| Policy | Behavior |
|--------|----------|
| **Retain** | Manual cleanup required |
| **Delete** | Deletes volume when PVC deleted |
| **Recycle** | Basic scrub (deprecated) |

### Volume Binding Modes

| Mode | Behavior |
|------|----------|
| **Immediate** | Bind as soon as PVC created |
| **WaitForFirstConsumer** | Wait until pod scheduled |

---

## StatefulSet Concepts

StatefulSets provide:
- Stable, unique network identifiers
- Stable, persistent storage
- Ordered, graceful deployment and scaling
- Ordered, automated rolling updates

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"  # Headless service
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:  # Auto-creates PVC per pod
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

---

## Tips for State Persistence

- emptyDir is lost when pod is deleted (not on restart)
- emptyDir can use memory: `emptyDir: {medium: "Memory"}`
- hostPath bypasses security - avoid in production
- PV is cluster-scoped; PVC is namespace-scoped
- PVC binds to PV based on: size, access mode, storage class
- Dynamic provisioning creates PV automatically from PVC
- StatefulSet pods: `<statefulset-name>-<ordinal>` (web-0, web-1, web-2)
- StatefulSet PVCs: `<volumeClaimTemplate-name>-<pod-name>` (www-web-0)
- Use WaitForFirstConsumer for topology-aware scheduling
- ConfigMap/Secret updates in mounted volumes have delay (~60s)
- Secrets in volumes are stored in tmpfs (memory)
- Default secret permissions: 0644 (readable by all)
- Use `defaultMode: 0400` for secret-only read by owner

---

## Quick Command Reference

```bash
# PersistentVolumes
kubectl get pv
kubectl describe pv <pv-name>
kubectl delete pv <pv-name>

# PersistentVolumeClaims
kubectl get pvc
kubectl describe pvc <pvc-name>
kubectl delete pvc <pvc-name>

# StorageClass
kubectl get storageclass
kubectl describe storageclass <sc-name>
kubectl get sc  # Short form

# Check PVC status
kubectl get pvc <pvc-name> -o jsonpath='{.status.phase}'

# Check which PV is bound to PVC
kubectl get pvc <pvc-name> -o jsonpath='{.spec.volumeName}'

# StatefulSets
kubectl get statefulsets
kubectl scale statefulset <name> --replicas=5
kubectl delete statefulset <name> --cascade=orphan  # Keep pods

# Volume info in pod
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o jsonpath='{.spec.volumes}'

# Exec into pod to check mounted volumes
kubectl exec -it <pod-name> -- df -h
kubectl exec -it <pod-name> -- ls -la /mnt/volume

# Expand PVC (if StorageClass allows)
kubectl edit pvc <pvc-name>
# Change spec.resources.requests.storage to larger value
```

---

## Common Patterns

### EmptyDir Volume
```yaml
volumes:
- name: cache-volume
  emptyDir: {}
```

### EmptyDir in Memory
```yaml
volumes:
- name: memory-volume
  emptyDir:
    medium: Memory
    sizeLimit: 128Mi
```

### ConfigMap Volume
```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
    items:
    - key: app.conf
      path: application.conf
```

### Secret Volume with Permissions
```yaml
volumes:
- name: secret-volume
  secret:
    secretName: db-secret
    defaultMode: 0400
```

### Projected Volume (Combine sources)
```yaml
volumes:
- name: all-in-one
  projected:
    sources:
    - configMap:
        name: config1
    - secret:
        name: secret1
    - serviceAccountToken:
        path: token
```

### PVC with Specific StorageClass
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fast-storage
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: ssd
  resources:
    requests:
      storage: 10Gi
```

---

## References

- [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [Dynamic Volume Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)
- [Configure Pods with ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
