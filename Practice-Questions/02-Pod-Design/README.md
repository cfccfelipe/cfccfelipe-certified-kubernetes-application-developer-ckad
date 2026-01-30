# Pod Design - Practice Questions

## Question 1: Labels, Selectors, and Annotations

**Scenario:** Organize pods using labels and annotations for better management.

**Tasks:**
1. Create 5 pods with different labels:
   - Pod 1: `app=web, tier=frontend, env=prod`
   - Pod 2: `app=web, tier=frontend, env=dev`
   - Pod 3: `app=api, tier=backend, env=prod`
   - Pod 4: `app=api, tier=backend, env=dev`
   - Pod 5: `app=cache, tier=backend, env=prod`
2. Add annotations to each pod with deployment info
3. Select pods using various label selectors
4. Update labels on running pods
5. Delete pods using label selectors

**Solution:** See `solution-01.md`

---

## Question 2: Deployments with Rolling Updates

**Scenario:** Deploy an application and perform rolling updates with rollback capability.

**Tasks:**
1. Create a deployment named `webapp` with:
   - Image: `nginx:1.19`
   - Replicas: `4`
   - Labels: `app=webapp`
2. Update the deployment to `nginx:1.21` and observe the rollout
3. Check rollout status and history
4. Update with a bad image `nginx:invalid` (will fail)
5. Rollback to the previous working version
6. Pause and resume a rollout

**Files:** `webapp-deployment.yaml`
**Solution:** See `solution-02.md`

---

## Question 3: Deployment Strategies

**Scenario:** Implement different deployment strategies.

**Tasks:**
1. Create a deployment with `RollingUpdate` strategy:
   - `maxSurge: 1`
   - `maxUnavailable: 0` (zero downtime)
2. Create a deployment with `Recreate` strategy
3. Compare the behavior during updates
4. Document the differences
5. Choose appropriate strategy for different scenarios

**Files:** `rolling-update.yaml`, `recreate-deployment.yaml`
**Solution:** See `solution-03.md`

---

## Question 4: Jobs and CronJobs

**Scenario:** Create batch processing jobs and scheduled tasks.

**Tasks:**
1. Create a Job named `batch-processor` that:
   - Runs 5 parallel pods
   - Each pod processes and exits
   - Uses `completions: 10`
2. Create a CronJob named `backup-job` that:
   - Runs every 30 minutes: `*/30 * * * *`
   - Keeps last 3 successful jobs
   - Keeps last 1 failed job
   - Has deadline of 100 seconds
3. Manually trigger the CronJob
4. View job history and logs

**Solution:** See `solution-04.md`

---

## Question 5: Blue-Green Deployment with Labels

**Scenario:** Implement blue-green deployment pattern using labels.

**Tasks:**
1. Deploy "blue" version with label `version=blue`
2. Create a service pointing to blue version
3. Deploy "green" version with label `version=green`
4. Test green version without affecting traffic
5. Switch service to green version by updating selector
6. Rollback to blue if needed
7. Delete old blue deployment

**Files:** `blue-green-setup.yaml`
**Solution:** See `solution-05.md`

---

## Question 6: Canary Deployment

**Scenario:** Implement canary deployment to test new version with small traffic percentage.

**Tasks:**
1. Deploy stable version with 4 replicas
2. Deploy canary version with 1 replica (20% traffic)
3. Both versions share the same service labels
4. Monitor canary version
5. Scale up canary and scale down stable gradually
6. Complete the rollout or rollback

**Solution:** See `solution-06.md`

---

## Question 7: ReplicaSet Management

**Scenario:** Work directly with ReplicaSets to understand their behavior.

**Tasks:**
1. Create a ReplicaSet with 3 replicas
2. Manually delete a pod and observe self-healing
3. Scale the ReplicaSet up and down
4. Update the pod template (note: ReplicaSet doesn't trigger update)
5. Delete the ReplicaSet without deleting pods
6. Adopt existing pods into a new ReplicaSet

**Files:** `replicaset.yaml`
**Solution:** See `solution-07.md`

---

## Question 8: Job with Completions and Parallelism

**Scenario:** Run batch processing job with specific completion requirements.

**Tasks:**
1. Create Job that processes 10 items (completions=10)
2. Run 3 pods in parallel (parallelism=3)
3. Each pod processes one item and exits
4. Monitor job progress
5. Verify all 10 completions succeeded
6. Check job duration

---

## Question 9: CronJob Manual Trigger and Suspend

**Scenario:** Manage CronJob execution manually.

**Tasks:**
1. Create CronJob that runs every hour
2. Manually trigger it immediately: `kubectl create job --from=cronjob/my-cron manual-1`
3. Suspend the CronJob: `kubectl patch cronjob my-cron -p '{"spec":{"suspend":true}}'`
4. Verify no new jobs are created
5. Resume the CronJob
6. Clean up old jobs

---

## Question 10: Deployment with Max Surge and Max Unavailable

**Scenario:** Fine-tune rolling update strategy.

**Tasks:**
1. Create deployment with 10 replicas
2. Set `maxSurge: 3` and `maxUnavailable: 2`
3. Update image and observe rollout
4. At most 13 pods during update (10 + 3 surge)
5. At least 8 pods available during update (10 - 2 unavailable)
6. Verify behavior matches strategy

---

## Question 11: Label Selector with Set-Based Requirements

**Scenario:** Use advanced label selectors.

**Tasks:**
1. Create pods with various labels: `env=prod`, `env=dev`, `tier=frontend`, `tier=backend`
2. Select pods where `env in (prod,staging)`
3. Select pods where `tier notin (cache)`
4. Select pods with label `critical` exists
5. Select pods where label `temp` doesn't exist
6. Combine multiple selectors

---

## Question 12: Deployment History and Revision Tracking

**Scenario:** Manage deployment revisions.

**Tasks:**
1. Create deployment
2. Make 5 image updates (track changes)
3. View history: `kubectl rollout history deployment/myapp`
4. Rollback to specific revision: `--to-revision=3`
5. Check revision details
6. Understand `revisionHistoryLimit`

---

## Question 13: Job Failure Handling and Backoff

**Scenario:** Handle job failures with retry logic.

**Tasks:**
1. Create Job with command that fails
2. Set `backoffLimit: 3` (max retries)
3. Job retries 3 times then marks as Failed
4. Check pod restart count
5. View failed job logs
6. Fix and recreate job

---

## Question 14: HorizontalPodAutoscaler (HPA)

**Scenario:** Auto-scale deployment based on CPU usage.

**Tasks:**
1. Create deployment with resource requests
2. Create HPA: min=2, max=10, targetCPU=50%
3. Generate load to trigger scaling
4. Monitor HPA status: `kubectl get hpa -w`
5. Observe pods scaling up
6. Remove load and watch scale down

---

## Question 15: DaemonSet Creation and Management

**Scenario:** Run one pod per node.

**Tasks:**
1. Create DaemonSet for log collector
2. Verify one pod per node
3. Add new node and verify pod is created
4. Update DaemonSet with new image
5. Observe rolling update behavior
6. Use `kubectl rollout` commands

---

## Question 16: Deployment with PodDisruptionBudget

**Scenario:** Ensure minimum availability during disruptions.

**Tasks:**
1. Create deployment with 5 replicas
2. Create PodDisruptionBudget: minAvailable=3
3. Try to drain a node
	1. Verify PDB prevents too many pods from being evicted
4. Understand voluntary vs involuntary disruptions

---

## Question 17: Job with TTL After Finished

**Scenario:** Automatically clean up completed jobs.

**Tasks:**
1. Create Job with `ttlSecondsAfterFinished: 100`
2. Job completes successfully
3. After 100 seconds, job and pods are automatically deleted
4. Verify cleanup happens
5. Use for jobs that don't need history

---

## Question 18: CronJob Concurrency Policy

**Scenario:** Control concurrent CronJob executions.

**Tasks:**
1. Create CronJob that runs every minute
2. Each job takes 2 minutes to complete
3. Set `concurrencyPolicy: Forbid` (skip if previous running)
4. Or `concurrencyPolicy: Replace` (kill previous, start new)
5. Or `concurrencyPolicy: Allow` (run both)
6. Observe different behaviors

---

## Question 19: Deployment Rollback on Failure

**Scenario:** Automatic rollback when update fails.

**Tasks:**
1. Create deployment with working image
2. Update to bad image (nginx:invalid)
3. Observe pods failing
4. Manually rollback: `kubectl rollout undo`
5. Verify return to previous version
6. Check rollout history

---

## Question 20: StatefulSet with Ordered Updates

**Scenario:** Update StatefulSet pods in order.

**Tasks:**
1. Create StatefulSet with 5 replicas
2. Update to new image
3. Observe pods updated one at a time: web-4, web-3, web-2, web-1, web-0
4. Verify ordering with `kubectl get pods -w`
5. Understand partition-based updates

---

## Question 21: Job Completion Mode - Indexed

**Scenario:** Run indexed batch processing.

**Tasks:**
1. Create Job with `completionMode: Indexed`
2. Each pod gets unique index (0, 1, 2, ...)
3. Pod processes its specific index
4. Use `JOB_COMPLETION_INDEX` environment variable
5. Verify each index is processed once

---

## Question 22: Deployment Annotations and Change-Cause

**Scenario:** Track deployment changes with annotations.

**Tasks:**
1. Create deployment
2. Update with: `kubectl set image deployment/app nginx=nginx:1.21 --record`
3. Add annotation: `kubernetes.io/change-cause="Updated to 1.21"`
4. View history with change causes
5. Makes rollback decisions easier

---

## Pod Design Concepts

### Labels and Selectors

**Labels:** Key-value pairs attached to objects
```yaml
metadata:
  labels:
    app: myapp
    tier: frontend
    environment: production
```

**Selectors:** Query objects by labels
```bash
# Equality-based
kubectl get pods -l app=myapp
kubectl get pods -l app=myapp,tier=frontend

# Set-based
kubectl get pods -l 'environment in (prod,staging)'
kubectl get pods -l 'tier notin (cache)'
```

### Annotations

Attach non-identifying metadata:
```yaml
metadata:
  annotations:
    description: "Web application frontend"
    version: "1.2.3"
    contact: "team@example.com"
    deployed-by: "jenkins"
```

### Deployment Strategies

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| **RollingUpdate** | Gradual replacement | Zero-downtime updates |
| **Recreate** | Delete all, then create new | Incompatible versions, not HA |

**RollingUpdate Parameters:**
- `maxSurge`: Max pods above desired count during update
- `maxUnavailable`: Max pods unavailable during update

### Job Types

**Job:** Run to completion
```yaml
spec:
  completions: 5      # Total successful completions needed
  parallelism: 2      # How many pods to run in parallel
  backoffLimit: 4     # Number of retries before marking failed
```

**CronJob:** Scheduled jobs
```yaml
spec:
  schedule: "*/5 * * * *"  # Every 5 minutes
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  concurrencyPolicy: Forbid  # or Allow, Replace
```

---

## Tips for Pod Design

- Use labels for organization and selection, not configuration
- Use annotations for non-identifying metadata
- Deployments manage ReplicaSets automatically
- Never update ReplicaSet template directly (use Deployments)
- Jobs: Set `restartPolicy: OnFailure` or `Never`
- CronJobs: Use `suspend: true` to temporarily disable
- Blue-green: Instant switch, requires double resources
- Canary: Gradual rollout, more controlled
- Use `kubectl rollout` commands for deployment management
- `kubectl scale` affects only replica count, not template
- Deployment history is kept (can rollback multiple versions)

---

## Quick Command Reference

```bash
# Labels and Selectors
kubectl get pods --show-labels
kubectl get pods -l app=myapp
kubectl get pods -l 'env in (prod,staging)'
kubectl label pod <pod-name> env=prod
kubectl label pod <pod-name> env-  # Remove label

# Annotations
kubectl annotate pod <pod-name> description="My description"
kubectl annotate pod <pod-name> description-  # Remove annotation

# Deployments
kubectl create deployment <name> --image=<image> --replicas=3
kubectl scale deployment <name> --replicas=5
kubectl set image deployment/<name> <container>=<new-image>
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=2
kubectl rollout pause deployment/<name>
kubectl rollout resume deployment/<name>

# Jobs
kubectl create job <name> --image=<image>
kubectl create job <name> --from=cronjob/<cronjob-name>
kubectl get jobs
kubectl delete job <name>

# CronJobs
kubectl create cronjob <name> --image=<image> --schedule="*/5 * * * *"
kubectl get cronjobs
kubectl describe cronjob <name>
kubectl patch cronjob <name> -p '{"spec":{"suspend":true}}'  # Suspend

# ReplicaSets
kubectl get replicasets
kubectl scale replicaset <name> --replicas=3
kubectl delete replicaset <name> --cascade=orphan  # Keep pods
```

---

## Common Patterns

### Rolling Update with Zero Downtime
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

### Fast Rollout
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 50%
    maxUnavailable: 50%
```

### Parallel Job Processing
```yaml
spec:
  completions: 10
  parallelism: 3
  backoffLimit: 4
```

### CronJob with Deadline
```yaml
spec:
  schedule: "0 */6 * * *"
  startingDeadlineSeconds: 100
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```

---

## References

- [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
- [Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
- [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
- [Deployment Strategies](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy)
