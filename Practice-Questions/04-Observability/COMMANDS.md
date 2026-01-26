# Observability - Essential Commands

## Health Probes

```bash
# Add liveness probe (HTTP)
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
# Edit pod.yaml to add:
# livenessProbe:
#   httpGet:
#     path: /healthz
#     port: 80
#   initialDelaySeconds: 15
#   periodSeconds: 10

# Add readiness probe (TCP)
# readinessProbe:
#   tcpSocket:
#     port: 80
#   initialDelaySeconds: 5
#   periodSeconds: 5

# Add startup probe (exec)
# startupProbe:
#   exec:
#     command:
#     - cat
#     - /tmp/healthy
#   failureThreshold: 30
#   periodSeconds: 10
```

## Viewing Logs

```bash
# Basic logs
kubectl logs <pod-name>

# Follow logs (stream)
kubectl logs -f <pod-name>

# Last N lines
kubectl logs --tail=50 <pod-name>

# Since time
kubectl logs --since=1h <pod-name>
kubectl logs --since-time=2024-01-25T10:00:00Z <pod-name>

# Previous container (crashed)
kubectl logs <pod-name> --previous

# All containers in pod
kubectl logs <pod-name> --all-containers=true

# Specific container
kubectl logs <pod-name> -c <container-name>

# Multiple pods by label
kubectl logs -l app=nginx --all-containers=true

# Save logs to file
kubectl logs <pod-name> > pod.log

# Timestamps
kubectl logs <pod-name> --timestamps=true
```

## Debugging Commands

```bash
# Describe pod (most important!)
kubectl describe pod <pod-name>

# Get pod YAML
kubectl get pod <pod-name> -o yaml

# Get pod status
kubectl get pod <pod-name> -o jsonpath='{.status.phase}'

# Get container statuses
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].state}'

# Check restart count
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].restartCount}'

# Get pod events
kubectl get events --field-selector involvedObject.name=<pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp

# Get all events in namespace
kubectl get events -n <namespace>

# Watch events
kubectl get events -w
```

## Execute Commands in Pods

```bash
# Interactive shell
kubectl exec -it <pod-name> -- /bin/sh
kubectl exec -it <pod-name> -- /bin/bash

# Run single command
kubectl exec <pod-name> -- ls -la /
kubectl exec <pod-name> -- env
kubectl exec <pod-name> -- ps aux
kubectl exec <pod-name> -- cat /etc/hosts

# Check if file exists
kubectl exec <pod-name> -- test -f /path/to/file && echo "exists" || echo "not found"

# Test network connectivity
kubectl exec <pod-name> -- curl -v http://service:80
kubectl exec <pod-name> -- wget -qO- http://service:80
kubectl exec <pod-name> -- nc -zv service 80

# DNS resolution
kubectl exec <pod-name> -- nslookup kubernetes
kubectl exec <pod-name> -- nslookup service-name

# Check disk space
kubectl exec <pod-name> -- df -h

# Check memory
kubectl exec <pod-name> -- free -m
```

## Resource Monitoring

```bash
# Pod resource usage (requires metrics-server)
kubectl top pod
kubectl top pod <pod-name>
kubectl top pod --containers
kubectl top pod -l app=nginx

# Node resource usage
kubectl top node
kubectl top node <node-name>

# Sort by CPU
kubectl top pod --sort-by=cpu

# Sort by memory
kubectl top pod --sort-by=memory

# All namespaces
kubectl top pod -A
```

## Port Forwarding (Testing)

```bash
# Forward pod port to local
kubectl port-forward pod/<pod-name> 8080:80

# Forward service port
kubectl port-forward svc/<service-name> 8080:80

# Multiple ports
kubectl port-forward pod/<pod-name> 8080:80 9090:9090

# Bind to specific address
kubectl port-forward --address 0.0.0.0 pod/<pod-name> 8080:80

# Background
kubectl port-forward pod/<pod-name> 8080:80 &
```

## Copy Files (Debugging)

```bash
# Copy from pod to local
kubectl cp <pod-name>:/path/to/file ./local-file

# Copy from local to pod
kubectl cp ./local-file <pod-name>:/path/to/file

# Specific container
kubectl cp <pod-name>:/path -c <container> ./local-file

# Copy directory
kubectl cp <pod-name>:/var/log ./logs
```

## Debug with Ephemeral Containers

```bash
# Add debug container to running pod
kubectl debug <pod-name> -it --image=busybox

# Debug with specific image
kubectl debug <pod-name> -it --image=nicolaka/netshoot

# Debug node
kubectl debug node/<node-name> -it --image=busybox

# Create copy of pod for debugging
kubectl debug <pod-name> -it --copy-to=<new-pod-name> --container=debug
```

## Temporary Test Pods

```bash
# Quick test pod (auto-deleted)
kubectl run test --image=busybox --rm -it --restart=Never -- sh

# Test DNS
kubectl run dnstest --image=busybox --rm -it --restart=Never -- nslookup kubernetes

# Test curl
kubectl run curl-test --image=curlimages/curl --rm -it --restart=Never -- curl http://service

# Test network
kubectl run netshoot --image=nicolaka/netshoot --rm -it --restart=Never -- bash
```

## Checking Probe Status

```bash
# Get liveness probe config
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].livenessProbe}'

# Get readiness probe config
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].readinessProbe}'

# Check pod conditions
kubectl get pod <pod> -o jsonpath='{.status.conditions[*]}'

# Check if pod is ready
kubectl get pod <pod> -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'

# Get probe failure messages
kubectl describe pod <pod> | grep -A 5 "Liveness:\|Readiness:\|Startup:"

# Check if probes are failing
kubectl get events --field-selector involvedObject.name=<pod> | grep -i "probe\|liveness\|readiness"
```

## Troubleshooting Common Issues

### CrashLoopBackOff
```bash
# Check previous logs
kubectl logs <pod> --previous

# Check events
kubectl describe pod <pod> | grep -A 10 Events

# Check exit code
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.exitCode}'

# Check termination reason
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'
```

### ImagePullBackOff
```bash
# Check events
kubectl describe pod <pod> | grep -A 5 "Events"

# Check image name
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].image}'

# Check image pull policy
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].imagePullPolicy}'
```

### Pending Pod
```bash
# Check events
kubectl describe pod <pod> | grep -A 10 "Events"

# Check node resources
kubectl top node

# Check resource requests
kubectl describe pod <pod> | grep -A 10 "Requests:"

# Check PVC status (if using volumes)
kubectl get pvc
```

### Not Ready
```bash
# Check readiness probe
kubectl describe pod <pod> | grep -A 5 "Readiness:"

# Check if containers are running
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].ready}'

# Check logs
kubectl logs <pod>

# Test readiness endpoint manually
kubectl exec <pod> -- curl localhost:80/ready
```

### OOMKilled
```bash
# Check if pod was killed
kubectl describe pod <pod> | grep -i "oomkilled"

# Check memory limits
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].resources.limits.memory}'

# Check actual memory usage
kubectl top pod <pod> --containers

# View previous pod state
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'
```

## Monitoring Best Practices

```bash
# Check all pods status
kubectl get pods --field-selector status.phase!=Running

# List pods with restarts
kubectl get pods --field-selector status.phase=Running -o json | \
  jq -r '.items[] | select(.status.containerStatuses[].restartCount > 0) | .metadata.name'

# Check probe configuration
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[*].livenessProbe}{"\n"}{end}'

# Monitor pod events
kubectl get events --watch --field-selector involvedObject.kind=Pod
```

## Quick Debugging Workflow

```bash
# Step 1: Check status
kubectl get pod <pod>

# Step 2: Describe (most important!)
kubectl describe pod <pod>

# Step 3: Check logs
kubectl logs <pod>
kubectl logs <pod> --previous  # If crashed

# Step 4: Check events
kubectl get events --field-selector involvedObject.name=<pod>

# Step 5: Exec into pod (if running)
kubectl exec -it <pod> -- sh

# Step 6: Test from another pod
kubectl run test --image=busybox --rm -it -- wget -qO- http://<pod-ip>
```

## JSONPath Queries for Troubleshooting

```bash
# Get all pod IPs
kubectl get pods -o jsonpath='{.items[*].status.podIP}'

# Get all container images
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'

# Get pods not running
kubectl get pods -o jsonpath='{.items[?(@.status.phase!="Running")].metadata.name}'

# Get restart counts
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[*].restartCount}{"\n"}{end}'

# Get pod creation times
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.creationTimestamp}{"\n"}{end}'
```

## Remember

- **Always start with `kubectl describe pod`** - 80% of issues are visible here
- **Check logs** - Both current and previous (--previous)
- **Use ephemeral debug containers** for minimal images
- **Liveness probe** - Restart if app is unhealthy
- **Readiness probe** - Remove from service if not ready
- **Startup probe** - Protect slow-starting apps from liveness probe
- **Probe failure threshold** - Number of failures before action
- **Initial delay** - Wait before first probe check
- **Period** - How often to check
