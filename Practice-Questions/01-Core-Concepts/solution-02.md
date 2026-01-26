# Solution: Question 2 - Fix Broken Pod from YAML

## Step 1: Apply the broken pod

```bash
kubectl apply -f broken-pod.yaml
```

## Step 2: Check pod status

```bash
kubectl get pod broken-pod
```

Expected output showing issues:
```
NAME         READY   STATUS    RESTARTS   AGE
broken-pod   0/2     Pending   0          5s
```

## Step 3: Describe the pod to identify the issue

```bash
kubectl describe pod broken-pod
```

Look for events showing the error:
```
Events:
  Warning  FailedScheduling  pod has unbound immediate PersistentVolumeClaims
  Warning  FailedCreate      Error: memory limit 32Mi is less than request 64Mi
```

## Step 4: Identify the bug

**Problem:** In the YAML file, the memory limit (32Mi) is **less than** the memory request (64Mi).

From `broken-pod.yaml`:
```yaml
resources:
  requests:
    memory: "64Mi"   # Request: 64Mi
  limits:
    memory: "32Mi"   # Limit: 32Mi - THIS IS THE BUG!
```

**Rule:** Limits must be greater than or equal to requests!

## Step 5: Fix the YAML file

Edit `broken-pod.yaml` and change the memory limit to be at least 64Mi:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: broken-pod
  namespace: default
spec:
  containers:
  - name: web-container
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"  # FIXED: Now greater than request
        cpu: "500m"
  - name: sidecar-container
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
```

## Step 6: Delete the broken pod and recreate

```bash
kubectl delete pod broken-pod
kubectl apply -f broken-pod.yaml
```

## Step 7: Verify the pod is running

```bash
kubectl get pod broken-pod
```

Expected output:
```
NAME         READY   STATUS    RESTARTS   AGE
broken-pod   2/2     Running   0          10s
```

## Step 8: Verify both containers are running

```bash
kubectl describe pod broken-pod
```

Check the Containers section shows both containers are running.

## Quick verification commands

```bash
# Check if both containers are ready
kubectl get pod broken-pod -o jsonpath='{.status.containerStatuses[*].ready}'

# Check container names
kubectl get pod broken-pod -o jsonpath='{.status.containerStatuses[*].name}'
```

## Cleanup

```bash
kubectl delete pod broken-pod
```

## Key Takeaways

- Always check `kubectl describe pod` for detailed error messages
- Resource limits must be >= resource requests
- Common resource validation errors:
  - Limit < Request (this question)
  - Invalid resource units (Mi vs M, m for CPU)
  - Negative values
- Multi-container pods show READY as `X/Y` where X is ready and Y is total
- Use `kubectl get pod -o yaml` to see the full pod specification
