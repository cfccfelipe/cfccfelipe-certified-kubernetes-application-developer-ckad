# Multi-Container Pods - Practice Questions

## Question 1: Ambassador Pattern - Proxy Container

**Scenario:** Create a pod with an ambassador container that acts as a proxy to an external service.

**Tasks:**
1. Create a pod named `ambassador-pod` with two containers:
   - Main container: `busybox:1.36` that makes HTTP requests to `localhost:8080`
   - Ambassador container: `nginx:1.21-alpine` that proxies to an external service
2. Configure the nginx container to listen on port `8080`
3. Both containers should share the same network namespace
4. Verify communication between containers

**Files:** `ambassador-pod.yaml`
**Solution:** See `solution-01.md`

---

## Question 2: Adapter Pattern - Log Formatter

**Scenario:** An application writes logs in a custom format. Create an adapter container to transform logs.

**Tasks:**
1. Create a pod named `log-adapter-pod` with two containers:
   - Main container: Writes logs to `/var/log/app.log`
   - Adapter container: Reads logs and transforms them
2. Use an `emptyDir` volume to share logs between containers
3. Main container: `busybox:1.36` writing logs in custom format
4. Adapter container: `busybox:1.36` reading and transforming logs
5. Verify both containers can access the shared volume

**Solution:** See `solution-02.md`

---

## Question 3: Sidecar Pattern - Log Collector

**Scenario:** Deploy an application with a sidecar container for log collection.

**Tasks:**
1. Create a pod named `logging-pod` with two containers:
   - Main app: `nginx:1.21` writing access logs
   - Sidecar: Log shipping agent (use `busybox:1.36` to simulate)
2. Share the log directory using a volume
3. Sidecar should continuously tail the logs
4. Verify both containers are running
5. Check logs from both containers

**Files:** `sidecar-logging.yaml`
**Solution:** See `solution-03.md`

---

## Question 4: Init Containers - Database Migration

**Scenario:** Run database migrations before starting the main application.

**Tasks:**
1. Create a pod named `migration-pod` with:
   - Init container 1: Check database connectivity (simulate with `nslookup`)
   - Init container 2: Run migrations (simulate with sleep and echo)
   - Main container: Application container
2. Init containers must complete successfully before main starts
3. Use `busybox:1.36` for all containers
4. Verify the execution order

**Solution:** See `solution-04.md`

---

## Question 5: Shared Volume Between Containers

**Scenario:** Multiple containers need to share data through a volume.

**Tasks:**
1. Create a pod named `shared-volume-pod` with three containers:
   - Writer container: Writes data to `/data/output.txt` every 5 seconds
   - Reader container: Reads from `/data/output.txt` every 10 seconds
   - Processor container: Processes data from `/data/output.txt`
2. All containers share an `emptyDir` volume
3. Use `busybox:1.36` for all containers
4. Verify all containers can access the shared data

**Solution:** See `solution-05.md`

---

## Question 6: Init Container with ConfigMap

**Scenario:** Use an init container to prepare configuration files from ConfigMap.

**Tasks:**
1. Create a ConfigMap named `app-config-raw` with raw configuration data
2. Create a pod with:
   - Init container: Transforms ConfigMap data into application config format
   - Main container: Uses the transformed configuration
3. Use `emptyDir` volume to share the transformed config
4. Verify the init container completes before main starts

**Files:** `init-configmap.yaml`
**Solution:** See `solution-06.md`

---

## Question 7: Sidecar Pattern - Metrics Exporter

**Scenario:** Add a metrics exporter sidecar to expose application metrics.

**Tasks:**
1. Create a pod with main application container (nginx)
2. Add sidecar container that exposes metrics on port 9090
3. Main container writes metrics to shared volume
4. Sidecar reads metrics and exposes via HTTP
5. Both containers share emptyDir volume at `/metrics`
6. Test metrics endpoint from sidecar

---

## Question 8: Init Container Chain - Multiple Init Containers

**Scenario:** Run multiple init containers in sequence.

**Tasks:**
1. Create pod with 3 init containers that run sequentially:
   - Init 1: Wait for database (simulate with sleep 10)
   - Init 2: Run database migrations (simulate with echo)
   - Init 3: Warm up cache (simulate with wget)
2. Main container starts only after all init containers succeed
3. Check logs from each init container
4. Verify execution order

---

## Question 9: Ambassador Pattern - Database Proxy

**Scenario:** Use ambassador pattern to proxy database connections.

**Tasks:**
1. Create pod with:
   - Main app container connecting to `localhost:5432`
   - Ambassador container proxying to external database
2. Ambassador listens on 5432 and forwards to real database
3. Main app only knows about localhost
4. Shared network namespace enables localhost communication

---

## Question 10: Adapter Pattern - Log Format Conversion

**Scenario:** Convert application logs to standard format.

**Tasks:**
1. Main container writes logs in custom format to shared volume
2. Adapter container reads logs and converts to JSON format
3. Adapter writes converted logs to separate file
4. Both containers share emptyDir volume
5. Verify log transformation

---

## Question 11: Init Container with Git Clone

**Scenario:** Use init container to clone git repository.

**Tasks:**
1. Init container clones git repo into shared volume
2. Main container serves the cloned content via nginx
3. Use emptyDir volume at `/usr/share/nginx/html`
4. Simulate git clone with: `echo "content" > index.html`
5. Verify main container serves the content

---

## Question 12: Sidecar Pattern - SSL/TLS Termination

**Scenario:** Add sidecar to handle SSL termination.

**Tasks:**
1. Main app container serves HTTP on port 8080
2. Sidecar container (nginx) handles HTTPS on port 443
3. Sidecar forwards to main app via localhost:8080
4. Mount SSL certificates from secret
5. Test HTTPS access to sidecar

---

## Question 13: Multi-Container with Different Restart Policies

**Scenario:** Understand container restart behavior.

**Tasks:**
1. Create pod with:
   - Container 1: Long-running service (should always run)
   - Container 2: One-time job (should complete and stop)
2. Note: All containers in pod share same restart policy
3. Observe what happens when one container exits
4. Document the behavior

---

## Question 14: Init Container Failure Handling

**Scenario:** Handle init container failures gracefully.

**Tasks:**
1. Create pod with init container that can fail
2. Set `restartPolicy: OnFailure` for init containers
3. Make init container fail first time, succeed second time
4. Observe pod behavior during init container retries
5. Check pod events and status

---

## Question 15: Sidecar with Resource Limits

**Scenario:** Set different resource limits for sidecar.

**Tasks:**
1. Main container: requests=100m CPU, limits=200m CPU
2. Sidecar container: requests=50m CPU, limits=100m CPU
3. Verify both containers have independent resource limits
4. Use `kubectl top pod --containers` to check actual usage
5. Ensure total doesn't exceed node capacity

---

## Question 16: Shared Process Namespace

**Scenario:** Share process namespace between containers.

**Tasks:**
1. Create pod with `shareProcessNamespace: true`
2. Two containers in the pod
3. Use `kubectl exec` to see all processes from both containers
4. Container 1 can see Container 2's processes
5. Verify with `ps aux` from each container

---

## Question 17: Init Container with Volume Permissions

**Scenario:** Init container sets up volume permissions.

**Tasks:**
1. Init container changes ownership of shared volume
2. Use `chown` to set correct user:group
3. Main container runs as non-root and needs write access
4. Verify main container can write to the volume
5. Check file permissions

---

## Question 18: Adapter Pattern - Metrics Aggregation

**Scenario:** Aggregate metrics from multiple sources.

**Tasks:**
1. Multiple containers write metrics to shared volume
2. Adapter container reads all metrics files
3. Adapter aggregates and exposes via single endpoint
4. Use emptyDir volume for metrics sharing
5. Test aggregated metrics endpoint

---

## Question 19: Sidecar Pattern - Config Reload

**Scenario:** Sidecar monitors ConfigMap and reloads app config.

**Tasks:**
1. Main app reads config from file
2. Sidecar watches ConfigMap changes
3. Sidecar updates config file when ConfigMap changes
4. Main app reloads configuration automatically
5. Use inotify or polling for change detection

---

## Question 20: Multiple Init Containers with Dependencies

**Scenario:** Init containers with dependency order.

**Tasks:**
1. Init container 1: Download dependencies
2. Init container 2: Build application (depends on 1)
3. Init container 3: Run tests (depends on 2)
4. Main container: Run application
5. If any init container fails, pod should not start
6. Verify strict sequential execution

---

## Multi-Container Patterns Summary

### Sidecar Pattern
- Helper container enhances/extends main container
- Examples: Log shippers, monitoring agents, proxies
- Both containers run simultaneously

### Ambassador Pattern
- Proxy container simplifies network connections
- Examples: Database proxy, API gateway
- Hides complexity from main container

### Adapter Pattern
- Transforms output to standard format
- Examples: Log formatters, metrics converters
- Standardizes heterogeneous containers

### Init Containers
- Run to completion before main containers start
- Examples: Setup, validation, waiting for dependencies
- Run sequentially in order defined

---

## Tips for Multi-Container Pods

- All containers in a pod share:
  - Network namespace (localhost communication)
  - IPC namespace
  - Volumes
  - Same lifecycle (scheduled together)
- Init containers:
  - Run sequentially before main containers
  - Must complete successfully
  - Can't have readiness probes
- Use `kubectl logs <pod> -c <container>` to view specific container logs
- Use `kubectl exec <pod> -c <container> -- <command>` for specific container
- Debugging: Check all containers with `kubectl describe pod`

---

## Quick Command Reference

```bash
# View logs from specific container
kubectl logs <pod-name> -c <container-name>

# View logs from all containers
kubectl logs <pod-name> --all-containers=true

# Execute command in specific container
kubectl exec <pod-name> -c <container-name> -- <command>

# View init container logs
kubectl logs <pod-name> -c <init-container-name>

# Debug multi-container pod
kubectl describe pod <pod-name>

# Check which containers are ready
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].name}'
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].ready}'
```

---

## References

- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Sidecar Containers](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/)
- [Multi-Container Pod Design Patterns](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/)
- [Shared Volumes](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)
