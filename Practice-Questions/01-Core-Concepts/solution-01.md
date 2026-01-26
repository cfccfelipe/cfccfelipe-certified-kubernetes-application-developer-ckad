# Solution: Question 1 - Create and Manage Basic Pods

## Step 1: Create the pod using kubectl run

```bash
kubectl run nginx-pod \
  --image=nginx:1.21 \
  --port=80 \
  --labels="app=web,tier=frontend" \
  --requests="memory=128Mi,cpu=100m" \
  --dry-run=client -o yaml > nginx-pod.yaml
```

## Step 2: Apply the pod

```bash
kubectl apply -f nginx-pod.yaml
```

**Alternative:** Direct creation without YAML file:
```bash
kubectl run nginx-pod \
  --image=nginx:1.21 \
  --port=80 \
  --labels="app=web,tier=frontend" \
  --requests="memory=128Mi,cpu=100m"
```

## Step 3: Verify the pod is running

```bash
kubectl get pod nginx-pod
```

Expected output:
```
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          10s
```

## Step 4: Get the pod's IP address

```bash
kubectl get pod nginx-pod -o wide
```

Or using jsonpath:
```bash
kubectl get pod nginx-pod -o jsonpath='{.status.podIP}'
```

## Step 5: Verify labels

```bash
kubectl get pod nginx-pod --show-labels
```

## Step 6: Describe pod to verify all configurations

```bash
kubectl describe pod nginx-pod
```

Check for:
- Image: nginx:1.21
- Port: 80
- Labels: app=web, tier=frontend
- Resource Requests: memory=128Mi, cpu=100m

## Generated YAML for Reference

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
    tier: frontend
  name: nginx-pod
  namespace: default
spec:
  containers:
  - image: nginx:1.21
    name: nginx-pod
    ports:
    - containerPort: 80
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
  restartPolicy: Always
```

## Cleanup

```bash
kubectl delete pod nginx-pod
```

## Key Takeaways

- `kubectl run` is the fastest way to create pods in the exam
- Use `--dry-run=client -o yaml` to generate YAML without creating the resource
- Labels are key-value pairs that help organize and select resources
- Resource requests ensure the pod gets minimum guaranteed resources
- Always verify with `kubectl get` and `kubectl describe`
