# Configuration - Practice Questions

## Question 1: Create Secret from Hardcoded Variables and Update Deployment

**Scenario:** A deployment has hardcoded database credentials that need to be moved to a Secret.

**Tasks:**
1. Review the deployment in `app-deployment.yaml` which has hardcoded credentials
2. Create a Secret named `db-credentials` with:
   - `DB_USER=admin` YWRtaW4K
   - `DB_PASS=SecurePassword123!` U2VjdXJlUGFzc3dvcmQxMjMhCg==
1. Update the deployment to use `valueFrom.secretKeyRef` instead of hardcoded values
2. Verify the deployment rolled out successfully
3. Verify the environment variables in the pod

**Files:** `app-deployment.yaml`
**Solution:** See `solution-01.md`

---

## Question 2: Create ConfigMap from File and Mount as Volume

**Scenario:** You need to provide configuration files to an application via ConfigMap.

**Tasks:**
1. Create a configuration file `app-config.properties` with the following content:
   ```
   database.url=jdbc:mysql://mysql:3306/mydb
   database.driver=com.mysql.jdbc.Driver
   cache.enabled=true
   cache.ttl=3600
   ```
2. Create a ConfigMap named `app-config` from this file
3. Create a pod named `config-pod` using `nginx:1.21` image
4. Mount the ConfigMap as a volume at `/etc/config`
5. Verify the file exists inside the pod

**Solution:** See `solution-02.md`

---

## Question 3: SecurityContext - Run as Non-Root User

**Scenario:** Security policy requires all pods to run as non-root users.

**Tasks:**
1. Create a pod named `secure-pod` using `nginx:1.21` image
2. Set the pod to run as user ID `1000` and group ID `3000`
3. Make the filesystem read-only
4. Add a writable volume at `/tmp` for temporary files
5. Verify the security settings using `kubectl exec`

**Solution:** See `solution-03.md`

---

## Question 4: Resource Quotas and LimitRange

**Scenario:** Create a namespace with resource quotas and limits.

**Tasks:**
1. Create a namespace named `limited-namespace`
2. Create a ResourceQuota that limits:
   - Total CPU requests: `1000m`
   - Total memory requests: `1Gi`
   - Total pods: `5`
3. Create a LimitRange that sets:
   - Default CPU limit: `200m`
   - Default memory limit: `256Mi`
   - Min CPU request: `50m`
   - Min memory request: `64Mi`
4. Try to create pods that violate these limits
5. Document the behavior

**Files:** `resourcequota.yaml`, `limitrange.yaml`
**Solution:** See `solution-04.md`

---

## Question 5: ServiceAccount with Secret Token

**Scenario:** Create a ServiceAccount and use it in a pod to access the Kubernetes API.

**Tasks:**
1. Create a ServiceAccount named `api-service-account`
2. Create a pod named `api-client` using `curlimages/curl:8.1.0` image
3. Assign the ServiceAccount to the pod
4. Command: `sleep 3600`
5. Verify the ServiceAccount token is mounted
6. Use curl to test API access from inside the pod

**Solution:** See `solution-05.md`

---

## Question 6: Update ConfigMap and Trigger Rolling Update

**Scenario:** An application's configuration needs to be updated without recreating pods.

**Tasks:**
1. Apply the deployment from `configmap-deployment.yaml`
2. The deployment uses a ConfigMap named `app-settings`
3. Update the ConfigMap to change a configuration value
4. Trigger a rolling update of the deployment
5. Verify pods are using the new configuration

**Files:** `configmap-deployment.yaml`
**Solution:** See `solution-06.md`

---

## Question 7: Create Docker Registry Secret and Use in Pod

**Scenario:** Pull images from a private Docker registry.

**Tasks:**
1. Create a docker-registry secret named `regcred`
2. Use credentials: server=`docker.io`, username=`myuser`, password=`mypass`, email=`user@example.com`
3. Create a pod that uses this secret in `imagePullSecrets`
4. Verify the secret is correctly configured
5. Test with a private image reference

---

## Question 8: SecurityContext - RunAsNonRoot and Capabilities

**Scenario:** Run a container with restricted security settings.

**Tasks:**
1. Create a pod that runs as non-root user
2. Set `runAsNonRoot: true` and `runAsUser: 1000`
3. Drop all capabilities and add only `NET_BIND_SERVICE`
4. Set container filesystem to read-only
5. Add writable emptyDir volume at `/tmp`
6. Verify security settings with `kubectl exec`

---

## Question 9: Multiple ConfigMaps and Secrets in One Pod

**Scenario:** A pod needs multiple configuration sources.

**Tasks:**
1. Create ConfigMap `app-config` with app settings
2. Create ConfigMap `feature-flags` with feature toggles
3. Create Secret `db-creds` with database credentials
4. Create Secret `api-keys` with API keys
5. Mount all as environment variables in a single pod
6. Verify all variables are available

---

## Question 10: ConfigMap with Binary Data

**Scenario:** Store binary files in ConfigMap.

**Tasks:**
1. Create a ConfigMap from a binary file (e.g., image or certificate)
2. Use `--from-file` with binary data
3. Mount the ConfigMap as a volume
4. Verify the binary file is correctly mounted
5. Check file permissions and content

---

## Question 11: ServiceAccount with Custom Secrets

**Scenario:** Create a ServiceAccount with additional secrets.

**Tasks:**
1. Create ServiceAccount `custom-sa`
2. Create a secret `custom-token` manually
3. Annotate the ServiceAccount to use this secret
4. Create a pod using this ServiceAccount
5. Verify the custom secret is mounted
6. Check token location: `/var/run/secrets/kubernetes.io/serviceaccount/`

---

## Question 12: Resource Quota - Exceed and Fix

**Scenario:** A namespace has resource quotas that are being exceeded.

**Tasks:**
1. Create namespace `limited` with ResourceQuota
2. Set limits: `requests.cpu=500m`, `requests.memory=512Mi`, `pods=3`
3. Try to create 4 pods (should fail)
4. Check quota usage with `kubectl describe quota`
5. Delete a pod and create another
6. Verify quota is enforced

---

## Question 13: LimitRange with Default Requests and Limits

**Scenario:** Set default resources for all pods in a namespace.

**Tasks:**
1. Create namespace `defaults`
2. Create LimitRange with default requests and limits
3. Create a pod without specifying resources
4. Verify default resources are applied
5. Create another pod with custom resources
6. Verify custom resources override defaults

---

## Question 14: SecurityContext - fsGroup for Shared Volumes

**Scenario:** Multiple containers need to write to a shared volume.

**Tasks:**
1. Create a pod with two containers sharing an emptyDir volume
2. Set `fsGroup: 2000` at pod level
3. Both containers should be able to write to the shared volume
4. Verify file ownership in the shared directory
5. Test writing from both containers

---

## Question 15: Environment Variables from All Sources

**Scenario:** Use all env variable sources in one pod.

**Tasks:**
1. Set literal env var: `APP_ENV=production`
2. Use env from ConfigMap key: `CONFIG_KEY`
3. Use env from Secret key: `SECRET_KEY`
4. Use fieldRef: `POD_NAME` from `metadata.name`
5. Use resourceFieldRef: `CPU_LIMIT` from container limits
6. Verify all environment variables in the pod

---

## Question 16: Secret with Specific File Permissions

**Scenario:** Mount secrets with restricted file permissions.

**Tasks:**
1. Create a secret `secure-data` with sensitive information
2. Mount as volume with `defaultMode: 0400` (read-only for owner)
3. Create a pod that mounts this secret
4. Verify file permissions are correctly set
5. Try to modify the file (should fail)
6. Check owner and group of the mounted files

---

## Question 17: ConfigMap from Multiple Files

**Scenario:** Create a ConfigMap from a directory of config files.

**Tasks:**
1. Create directory with multiple config files: `app.conf`, `db.conf`, `cache.conf`
2. Create ConfigMap from directory: `kubectl create cm multi-config --from-file=config-dir/`
3. Verify all files are in the ConfigMap
4. Mount entire ConfigMap as volume
5. Verify all files are available in the pod

---

## Question 18: Immutable ConfigMap and Secret

**Scenario:** Prevent accidental changes to critical configuration.

**Tasks:**
1. Create ConfigMap with `immutable: true`
2. Create Secret with `immutable: true`
3. Try to update them (should fail)
4. Understand when to use immutable configuration
5. Create new versions instead of updating

---

## Question 19: Pod Security Standards

**Scenario:** Apply Pod Security Standards to a namespace.

**Tasks:**
1. Create namespace `secure-ns`
2. Label it with `pod-security.kubernetes.io/enforce: restricted`
3. Try to create a privileged pod (should fail)
4. Create a compliant pod with restricted security context
5. Verify pod security standards are enforced

---

## Question 20: ServiceAccount Token Projection

**Scenario:** Use projected ServiceAccount tokens with audience.

**Tasks:**
1. Create a pod with projected ServiceAccount token volume
2. Set custom audience and expiration
3. Mount token at custom path
4. Verify token properties
5. Check token expiration and audience

---

## Tips for Configuration

- Secrets are base64 encoded, not encrypted at rest (unless configured)
- ConfigMaps and Secrets can be consumed as:
  - Environment variables
  - Command-line arguments
  - Files in a volume
- Use `kubectl create secret generic` or `kubectl create configmap` for quick creation
- SecurityContext can be set at pod level or container level
- Resource quotas are per namespace
- Use `kubectl describe quota` to check quota usage

---

## Quick Command Reference

```bash
# ConfigMaps
kubectl create configmap <name> --from-literal=key=value
kubectl create configmap <name> --from-file=file.txt
kubectl create configmap <name> --from-file=config-dir/

# Secrets
kubectl create secret generic <name> --from-literal=key=value
kubectl create secret generic <name> --from-file=ssh-key=~/.ssh/id_rsa
kubectl create secret docker-registry <name> --docker-server=DOCKER_SERVER --docker-username=USER --docker-password=PASSWORD

# ServiceAccounts
kubectl create serviceaccount <name>

# Resource Quotas
kubectl create quota <name> --hard=cpu=1,memory=1Gi,pods=2

# Check resource usage
kubectl describe quota -n <namespace>
kubectl top pods -n <namespace>
```

---

## References

- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Limit Ranges](https://kubernetes.io/docs/concepts/policy/limit-range/)
- [Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
