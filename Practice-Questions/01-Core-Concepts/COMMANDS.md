# Core Concepts - Essential Commands

## Quick Pod Creation

```bash
# Basic pod
kubectl run nginx --image=nginx

# Pod with specific settings
kubectl run nginx --image=nginx:1.21 \
  --port=80 \
  --labels="app=web,tier=frontend" \
  --requests="memory=128Mi,cpu=100m" \
  --limits="memory=256Mi,cpu=200m"

# Generate YAML for editing
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Pod with command
kubectl run busybox --image=busybox --command -- sleep 3600

# Pod with environment variables
kubectl run myapp --image=nginx \
  --env="DB_HOST=mysql" \
  --env="DB_PORT=3306"

# Pod in specific namespace
kubectl run nginx --image=nginx -n production
```

## Pod Management

```bash
# Get pods
kubectl get pods
kubectl get pods -o wide  # Show IPs and nodes
kubectl get pods --show-labels
kubectl get pods -l app=nginx  # Filter by label
kubectl get pods -A  # All namespaces

# Get pod details
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml
kubectl get pod <pod-name> -o json

# Watch pod status
kubectl get pods -w

# Get pod IP
kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'

# Get pod node
kubectl get pod <pod-name> -o jsonpath='{.spec.nodeName}'
```

## Pod Operations

```bash
# Delete pod
kubectl delete pod <pod-name>
kubectl delete pod <pod-name> --force --grace-period=0  # Force delete
kubectl delete pod -l app=nginx  # Delete by label

# Edit pod (limited fields)
kubectl edit pod <pod-name>

# Replace pod (for immutable fields)
kubectl get pod <pod-name> -o yaml > pod.yaml
# Edit pod.yaml
kubectl replace --force -f pod.yaml

# Patch pod
kubectl patch pod <pod-name> -p '{"spec":{"containers":[{"name":"nginx","image":"nginx:1.21"}]}}'
```

## Pod Debugging

```bash
# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container pod
kubectl logs <pod-name> --previous  # Previous crashed container
kubectl logs <pod-name> -f  # Follow logs
kubectl logs <pod-name> --tail=50  # Last 50 lines
kubectl logs <pod-name> --since=1h  # Last hour

# Execute commands
kubectl exec <pod-name> -- env
kubectl exec <pod-name> -- ls /
kubectl exec -it <pod-name> -- /bin/sh
kubectl exec -it <pod-name> -c <container> -- /bin/bash

# Port forwarding
kubectl port-forward pod/<pod-name> 8080:80

# Copy files
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

## Labels and Selectors

```bash
# Add label
kubectl label pod <pod-name> env=prod

# Update label
kubectl label pod <pod-name> env=staging --overwrite

# Remove label
kubectl label pod <pod-name> env-

# Show labels
kubectl get pods --show-labels

# Filter by labels
kubectl get pods -l app=nginx
kubectl get pods -l app=nginx,env=prod
kubectl get pods -l 'env in (prod,staging)'
kubectl get pods -l app!=nginx
```

## Namespace Operations

```bash
# Create namespace
kubectl create namespace dev

# Set default namespace
kubectl config set-context --current --namespace=dev

# Get resources in namespace
kubectl get all -n dev

# Get resources in all namespaces
kubectl get pods -A
```

## Resource Inspection

```bash
# Check resource requests/limits
kubectl describe pod <pod-name> | grep -A 5 "Limits:"
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].resources}'

# Check pod conditions
kubectl get pod <pod-name> -o jsonpath='{.status.conditions[*]}'

# Check restart count
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].restartCount}'

# Check pod events
kubectl get events --field-selector involvedObject.name=<pod-name>
```

## Quick Testing

```bash
# Test DNS
kubectl run test --image=busybox:1.36 --rm -it --restart=Never -- nslookup kubernetes

# Test connectivity
kubectl run curl --image=curlimages/curl --rm -it --restart=Never -- curl http://service

# Create temporary debug pod
kubectl run debug --image=nicolaka/netshoot --rm -it -- /bin/bash
```

## JSONPath Queries

```bash
# Pod names
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Pod IPs
kubectl get pods -o jsonpath='{.items[*].status.podIP}'

# Container images
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'

# Pod status
kubectl get pods -o jsonpath='{.items[*].status.phase}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP
```

## Time-Saving Tips

1. **Alias Setup:**
```bash
alias k=kubectl
alias kgp='kubectl get pods'
alias kdp='kubectl describe pod'
export do='--dry-run=client -o yaml'
```

2. **Fast YAML Generation:**
```bash
k run nginx --image=nginx $do > pod.yaml
```

3. **Quick Edit:**
```bash
KUBE_EDITOR=nano kubectl edit pod nginx
```

4. **Use kubectl explain:**
```bash
kubectl explain pod.spec
kubectl explain pod.spec.containers
kubectl explain pod.spec.containers.resources
```

## Common Patterns for CKAD

### Pattern 1: Create pod with specific settings
```bash
kubectl run myapp --image=nginx:1.21 \
  --port=80 \
  --labels="app=myapp,tier=frontend" \
  --requests="memory=64Mi,cpu=100m" \
  --env="ENV=prod" \
  --dry-run=client -o yaml > myapp.yaml

# Edit myapp.yaml if needed
kubectl apply -f myapp.yaml
```

### Pattern 2: Update and replace
```bash
kubectl get pod myapp -o yaml > myapp-update.yaml
# Edit myapp-update.yaml
kubectl delete pod myapp
kubectl apply -f myapp-update.yaml
```

### Pattern 3: Quick verification
```bash
kubectl apply -f pod.yaml && \
kubectl get pod <pod-name> && \
kubectl describe pod <pod-name>
```

### Pattern 4: Debug failing pod
```bash
kubectl get pod <pod-name>
kubectl describe pod <pod-name> | grep -A 10 Events
kubectl logs <pod-name>
kubectl get pod <pod-name> -o yaml | grep -A 10 "status:"
```

## Remember

- Pods are ephemeral - don't rely on their IP addresses
- Use labels for organization and selection
- Resource requests are for scheduling; limits prevent overconsumption
- RestartPolicy: Always (default), OnFailure, Never
- Phase: Pending, Running, Succeeded, Failed, Unknown
- Containers in a pod share network namespace (localhost communication)
