# Multi-Container Pods - Essential Commands

## Creating Multi-Container Pods

```bash
# Generate base pod YAML (then edit to add containers)
kubectl run multi-pod --image=nginx --dry-run=client -o yaml > multi-pod.yaml

# Edit multi-pod.yaml to add additional containers:
# spec:
#   containers:
#   - name: main-container
#     image: nginx
#   - name: sidecar-container
#     image: busybox
#     command: ['sh', '-c', 'tail -f /var/log/app.log']

kubectl apply -f multi-pod.yaml
```

## Init Container Creation

```bash
# Generate pod with init container
kubectl run myapp --image=nginx --dry-run=client -o yaml > pod.yaml

# Add init containers section:
# spec:
#   initContainers:
#   - name: init-myservice
#     image: busybox
#     command: ['sh', '-c', 'until nslookup myservice; do sleep 2; done']
#   containers:
#   - name: myapp
#     image: nginx
```

## Managing Multi-Container Pods

```bash
# Get pod with container status
kubectl get pods
kubectl get pod <pod-name> -o wide

# Check all containers are ready
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].ready}'

# Get container names
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].name}'

# Get init container names
kubectl get pod <pod-name> -o jsonpath='{.spec.initContainers[*].name}'

# Describe pod (shows all containers)
kubectl describe pod <pod-name>
```

## Logs from Specific Containers

```bash
# View logs from specific container
kubectl logs <pod-name> -c <container-name>

# Follow logs
kubectl logs <pod-name> -c <container-name> -f

# View logs from all containers
kubectl logs <pod-name> --all-containers=true

# View previous container logs (if crashed)
kubectl logs <pod-name> -c <container-name> --previous

# View init container logs
kubectl logs <pod-name> -c <init-container-name>

# Stream logs from multiple containers
kubectl logs <pod-name> -c container1 -f &
kubectl logs <pod-name> -c container2 -f &
```

## Execute Commands in Specific Containers

```bash
# Exec into specific container
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh

# Run command in specific container
kubectl exec <pod-name> -c <container-name> -- env
kubectl exec <pod-name> -c <container-name> -- ls -la /shared-data

# Check processes in container
kubectl exec <pod-name> -c <container-name> -- ps aux

# Test network connectivity between containers
kubectl exec <pod-name> -c container1 -- curl localhost:8080
```

## Shared Volumes

```bash
# Create pod with shared emptyDir volume
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'echo "data" > /data/file.txt && sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  - name: reader
    image: busybox
    command: ['sh', '-c', 'cat /data/file.txt && sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
EOF

# Verify data sharing
kubectl exec shared-volume-pod -c writer -- ls -la /data
kubectl exec shared-volume-pod -c reader -- cat /data/file.txt
```

## Debugging Multi-Container Pods

```bash
# Check which containers are running/ready
kubectl get pod <pod-name> -o jsonpath='{range .status.containerStatuses[*]}{.name}{"\t"}{.ready}{"\t"}{.restartCount}{"\n"}{end}'

# Check init container status
kubectl get pod <pod-name> -o jsonpath='{.status.initContainerStatuses[*].state}'

# Get events for pod
kubectl get events --field-selector involvedObject.name=<pod-name>

# Describe specific issues
kubectl describe pod <pod-name> | grep -A 10 "Events:"
kubectl describe pod <pod-name> | grep -A 5 "Containers:"

# Check container image pull status
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].imageID}'
```

## Checking Container States

```bash
# Get all container states
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].state}'

# Check if container is waiting
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[?(@.state.waiting)].name}'

# Check restart count per container
kubectl get pod <pod-name> -o jsonpath='{range .status.containerStatuses[*]}{.name}{"\t"}{.restartCount}{"\n"}{end}'

# Get last termination reason
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'
```

## Port Forwarding with Multi-Container Pods

```bash
# Forward to specific container port
kubectl port-forward pod/<pod-name> 8080:80

# Forward multiple ports
kubectl port-forward pod/<pod-name> 8080:80 9090:9090
```

## Common Multi-Container Patterns

### Sidecar Pattern
```bash
# Main app + Log shipper
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: main-app
    image: nginx
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
  - name: log-shipper
    image: busybox
    command: ['sh', '-c', 'tail -f /logs/access.log']
    volumeMounts:
    - name: logs
      mountPath: /logs
  volumes:
  - name: logs
    emptyDir: {}
EOF

# Verify both containers running
kubectl get pod sidecar-pod
kubectl logs sidecar-pod -c log-shipper
```

### Ambassador Pattern
```bash
# Main app + Proxy
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-pod
spec:
  containers:
  - name: main-app
    image: busybox
    command: ['sh', '-c', 'while true; do wget -qO- localhost:8080; sleep 5; done']
  - name: ambassador
    image: nginx
    ports:
    - containerPort: 8080
EOF

# Test communication
kubectl logs ambassador-pod -c main-app
```

### Adapter Pattern
```bash
# Main app + Log adapter
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: adapter-pod
spec:
  containers:
  - name: main-app
    image: busybox
    command: ['sh', '-c', 'while true; do echo "LOG: data" >> /logs/app.log; sleep 5; done']
    volumeMounts:
    - name: logs
      mountPath: /logs
  - name: adapter
    image: busybox
    command: ['sh', '-c', 'tail -f /logs/app.log | sed "s/LOG:/[FORMATTED]/"']
    volumeMounts:
    - name: logs
      mountPath: /logs
  volumes:
  - name: logs
    emptyDir: {}
EOF

# Check adapted logs
kubectl logs adapter-pod -c adapter
```

### Init Container Pattern
```bash
# Wait for dependency + Main app
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'echo "Waiting for DB..."; sleep 10; echo "DB ready!"']
  - name: run-migrations
    image: busybox
    command: ['sh', '-c', 'echo "Running migrations..."; sleep 5; echo "Migrations complete!"']
  containers:
  - name: main-app
    image: nginx
EOF

# Check init container logs
kubectl logs init-pod -c wait-for-db
kubectl logs init-pod -c run-migrations
```

## Verification Commands

```bash
# Verify all containers started
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].state.running}'

# Check init containers completed
kubectl get pod <pod-name> -o jsonpath='{.status.initContainerStatuses[*].state.terminated.reason}'

# Verify shared volume access
kubectl exec <pod-name> -c container1 -- ls -la /shared
kubectl exec <pod-name> -c container2 -- ls -la /shared

# Check container resource usage
kubectl top pod <pod-name> --containers

# Test localhost communication
kubectl exec <pod-name> -c container1 -- curl localhost:8080
kubectl exec <pod-name> -c container1 -- nc -zv localhost 8080
```

## Troubleshooting Multi-Container Pods

```bash
# Init container failing
kubectl logs <pod-name> -c <init-container-name>
kubectl describe pod <pod-name> | grep -A 20 "Init Containers:"

# Container crashlooping
kubectl logs <pod-name> -c <container-name> --previous
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[?(@.name=="<container-name>")].lastState}'

# Shared volume issues
kubectl exec <pod-name> -c container1 -- df -h
kubectl exec <pod-name> -c container1 -- ls -la /shared-path

# Network communication issues between containers
kubectl exec <pod-name> -c container1 -- nc -zv localhost <port>
kubectl exec <pod-name> -c container1 -- curl -v localhost:<port>

# Check resource constraints
kubectl describe pod <pod-name> | grep -A 10 "Limits:"
kubectl top pod <pod-name> --containers
```

## Quick Tips

1. **All containers in a pod share:**
   - Network namespace (can communicate via localhost)
   - IPC namespace
   - UTS namespace (same hostname)
   - Volumes

2. **Init containers:**
   - Run sequentially before main containers
   - Must complete successfully
   - Useful for setup, wait for dependencies
   - Can't have readiness probes

3. **Sidecar containers:**
   - Run alongside main container
   - Enhance or extend main container functionality
   - Examples: log shippers, monitoring agents, proxies

4. **Use `-c` flag** to specify container in multi-container pods

5. **EmptyDir volumes:**
   - Shared storage between containers
   - Deleted when pod is deleted
   - Can be backed by memory: `emptyDir: {medium: "Memory"}`

6. **Container order doesn't matter** (except init containers)

7. **All containers must be ready** for pod to be ready

## Common Issues

**Problem:** Init container stuck
```bash
kubectl logs <pod> -c <init-container>
kubectl describe pod <pod> | grep -A 10 "Init Containers:"
```

**Problem:** Sidecar container crashlooping
```bash
kubectl logs <pod> -c <sidecar> --previous
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[?(@.name=="<sidecar>")].restartCount}'
```

**Problem:** Can't exec into specific container
```bash
# List container names first
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].name}'
# Then exec with correct name
kubectl exec -it <pod> -c <correct-name> -- /bin/sh
```

**Problem:** Shared volume not working
```bash
# Check volume mount paths
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].volumeMounts}'
# Verify files in each container
kubectl exec <pod> -c container1 -- ls -la /mount-path
kubectl exec <pod> -c container2 -- ls -la /mount-path
```
