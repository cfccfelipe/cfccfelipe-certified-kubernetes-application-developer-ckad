# Pod Design - Essential Commands

## Labels and Selectors

```bash
# Add label to pod
kubectl label pod <pod> env=prod

# Update existing label
kubectl label pod <pod> env=staging --overwrite

# Remove label
kubectl label pod <pod> env-

# Add multiple labels
kubectl label pod <pod> tier=frontend app=web version=v1

# Label all pods matching selector
kubectl label pods -l app=nginx tier=frontend

# Show all labels
kubectl get pods --show-labels

# Show specific labels as columns
kubectl get pods -L app,env,tier

# Filter by labels (equality)
kubectl get pods -l app=nginx
kubectl get pods -l app=nginx,env=prod

# Filter by labels (set-based)
kubectl get pods -l 'env in (prod,staging)'
kubectl get pods -l 'tier notin (cache)'
kubectl get pods -l app
kubectl get pods -l '!app'

# Count pods by label
kubectl get pods -l app=nginx --no-headers | wc -l
```

## Annotations

```bash
# Add annotation
kubectl annotate pod <pod> description="Web server pod"

# Add multiple annotations
kubectl annotate pod <pod> version="1.0.0" owner="team-backend"

# Update annotation
kubectl annotate pod <pod> description="Updated description" --overwrite

# Remove annotation
kubectl annotate pod <pod> description-

# View annotations
kubectl get pod <pod> -o jsonpath='{.metadata.annotations}'
kubectl describe pod <pod> | grep -A 5 "Annotations:"
```

## Deployments

```bash
# Create deployment
kubectl create deployment nginx --image=nginx --replicas=3

# Generate YAML
kubectl create deployment nginx --image=nginx --replicas=3 --dry-run=client -o yaml > deploy.yaml

# Get deployments
kubectl get deployments
kubectl get deploy  # Short form

# Describe deployment
kubectl describe deployment nginx

# Get deployment YAML
kubectl get deployment nginx -o yaml

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Autoscale deployment
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# Delete deployment
kubectl delete deployment nginx
```

## Rolling Updates and Rollbacks

```bash
# Update deployment image
kubectl set image deployment/nginx nginx=nginx:1.21

# Update with record (deprecated but useful)
kubectl set image deployment/nginx nginx=nginx:1.21 --record

# Check rollout status
kubectl rollout status deployment/nginx

# View rollout history
kubectl rollout history deployment/nginx

# View specific revision
kubectl rollout history deployment/nginx --revision=2

# Pause rollout
kubectl rollout pause deployment/nginx

# Resume rollout
kubectl rollout resume deployment/nginx

# Undo rollout (go back one version)
kubectl rollout undo deployment/nginx

# Undo to specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# Restart deployment (rolling restart)
kubectl rollout restart deployment/nginx
```

## Deployment Strategies

```bash
# Set rolling update strategy
kubectl patch deployment nginx -p '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'

# Set recreate strategy
kubectl patch deployment nginx -p '{"spec":{"strategy":{"type":"Recreate"}}}'

# View strategy
kubectl get deployment nginx -o jsonpath='{.spec.strategy}'
```

## ReplicaSets

```bash
# Get ReplicaSets
kubectl get replicasets
kubectl get rs  # Short form

# Describe ReplicaSet
kubectl describe rs <rs-name>

# Scale ReplicaSet (don't do this - use Deployment!)
kubectl scale rs <rs-name> --replicas=3

# Delete ReplicaSet (keep pods)
kubectl delete rs <rs-name> --cascade=orphan

# Get RS for deployment
kubectl get rs -l app=nginx
```

## Jobs

```bash
# Create job
kubectl create job hello --image=busybox -- echo "Hello World"

# Create job from CronJob
kubectl create job test-job --from=cronjob/my-cronjob

# Generate YAML
kubectl create job hello --image=busybox --dry-run=client -o yaml -- echo "Hello" > job.yaml

# Get jobs
kubectl get jobs

# Describe job
kubectl describe job hello

# View job pods
kubectl get pods -l job-name=hello

# View job logs
kubectl logs -l job-name=hello

# Delete job (and its pods)
kubectl delete job hello

# Delete job (keep pods)
kubectl delete job hello --cascade=orphan

# Wait for job completion
kubectl wait --for=condition=complete job/hello --timeout=60s
```

## CronJobs

```bash
# Create CronJob
kubectl create cronjob hello --image=busybox --schedule="*/5 * * * *" -- echo "Hello"

# Generate YAML
kubectl create cronjob backup --image=busybox --schedule="0 2 * * *" --dry-run=client -o yaml -- /backup.sh > cronjob.yaml

# Get CronJobs
kubectl get cronjobs
kubectl get cj  # Short form

# Describe CronJob
kubectl describe cj hello

# Manually trigger CronJob
kubectl create job --from=cronjob/hello manual-trigger

# Suspend CronJob
kubectl patch cronjob hello -p '{"spec":{"suspend":true}}'

# Resume CronJob
kubectl patch cronjob hello -p '{"spec":{"suspend":false}}'

# View CronJob history
kubectl get jobs -l app=hello

# Delete CronJob
kubectl delete cronjob hello
```

## Deployment Management

```bash
# Edit deployment
kubectl edit deployment nginx

# Patch deployment (specific field)
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'

# Set image
kubectl set image deployment/nginx nginx=nginx:1.21 --record

# Set resources
kubectl set resources deployment nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi

# Set environment variable
kubectl set env deployment/nginx ENV=production

# View environment variables
kubectl set env deployment/nginx --list

# Remove environment variable
kubectl set env deployment/nginx ENV-

# Add volume
kubectl set volume deployment/nginx --add --name=cache --type=emptyDir --mount-path=/cache

# Set service account
kubectl set serviceaccount deployment nginx my-sa
```

## Blue-Green Deployment

```bash
# Deploy blue version
kubectl create deployment app-blue --image=nginx:1.19
kubectl label deployment app-blue version=blue

# Expose service pointing to blue
kubectl expose deployment app-blue --port=80 --target-port=80 --name=app-service --selector=version=blue

# Deploy green version
kubectl create deployment app-green --image=nginx:1.21
kubectl label deployment app-green version=green

# Test green
kubectl port-forward deployment/app-green 8080:80

# Switch to green
kubectl patch service app-service -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback to blue if needed
kubectl patch service app-service -p '{"spec":{"selector":{"version":"blue"}}}'

# Delete old version
kubectl delete deployment app-blue
```

## Canary Deployment

```bash
# Deploy stable version
kubectl create deployment app-stable --image=nginx:1.19 --replicas=4
kubectl label deployment app-stable version=stable

# Deploy canary version (fewer replicas)
kubectl create deployment app-canary --image=nginx:1.21 --replicas=1
kubectl label deployment app-canary version=canary

# Service selects both (by app label, not version)
kubectl expose deployment app-stable --port=80 --selector=app=myapp

# Monitor canary
kubectl logs -l version=canary -f

# Scale up canary, scale down stable
kubectl scale deployment app-canary --replicas=3
kubectl scale deployment app-stable --replicas=2

# Complete rollout
kubectl scale deployment app-canary --replicas=5
kubectl scale deployment app-stable --replicas=0
kubectl delete deployment app-stable
kubectl patch deployment app-canary --type='json' -p='[{"op": "remove", "path": "/spec/template/metadata/labels/version"}]'
```

## Checking Deployment Status

```bash
# Get deployment status
kubectl get deployment nginx -o wide

# Check replica counts
kubectl get deployment nginx -o jsonpath='{.spec.replicas}{" desired, "}{.status.replicas}{" current, "}{.status.availableReplicas}{" available"}'

# Check rollout progress
kubectl get deployment nginx -o jsonpath='{.status.conditions[?(@.type=="Progressing")].message}'

# Get pods for deployment
kubectl get pods -l app=nginx

# Check which revision is active
kubectl rollout status deployment/nginx
kubectl get rs -l app=nginx
```

## Troubleshooting Deployments

```bash
# Deployment not progressing
kubectl describe deployment nginx
kubectl get events --field-selector involvedObject.name=nginx

# Pods not starting
kubectl get pods -l app=nginx
kubectl describe pod <pod-from-deployment>
kubectl logs <pod-from-deployment>

# Rollout stuck
kubectl rollout status deployment/nginx
kubectl describe deployment nginx | grep -A 10 "Conditions:"

# ReplicaSet issues
kubectl get rs -l app=nginx
kubectl describe rs <rs-name>

# Image pull issues
kubectl describe deployment nginx | grep -i image
kubectl get pods -l app=nginx -o jsonpath='{.items[*].status.containerStatuses[*].state}'
```

## Quick Patterns

### Pattern 1: Quick deployment update
```bash
kubectl set image deployment/nginx nginx=nginx:1.21 && \
kubectl rollout status deployment/nginx
```

### Pattern 2: Rollback if update fails
```bash
kubectl set image deployment/nginx nginx=nginx:bad-tag
# Wait and see it fail
kubectl rollout undo deployment/nginx
```

### Pattern 3: Zero-downtime deployment
```bash
# Edit deployment to set:
# strategy:
#   type: RollingUpdate
#   rollingUpdate:
#     maxSurge: 1
#     maxUnavailable: 0
kubectl patch deployment nginx -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'
```

### Pattern 4: Force pod restart
```bash
kubectl rollout restart deployment/nginx
```

## Remember

- **Deployments manage ReplicaSets** - don't edit RS directly
- **Labels connect everything** - Deployment → RS → Pods
- **Update = new ReplicaSet** - old RS kept for rollback
- **Rollout history** - limited by `revisionHistoryLimit`
- **Jobs run to completion** - use `restartPolicy: OnFailure` or `Never`
- **CronJobs** - use standard cron schedule format
- **Parallelism** - how many pods run concurrently
- **Completions** - how many successful pods needed
- **Blue-green** - instant switch, double resources temporarily
- **Canary** - gradual rollout, controlled traffic split
- **Always test before switching** - use port-forward or separate service
