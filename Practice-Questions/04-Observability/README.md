# Observability - Practice Questions

## Question 1: Configure Liveness and Readiness Probes

**Scenario:** An application needs health checks to ensure it's running and ready to serve traffic.

**Tasks:**
1. Create a deployment named `probe-demo` using `nginx:1.21` image
2. Configure liveness probe:
   - HTTP GET to `/healthz` on port `80`
   - Initial delay: `10s`
   - Period: `5s`
   - Failure threshold: `3`
3. Configure readiness probe:
   - HTTP GET to `/ready` on port `80`
   - Initial delay: `5s`
   - Period: `3s`
4. Observe what happens when probes fail
5. Fix the probe paths to `/` to make them work

**Files:** `probe-deployment.yaml`
**Solution:** See `solution-01.md`

---

## Question 2: Debug Failing Pod with Logs

**Scenario:** A pod is crash-looping. Use logs to identify and fix the issue.

**Tasks:**
1. Apply the pod from `crashing-pod.yaml`
2. Check why the pod is failing using `kubectl logs`
3. Check previous container logs if needed
4. Identify the error in the configuration
5. Fix and reapply the pod
6. Verify it's running successfully

**Files:** `crashing-pod.yaml`
**Solution:** See `solution-02.md`

---

## Question 3: Monitor Container Resources

**Scenario:** Monitor resource usage of containers to optimize resource requests/limits.

**Tasks:**
1. Create a deployment from `resource-demo.yaml`
2. Use `kubectl top` to check CPU and memory usage
3. Identify containers using more resources than requested
4. Update resource requests and limits based on actual usage
5. Verify the deployment rolls out with new resources

**Files:** `resource-demo.yaml`
**Solution:** See `solution-03.md`

---

## Question 4: Startup Probe for Slow Starting Application

**Scenario:** An application takes 60 seconds to start. Configure appropriate startup probe.

**Tasks:**
1. Create a pod named `slow-start-app` that simulates slow startup
2. Configure startup probe:
   - HTTP GET probe
   - Allow up to 90 seconds for startup
   - Check every 5 seconds
3. After startup, use liveness probe every 10 seconds
4. Verify the pod doesn't get killed during startup
5. Verify liveness probe takes over after startup

**Solution:** See `solution-04.md`

---

## Question 5: Debug Pod with Exec and Describe

**Scenario:** A pod is running but not behaving correctly. Debug using kubectl exec and describe.

**Tasks:**
1. Deploy the pod from `debug-pod.yaml`
2. Use `kubectl describe` to check events and conditions
3. Use `kubectl exec` to:
   - Check if required files exist
   - Test network connectivity
   - Check environment variables
   - Verify processes are running
4. Identify and document the issue
5. Fix the configuration

**Files:** `debug-pod.yaml`
**Solution:** See `solution-05.md`

---

## Question 6: Container Logs from Multiple Pods

**Scenario:** A deployment has multiple pods. View and filter logs from all pods.

**Tasks:**
1. Deploy the application from `multi-pod-app.yaml` with 3 replicas
2. View logs from all pods using label selectors
3. Stream logs in real-time
4. Filter logs to show only error messages
5. Save logs to a file for analysis

**Files:** `multi-pod-app.yaml`
**Solution:** See `solution-06.md`

---

## Question 7: Ephemeral Debug Container

**Scenario:** A minimal container has no debugging tools. Use ephemeral containers to debug.

**Tasks:**
1. Create a pod with a minimal image (distroless or scratch-based)
2. Try to exec into the pod (it will fail - no shell)
3. Use `kubectl debug` to attach an ephemeral container with debugging tools
4. Investigate the pod's filesystem and network from the debug container
5. Remove the ephemeral container when done

**Solution:** See `solution-07.md`

---

## Question 8: Combined Probes - Startup, Liveness, and Readiness

**Scenario:** Configure all three probe types for a slow-starting application.

**Tasks:**
1. Application takes 60 seconds to start
2. Configure startup probe: 90 seconds max (failureThreshold=9, periodSeconds=10)
3. Configure liveness probe: Check every 10 seconds after startup
4. Configure readiness probe: Check every 5 seconds
5. Verify pod doesn't get killed during startup
6. Verify liveness/readiness probes activate after startup

---

## Question 9: Logs from Crashed Pod

**Scenario:** A pod crashed and restarted. Retrieve logs from the previous instance.

**Tasks:**
1. Create a pod that crashes after 30 seconds
2. Wait for it to restart
3. View logs from the previous (crashed) container
4. Use `--previous` flag
5. Compare with current container logs

---

## Question 10: Container with Exit Code Debugging

**Scenario:** Container exits with error code. Debug the issue.

**Tasks:**
1. Create a pod with command that exits with code 1
2. Check container status and exit code
3. Check termination reason and message
4. View events for the pod
5. Fix the issue and redeploy

---

## Question 11: Probe with Custom HTTP Headers

**Scenario:** Application requires custom headers for health check.

**Tasks:**
1. Configure HTTP liveness probe with custom headers
2. Add header: `X-Custom-Header: health-check`
3. Application validates the header
4. Test probe behavior with and without header
5. Verify probe succeeds only with correct header

---

## Question 12: TCP Socket Probe

**Scenario:** Check application health using TCP connection.

**Tasks:**
1. Create deployment with TCP socket liveness probe
2. Probe connects to port 8080
3. No HTTP required, just TCP connection
4. Configure timeouts and thresholds
5. Simulate port being closed and observe behavior

---

## Question 13: Exec Probe with Script

**Scenario:** Use command execution for health checking.

**Tasks:**
1. Create pod with exec liveness probe
2. Probe runs: `cat /tmp/healthy`
3. Main container creates/deletes this file
4. Observe pod restarts when file is missing
5. Test probe failure and recovery

---

## Question 14: Monitoring Multiple Containers in Pod

**Scenario:** Debug multi-container pod where one container is failing.

**Tasks:**
1. Create pod with 3 containers
2. Container 2 is crashlooping
3. Use `kubectl describe` to identify which container
4. View logs from the failing container
5. Check container status and restart count
6. Fix the issue

---

## Question 15: Events and Troubleshooting Workflow

**Scenario:** Use events to debug pod issues.

**Tasks:**
1. Create a pod with multiple issues (image pull, probe failure, OOM)
2. Use `kubectl get events --sort-by=.metadata.creationTimestamp`
3. Filter events by pod name
4. Identify all issues from events
5. Fix each issue systematically

---

## Question 16: Resource Metrics and Vertical Scaling

**Scenario:** Monitor resource usage and adjust requests/limits.

**Tasks:**
1. Deploy app with initial resources: 100m CPU, 128Mi memory
2. Use `kubectl top pod` to monitor actual usage
3. Application uses 250m CPU and 256Mi memory
4. Update deployment with appropriate resources
5. Verify no OOMKilled or CPU throttling

---

## Question 17: Liveness Probe Causing Crash Loop

**Scenario:** Misconfigured liveness probe causes unnecessary restarts.

**Tasks:**
1. Create deployment with aggressive liveness probe (initialDelaySeconds=5, periodSeconds=5)
2. Application takes 20 seconds to start
3. Observe CrashLoopBackOff
4. Fix by adjusting initialDelaySeconds
5. Or use startup probe instead

---

## Question 18: Readiness Probe with Dependencies

**Scenario:** Pod shouldn't receive traffic until dependencies are ready.

**Tasks:**
1. Application depends on external database
2. Configure readiness probe to check database connectivity
3. Simulate database being unavailable
4. Verify pod is Running but not Ready
5. Pod shouldn't receive service traffic

---

## Question 19: Debug Node Issues Affecting Pods

**Scenario:** Pods fail on specific node.

**Tasks:**
1. Identify node with failing pods
2. Use `kubectl describe node` to check node conditions
3. Check node events
4. Use `kubectl debug node/<node-name>` to investigate
5. Cordon node if necessary
6. Drain and fix node issues

---

## Question 20: Logs with Timestamps and Filtering

**Scenario:** Find specific errors in logs from multiple pods.

**Tasks:**
1. Deployment with 5 replicas
2. View logs from all pods: `kubectl logs -l app=myapp --all-containers=true`
3. Filter logs for ERROR messages
4. Add timestamps to correlate events
5. Use `--since=1h` to view recent logs only
6. Save filtered logs to file for analysis

---

## Question 21: Port Forward for Local Testing

**Scenario:** Test application locally without exposing service.

**Tasks:**
1. Deploy application without service
2. Use `kubectl port-forward` to access pod locally
3. Test on localhost:8080
4. Forward multiple ports simultaneously
5. Background the port-forward and continue working

---

## Question 22: Container Resource Requests vs Actual Usage

**Scenario:** Identify over-provisioned or under-provisioned pods.

**Tasks:**
1. List all pods with resource requests
2. Use `kubectl top pod` for actual usage
3. Calculate request/usage ratio
4. Identify pods using <50% of requests (over-provisioned)
5. Identify pods at >90% of limits (under-provisioned)
6. Recommend adjustments

---

## Observability Concepts

### Health Probes

| Probe Type | Purpose | When to Use |
|------------|---------|-------------|
| **Liveness** | Detects if container is alive | Restart unhealthy containers |
| **Readiness** | Detects if container can serve traffic | Remove from service endpoints |
| **Startup** | Allows slow-starting containers | Protect during initialization |

### Probe Mechanisms

- **httpGet**: HTTP GET request to specified path
- **tcpSocket**: TCP connection to specified port
- **exec**: Execute command in container
- **grpc**: gRPC health check (Kubernetes 1.24+)

### Probe Configuration

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Value
  initialDelaySeconds: 15
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3
```

---

## Tips for Observability

- Always configure readiness probes for production
- Liveness probes should check application health, not dependencies
- Startup probes prevent premature liveness check failures
- Use `kubectl logs --previous` to see logs from crashed containers
- Use `kubectl describe pod` to see events and state transitions
- Container restarts indicate liveness probe failures
- Pod stays in service only when readiness probe succeeds
- Use `kubectl top pod` for resource monitoring (requires metrics-server)
- Ephemeral containers (kubectl debug) are useful for minimal images

---

## Quick Command Reference

```bash
# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> --previous
kubectl logs -f <pod-name>  # Follow/stream logs
kubectl logs -l app=myapp --all-containers=true

# Debug
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl exec -it <pod-name> -- /bin/sh
kubectl debug <pod-name> -it --image=busybox:1.36

# Resource monitoring
kubectl top pod
kubectl top pod <pod-name> --containers
kubectl top node

# Port forwarding for testing
kubectl port-forward pod/<pod-name> 8080:80

# Copy files for debugging
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

---

## Common Issues and Debugging Steps

### Pod Stuck in Pending
1. Check events: `kubectl describe pod`
2. Check node resources: `kubectl top node`
3. Check PVC status if volumes are used
4. Check for resource quotas: `kubectl describe quota -n <namespace>`

### Pod Crash Loop (CrashLoopBackOff)
1. Check logs: `kubectl logs <pod> --previous`
2. Check events: `kubectl describe pod`
3. Check container command/args
4. Check resource limits
5. Check liveness probe configuration

### Pod Running but Not Ready
1. Check readiness probe configuration
2. Check logs for application errors
3. Exec into container and test probe endpoint manually
4. Check if service dependencies are available

### High Restart Count
1. Liveness probe failing
2. Application crashes
3. OOMKilled (memory limit exceeded)
4. Node issues

---

## References

- [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Debug Running Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
- [Debug Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)
- [Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
- [Ephemeral Containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)
- [Resource Metrics Pipeline](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/)
