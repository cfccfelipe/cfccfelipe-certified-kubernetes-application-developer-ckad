# Solution: Create Secret from Hardcoded Variables and Update Deployment

## Step 1: Review the deployment

```bash
kubectl get deployment app-deployment -o yaml
```

Notice the hardcoded credentials:
```yaml
env:
- name: DB_USER
  value: "admin"
- name: DB_PASS
  value: "SecurePassword123!"
```

## Step 2: Create the Secret

```bash
kubectl create secret generic db-credentials \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASS=SecurePassword123!
```

Verify:
```bash
kubectl get secret db-credentials
kubectl describe secret db-credentials
```

## Step 3: Update the deployment

Get the current deployment YAML:
```bash
kubectl get deployment app-deployment -o yaml > app-deployment-updated.yaml
```

Edit the file and replace the hardcoded env vars with secretKeyRef:
```yaml
env:
- name: DB_USER
  valueFrom:
    secretKeyRef:
      name: db-credentials
      key: DB_USER
- name: DB_PASS
  valueFrom:
    secretKeyRef:
      name: db-credentials
      key: DB_PASS
- name: DB_HOST
  value: "mysql-service.default.svc.cluster.local"
- name: DB_PORT
  value: "3306"
```

Apply the changes:
```bash
kubectl apply -f app-deployment-updated.yaml
```

## Step 4: Verify the rollout

```bash
kubectl rollout status deployment app-deployment
kubectl get pods -l app=backend
```

## Step 5: Verify environment variables in pod

Get a pod name:
```bash
POD=$(kubectl get pods -l app=backend -o jsonpath='{.items[0].metadata.name}')
```

Check environment variables:
```bash
kubectl exec $POD -- env | grep DB_
```

Expected output:
```
DB_USER=admin
DB_PASS=SecurePassword123!
DB_HOST=mysql-service.default.svc.cluster.local
DB_PORT=3306
```

## Alternative: Patch method

```bash
kubectl patch deployment app-deployment --type='json' -p='[
  {
    "op": "replace",
    "path": "/spec/template/spec/containers/0/env/0",
    "value": {
      "name": "DB_USER",
      "valueFrom": {
        "secretKeyRef": {
          "name": "db-credentials",
          "key": "DB_USER"
        }
      }
    }
  },
  {
    "op": "replace",
    "path": "/spec/template/spec/containers/0/env/1",
    "value": {
      "name": "DB_PASS",
      "valueFrom": {
        "secretKeyRef": {
          "name": "db-credentials",
          "key": "DB_PASS"
        }
      }
    }
  }
]'
```

## Cleanup

```bash
kubectl delete deployment app-deployment
kubectl delete secret db-credentials
```

## Key Takeaways

- Never hardcode credentials in deployment YAML
- Use Secrets for sensitive data
- `secretKeyRef` references a key in a Secret
- Updating deployment triggers rolling update
- Secrets are base64 encoded (not encrypted by default)
- Use `kubectl create secret` for quick secret creation
- Always verify the rollout completed successfully
