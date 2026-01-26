# Solution: Question 3 - Create Pod with Environment Variables

## Step 1: Create the pod with environment variables

```bash
kubectl run env-pod \
  --image=busybox:1.36 \
  --env="DB_HOST=mysql-service" \
  --env="DB_PORT=3306" \
  --env="APP_ENV=production" \
  --command -- sleep 3600
```

## Step 2: Verify the pod is running

```bash
kubectl get pod env-pod
```

Expected output:
```
NAME      READY   STATUS    RESTARTS   AGE
env-pod   1/1     Running   0          5s
```

## Step 3: Verify environment variables inside the pod

```bash
kubectl exec env-pod -- env | grep -E 'DB_|APP_'
```

Expected output:
```
DB_HOST=mysql-service
DB_PORT=3306
APP_ENV=production
```

## Step 4: Check individual environment variable

```bash
kubectl exec env-pod -- sh -c 'echo $DB_HOST'
kubectl exec env-pod -- sh -c 'echo $DB_PORT'
kubectl exec env-pod -- sh -c 'echo $APP_ENV'
```

## Alternative: Using YAML file

Create `env-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
  namespace: default
spec:
  containers:
  - name: env-pod
    image: busybox:1.36
    command: ["sleep", "3600"]
    env:
    - name: DB_HOST
      value: mysql-service
    - name: DB_PORT
      value: "3306"
    - name: APP_ENV
      value: production
  restartPolicy: Always
```

Apply it:
```bash
kubectl apply -f env-pod.yaml
```

## Step 5: View all environment variables

```bash
kubectl exec env-pod -- env
```

This shows all environment variables including Kubernetes injected ones.

## Step 6: Interactive shell to test

```bash
kubectl exec -it env-pod -- sh
```

Inside the container:
```sh
echo $DB_HOST
echo $DB_PORT
echo $APP_ENV
exit
```

## Step 7: Verify using describe

```bash
kubectl describe pod env-pod
```

Look for the Environment section:
```
Environment:
  DB_HOST:   mysql-service
  DB_PORT:   3306
  APP_ENV:   production
```

## Step 8: Get environment variables using jsonpath

```bash
kubectl get pod env-pod -o jsonpath='{.spec.containers[0].env[*].name}'
kubectl get pod env-pod -o jsonpath='{.spec.containers[0].env[*].value}'
```

## Cleanup

```bash
kubectl delete pod env-pod
```

## Key Takeaways

- Use `--env` flag with `kubectl run` to set environment variables
- Use `--command` to override the default entrypoint
- Environment variables are defined in the `env` section of the container spec
- Each env variable has a `name` and `value` field
- Use `kubectl exec` to verify environment variables inside the running container
- Numeric values in YAML should be quoted (e.g., `"3306"`)
- In CKAD, you'll often need to set env vars from ConfigMaps and Secrets (covered in Configuration section)

## Common Environment Variable Patterns

1. **Hardcoded values** (this question)
2. **From ConfigMap**: `valueFrom.configMapKeyRef`
3. **From Secret**: `valueFrom.secretKeyRef`
4. **From Field**: `valueFrom.fieldRef` (e.g., pod name, namespace)
5. **From Resource**: `valueFrom.resourceFieldRef` (e.g., CPU limit)
