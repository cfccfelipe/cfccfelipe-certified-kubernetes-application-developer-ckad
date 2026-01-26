# Configuration - Essential Commands

## ConfigMap Commands

```bash
# Create from literals
kubectl create configmap app-config \
  --from-literal=db_host=mysql \
  --from-literal=db_port=3306 \
  --from-literal=log_level=INFO

# Create from file
kubectl create configmap nginx-config --from-file=nginx.conf

# Create from directory (all files)
kubectl create configmap app-configs --from-file=config-dir/

# Create from env file
kubectl create configmap app-env --from-env-file=app.env

# Generate YAML
kubectl create configmap app-config \
  --from-literal=key=value \
  --dry-run=client -o yaml > configmap.yaml

# Get ConfigMaps
kubectl get configmaps
kubectl get cm  # Short form
kubectl describe cm app-config
kubectl get cm app-config -o yaml

# Edit ConfigMap
kubectl edit cm app-config

# Delete ConfigMap
kubectl delete cm app-config
```

## Secret Commands

```bash
# Create generic secret from literals
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=SecretPass123

# Create from file
kubectl create secret generic ssh-key --from-file=ssh-privatekey=~/.ssh/id_rsa

# Create TLS secret
kubectl create secret tls tls-secret \
  --cert=path/to/tls.cert \
  --key=path/to/tls.key

# Create Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# Generate YAML
kubectl create secret generic my-secret \
  --from-literal=key=value \
  --dry-run=client -o yaml > secret.yaml

# Get Secrets
kubectl get secrets
kubectl describe secret db-secret  # Values hidden
kubectl get secret db-secret -o yaml  # Base64 encoded
kubectl get secret db-secret -o jsonpath='{.data.username}' | base64 -d  # Decode

# Edit Secret
kubectl edit secret db-secret

# Delete Secret
kubectl delete secret db-secret
```

## Using ConfigMaps and Secrets in Pods

### Environment Variables from ConfigMap
```bash
# Generate pod using ConfigMap
kubectl run myapp --image=nginx --dry-run=client -o yaml > pod.yaml
# Then add envFrom section in YAML:
# envFrom:
# - configMapRef:
#     name: app-config
```

### Environment Variables from Secret
```yaml
# Add to pod spec:
envFrom:
- secretRef:
    name: db-secret
```

### Single env var from ConfigMap
```yaml
env:
- name: DB_HOST
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: db_host
```

### Single env var from Secret
```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

### Mount as Volume
```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
volumeMounts:
- name: config-volume
  mountPath: /etc/config
```

## ServiceAccount Commands

```bash
# Create ServiceAccount
kubectl create serviceaccount api-sa

# Get ServiceAccounts
kubectl get serviceaccounts
kubectl get sa  # Short form
kubectl describe sa api-sa

# Use in pod spec:
# serviceAccountName: api-sa
# OR during creation:
kubectl run nginx --image=nginx --serviceaccount=api-sa --dry-run=client -o yaml

# Get SA token (pre-1.24)
kubectl get secret $(kubectl get sa api-sa -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 -d

# Create token (1.24+)
kubectl create token api-sa

# Delete ServiceAccount
kubectl delete sa api-sa
```

## SecurityContext Commands

```bash
# Run as specific user (pod level)
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
# Add to pod spec:
# securityContext:
#   runAsUser: 1000
#   runAsGroup: 3000
#   fsGroup: 2000

# Check pod security context
kubectl get pod <pod> -o jsonpath='{.spec.securityContext}'

# Check container security context
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].securityContext}'

# Verify user inside pod
kubectl exec <pod> -- id
kubectl exec <pod> -- whoami
```

## Resource Quota Commands

```bash
# Create ResourceQuota
kubectl create quota my-quota \
  --hard=cpu=1,memory=1Gi,pods=2,services=3,secrets=5

# Create from YAML
kubectl apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "1000m"
    requests.memory: "1Gi"
    limits.cpu: "2000m"
    limits.memory: "2Gi"
    pods: "10"
EOF

# Get quotas
kubectl get quota
kubectl describe quota my-quota

# Check quota usage
kubectl describe quota -n production

# Delete quota
kubectl delete quota my-quota
```

## LimitRange Commands

```bash
# Create LimitRange
kubectl apply -f - <<EOF
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-mem-limit-range
spec:
  limits:
  - default:
      cpu: 200m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    max:
      cpu: 500m
      memory: 512Mi
    min:
      cpu: 50m
      memory: 64Mi
    type: Container
EOF

# Get LimitRanges
kubectl get limitrange
kubectl describe limitrange cpu-mem-limit-range

# Delete LimitRange
kubectl delete limitrange cpu-mem-limit-range
```

## Updating Deployments with ConfigMap/Secret Changes

```bash
# Edit ConfigMap
kubectl edit cm app-config

# Trigger rolling update (add annotation to deployment)
kubectl patch deployment myapp -p \
  '{"spec":{"template":{"metadata":{"annotations":{"version":"2"}}}}}'

# OR restart deployment
kubectl rollout restart deployment myapp

# Check rollout status
kubectl rollout status deployment myapp
```

## Verification Commands

```bash
# Check if ConfigMap is mounted
kubectl exec <pod> -- ls -la /etc/config

# Check ConfigMap files content
kubectl exec <pod> -- cat /etc/config/app.conf

# Check environment variables from ConfigMap
kubectl exec <pod> -- env | grep DB_

# Check Secret is mounted
kubectl exec <pod> -- ls -la /etc/secrets
kubectl exec <pod> -- cat /etc/secrets/username

# Verify ServiceAccount token
kubectl exec <pod> -- cat /var/run/secrets/kubernetes.io/serviceaccount/token

# Check resource limits applied
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].resources}'

# Verify security context
kubectl exec <pod> -- id
kubectl exec <pod> -- ps aux
```

## Common Patterns

### Pattern 1: Create ConfigMap and use in Pod
```bash
# Create ConfigMap
kubectl create cm app-config --from-literal=ENV=prod

# Create pod using it
kubectl run myapp --image=nginx --dry-run=client -o yaml > pod.yaml
# Edit pod.yaml to add envFrom
kubectl apply -f pod.yaml

# Verify
kubectl exec myapp -- env | grep ENV
```

### Pattern 2: Update ConfigMap and restart pods
```bash
# Update ConfigMap
kubectl edit cm app-config

# Restart deployment to pick up changes
kubectl rollout restart deployment myapp
kubectl rollout status deployment myapp
```

### Pattern 3: Create Secret and mount as volume
```bash
# Create secret
kubectl create secret generic db-creds --from-literal=password=secret

# Generate pod YAML
kubectl run db-client --image=mysql --dry-run=client -o yaml > pod.yaml

# Edit to add volume mount:
# volumes:
# - name: secret-volume
#   secret:
#     secretName: db-creds
# volumeMounts:
# - name: secret-volume
#   mountPath: /etc/secrets
#   readOnly: true

kubectl apply -f pod.yaml

# Verify
kubectl exec db-client -- ls -la /etc/secrets
```

### Pattern 4: Pod with SecurityContext
```bash
kubectl run secure-pod --image=nginx --dry-run=client -o yaml > pod.yaml

# Add securityContext in YAML:
# spec:
#   securityContext:
#     runAsUser: 1000
#     runAsNonRoot: true
#     fsGroup: 2000

kubectl apply -f pod.yaml
kubectl exec secure-pod -- id  # Verify user
```

## Quick Tips

1. **ConfigMaps and Secrets are namespace-scoped**
2. **Secrets are base64 encoded, NOT encrypted** (use encryption at rest)
3. **ConfigMap/Secret changes don't auto-update pods** (need restart)
4. **Mounted ConfigMaps/Secrets update ~60s after change** (not env vars)
5. **Use `defaultMode` to set file permissions** (default 0644 for ConfigMap, 0400 for Secret recommended)
6. **ServiceAccount tokens are auto-mounted** at `/var/run/secrets/kubernetes.io/serviceaccount/`
7. **Resource Quotas require requests/limits** on all pods
8. **LimitRanges set defaults** if not specified

## Troubleshooting

```bash
# ConfigMap not found
kubectl get cm  # List all
kubectl get cm -n <namespace>  # Check correct namespace

# Secret not decoding properly
kubectl get secret <name> -o jsonpath='{.data.key}' | base64 -d

# Pod not picking up ConfigMap changes
kubectl rollout restart deployment <name>

# Permission denied in pod
kubectl get pod <pod> -o jsonpath='{.spec.securityContext}'
kubectl exec <pod> -- id

# Resource quota exceeded
kubectl describe quota -n <namespace>
kubectl get pods -n <namespace> -o jsonpath='{.items[*].spec.containers[*].resources}'
```
