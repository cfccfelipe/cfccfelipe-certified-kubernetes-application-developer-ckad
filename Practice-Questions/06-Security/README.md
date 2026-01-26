# Security - Practice Questions

## Question 1: Configure SecurityContext for Pod

**Scenario:** Run a pod with specific security settings to follow principle of least privilege.

**Tasks:**
1. Create a pod named `secure-app` with `nginx:1.21` image
2. Configure pod-level SecurityContext:
   - Run as user ID `1000`
   - Run as group ID `3000`
   - `fsGroup: 2000` for volume permissions
   - `runAsNonRoot: true`
3. Verify the security settings inside the container
4. Check that the pod runs as non-root user
5. Verify file permissions with fsGroup

**Solution:** See `solution-01.md`

---

## Question 2: Container SecurityContext - Drop Capabilities

**Scenario:** Harden a container by dropping unnecessary Linux capabilities.

**Tasks:**
1. Create a pod with container-level SecurityContext
2. Drop ALL capabilities: `drop: ["ALL"]`
3. Add only required capability: `add: ["NET_BIND_SERVICE"]`
4. Set `allowPrivilegeEscalation: false`
5. Verify the container cannot escalate privileges
6. Test that container can bind to port 80 but has minimal capabilities

**Files:** `secure-container.yaml`
**Solution:** See `solution-02.md`

---

## Question 3: ServiceAccount and RBAC - Role Binding

**Scenario:** Create a ServiceAccount with specific permissions using RBAC.

**Tasks:**
1. Create a ServiceAccount named `pod-reader`
2. Create a Role that allows:
   - Get, list, watch pods
   - In the default namespace
3. Create a RoleBinding linking the Role to the ServiceAccount
4. Create a pod using this ServiceAccount
5. Test permissions from inside the pod using kubectl
6. Verify the pod cannot perform unauthorized operations

**Solution:** See `solution-03.md`

---

## Question 4: ClusterRole and ClusterRoleBinding

**Scenario:** Grant cluster-wide permissions to a ServiceAccount.

**Tasks:**
1. Create a ServiceAccount named `node-viewer`
2. Create a ClusterRole allowing:
   - Get, list nodes cluster-wide
   - Get, list persistentvolumes
3. Create a ClusterRoleBinding
4. Create a pod with this ServiceAccount
5. Verify it can list nodes across all namespaces
6. Verify it cannot create or delete resources

**Solution:** See `solution-04.md`

---

## Question 5: Secret Security - Mount with Specific Permissions

**Scenario:** Mount secrets with restrictive file permissions.

**Tasks:**
1. Create a Secret with sensitive data
2. Mount secret as volume (not environment variables)
3. Set `defaultMode: 0400` (read-only by owner)
4. Verify files are created with correct permissions
5. Container runs as specific user to access secret
6. Explain why this is more secure than environment variables

**Files:** `secret-permissions.yaml`
**Solution:** See `solution-05.md`

---

## Question 6: NetworkPolicy for Security - Deny All by Default

**Scenario:** Implement zero-trust network security using NetworkPolicies.

**Tasks:**
1. Create a deployment named `secure-backend`
2. Create NetworkPolicy denying all ingress traffic
3. Create second NetworkPolicy allowing traffic only from pods with label `role=frontend`
4. Allow only port 8080
5. Test that unauthorized pods cannot connect
6. Verify frontend pods can connect on port 8080 only

**Files:** `network-security-policy.yaml`
**Solution:** See `solution-06.md`

---

## Question 7: Read-Only Root Filesystem

**Scenario:** Prevent container from writing to root filesystem.

**Tasks:**
1. Create a pod with `readOnlyRootFilesystem: true`
2. Application needs /tmp for temporary files
3. Mount emptyDir volume at `/tmp`
4. Verify container cannot write to `/etc`, `/var`, etc.
5. Verify container can write to `/tmp`
6. Understand security benefits of read-only filesystem

**Solution:** See `solution-07.md`

---

## Question 8: Scan Image for Vulnerabilities

**Scenario:** Check container image for known vulnerabilities before deployment.

**Tasks:**
1. Use a scanning tool (trivy, or image scanning in registry)
2. Scan `nginx:1.19` image
3. Review vulnerabilities found
4. Update to newer image with fewer vulnerabilities
5. Document the importance of regular scanning
6. Understand CVE severity levels

**Solution:** See `solution-08.md`

---

## Question 9: ServiceAccount Token Projection

**Scenario:** Use projected ServiceAccount tokens with audience and expiration.

**Tasks:**
1. Create pod with projected ServiceAccount token
2. Set token audience and expiration time
3. Token auto-rotates before expiration
4. More secure than legacy tokens
5. Verify token projection in pod spec
6. Check token at `/var/run/secrets/tokens/`

---

## Question 10: Pod Security Standards - Restricted Profile

**Scenario:** Apply restricted Pod Security Standard to namespace.

**Tasks:**
1. Label namespace with `pod-security.kubernetes.io/enforce: restricted`
2. Try to create pod with privileged settings (will fail)
3. Create compliant pod:
   - runAsNonRoot: true
   - Drop all capabilities
   - readOnlyRootFilesystem: true
   - seccompProfile: RuntimeDefault
4. Verify pod is accepted
5. Understand baseline vs restricted profiles

---

## Question 11: RBAC - Prevent Secret Access

**Scenario:** Create role that allows pod management but not secret access.

**Tasks:**
1. Create Role allowing: get, list, create, delete pods
2. Explicitly deny access to secrets (by not including it)
3. Create ServiceAccount and RoleBinding
4. Test that ServiceAccount can manage pods
5. Verify it cannot access secrets
6. Understand default deny in RBAC

---

## Question 12: SecurityContext - Privileged Container

**Scenario:** Understand risks of privileged containers.

**Tasks:**
1. Create privileged pod: `privileged: true`
2. Observe it has access to host devices
3. Can perform dangerous operations
4. Create non-privileged pod and compare
5. Understand when privileged mode is needed
6. Document security risks and alternatives

---

## Question 13: AppArmor Profile

**Scenario:** Apply AppArmor profile to container for additional security.

**Tasks:**
1. Check if AppArmor is enabled on node
2. Create pod with AppArmor annotation
3. Use profile: `runtime/default` or custom profile
4. Verify profile is loaded and active
5. Test that denied operations fail
6. Understand mandatory access control

---

## Question 14: Seccomp Profile

**Scenario:** Restrict system calls using seccomp.

**Tasks:**
1. Apply seccomp profile to pod
2. Use `type: RuntimeDefault` for default Docker profile
3. Or specify custom profile
4. Verify restricted syscalls cannot be executed
5. Application works with allowed syscalls only
6. Understand seccomp vs AppArmor

---

## Question 15: RBAC - Service Account Can Impersonate

**Scenario:** Grant permission for ServiceAccount to impersonate users.

**Tasks:**
1. Create ServiceAccount named `impersonator`
2. Create ClusterRole with `impersonate` verb on users
3. Create ClusterRoleBinding
4. Use `kubectl --as=user@example.com` to impersonate
5. Verify impersonation works
6. Understand use cases and risks

---

## Question 16: Network Policy - Egress to External IP

**Scenario:** Allow pod to connect to specific external IP only.

**Tasks:**
1. Create NetworkPolicy with egress rules
2. Allow DNS (port 53)
3. Allow connection to specific external IP: `8.8.8.8/32`
4. Deny all other egress
5. Test pod can reach allowed IP
6. Verify other external IPs are blocked

---

## Question 17: Audit Logs for Security Events

**Scenario:** Enable and review audit logs for security monitoring.

**Tasks:**
1. Check if audit logging is enabled
2. Create audit policy for security events
3. Monitor events: authentication, authorization, secret access
4. Review audit logs for suspicious activity
5. Filter logs by user, verb, resource
6. Understand importance of audit trails

---

## Question 18: Image Pull Secrets

**Scenario:** Use private container registry with authentication.

**Tasks:**
1. Create docker-registry secret with credentials
2. Reference secret in pod spec: `imagePullSecrets`
3. Kubernetes pulls image using credentials
4. Create ServiceAccount with imagePullSecrets
5. Pods using ServiceAccount inherit credentials
6. Verify image pull succeeds

---

## Question 19: Resource Quotas for Security

**Scenario:** Prevent resource exhaustion attacks using quotas.

**Tasks:**
1. Create ResourceQuota in namespace
2. Limit: 10 pods, 4 CPU cores, 8Gi memory
3. Try to create pods exceeding quota (will fail)
4. Understand how quotas prevent DoS
5. Set PodSecurityPolicy equivalent using quotas
6. Monitor quota usage

---

## Question 20: RBAC - Bind to Built-in Roles

**Scenario:** Use Kubernetes built-in roles for common permissions.

**Tasks:**
1. Use built-in ClusterRole: `view`
2. Create RoleBinding to grant read-only access
3. Use `edit` ClusterRole for developers
4. Use `admin` ClusterRole for namespace admins
5. Use `cluster-admin` only when necessary
6. Understand principle of least privilege

---

## Question 21: Secure etcd - Secrets Encryption

**Scenario:** Ensure secrets are encrypted at rest in etcd.

**Tasks:**
1. Check if encryption is configured
2. Create EncryptionConfiguration
3. Encrypt secrets using AES
4. Verify secrets are encrypted in etcd
5. Rotate encryption keys
6. Understand defense in depth

---

## Question 22: Pod Security Admission - Warn and Audit

**Scenario:** Use Pod Security Admission in warn/audit mode before enforcing.

**Tasks:**
1. Label namespace: `pod-security.kubernetes.io/warn: restricted`
2. Label namespace: `pod-security.kubernetes.io/audit: restricted`
3. Create non-compliant pod (shows warning but succeeds)
4. Review audit logs for violations
5. Fix violations before switching to enforce mode
6. Gradual security hardening approach

---

## Question 23: Limit Container Resources to Prevent DoS

**Scenario:** Use LimitRange to enforce security boundaries.

**Tasks:**
1. Create LimitRange setting default and max resources
2. Prevent pods without resource limits
3. Set minimum resources to prevent too-small pods
4. Create pod and observe auto-applied limits
5. Try to exceed max limits (will fail)
6. Security through resource governance

---

## Question 24: Security Context - fsGroup for Volume Access

**Scenario:** Control volume file ownership with fsGroup.

**Tasks:**
1. Create PVC and mount in pod
2. Set `fsGroup: 2000` in SecurityContext
3. All files in volume owned by group 2000
4. Container runs as user 1000, group 2000
5. Verify file access with correct group ownership
6. Useful for multi-container pods sharing volumes

---

## Question 25: RBAC Troubleshooting - Auth Can I

**Scenario:** Debug RBAC permission issues.

**Tasks:**
1. User reports unable to delete pods
2. Use `kubectl auth can-i delete pods --as=user@example.com`
3. Check Role and RoleBinding
4. Identify missing permission
5. Update Role to grant delete permission
6. Verify with `auth can-i` again

---

## Security Concepts

### SecurityContext Levels

| Level | Scope | Use Case |
|-------|-------|----------|
| **Pod-level** | All containers in pod | Set fsGroup, runAsUser for all |
| **Container-level** | Specific container | Override pod settings, drop capabilities |
| **Both** | Combined | Pod sets defaults, container overrides |

### Common SecurityContext Fields

```yaml
securityContext:
  # User/Group
  runAsUser: 1000
  runAsGroup: 3000
  runAsNonRoot: true
  fsGroup: 2000

  # Capabilities
  capabilities:
    drop: ["ALL"]
    add: ["NET_BIND_SERVICE"]

  # Filesystem
  readOnlyRootFilesystem: true

  # Privilege
  privileged: false
  allowPrivilegeEscalation: false

  # Profiles
  seccompProfile:
    type: RuntimeDefault
  seLinuxOptions:
    level: "s0:c123,c456"
```

### RBAC Resources

| Resource | Scope | Purpose |
|----------|-------|---------|
| **Role** | Namespace | Permissions within namespace |
| **ClusterRole** | Cluster-wide | Cluster resources or across namespaces |
| **RoleBinding** | Namespace | Grant Role to subjects in namespace |
| **ClusterRoleBinding** | Cluster-wide | Grant ClusterRole cluster-wide |
| **ServiceAccount** | Namespace | Identity for pods |

### RBAC Example

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-reader
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: pod-reader
roleRef:
  kind: Role
  name: pod-reader-role
  apiGroup: rbac.authorization.k8s.io
```

### Pod Security Standards

| Profile | Description | Use Case |
|---------|-------------|----------|
| **Privileged** | Unrestricted | Trusted workloads, system components |
| **Baseline** | Minimally restrictive | Common containerized workloads |
| **Restricted** | Heavily restricted | Security-critical applications |

### Restricted Profile Requirements

- Run as non-root
- Drop all capabilities
- Read-only root filesystem
- seccompProfile: RuntimeDefault
- No privilege escalation
- No host namespaces
- No hostPath volumes

---

## Tips for Security

- **Always** run as non-root when possible
- **Drop ALL** capabilities, add only what's needed
- Use **read-only filesystem** when application allows
- **Mount secrets as volumes**, not environment variables
- Set **defaultMode: 0400** for secret files
- Use **ServiceAccounts** with minimal RBAC permissions
- Implement **NetworkPolicies** for pod-to-pod security
- Apply **Pod Security Standards** to namespaces
- **Scan images** regularly for vulnerabilities
- Enable **audit logging** for security events
- **Encrypt secrets** at rest in etcd
- Use **projected tokens** with expiration
- Set **resource limits** to prevent DoS
- **Never** use privileged containers in production
- Follow **principle of least privilege**

---

## Quick Command Reference

```bash
# ServiceAccounts
kubectl create serviceaccount <name>
kubectl get serviceaccounts
kubectl describe serviceaccount <name>

# RBAC
kubectl create role <name> --verb=get,list --resource=pods
kubectl create rolebinding <name> --role=<role> --serviceaccount=default:<sa>
kubectl create clusterrole <name> --verb=get,list --resource=nodes
kubectl create clusterrolebinding <name> --clusterrole=<role> --serviceaccount=default:<sa>

# Test Permissions
kubectl auth can-i get pods
kubectl auth can-i delete pods --as=user@example.com
kubectl auth can-i list nodes --as=system:serviceaccount:default:pod-reader

# View RBAC
kubectl get roles
kubectl get rolebindings
kubectl describe role <name>
kubectl describe rolebinding <name>

# Secrets
kubectl create secret generic <name> --from-literal=key=value
kubectl create secret docker-registry <name> --docker-server=<server> --docker-username=<user> --docker-password=<pass>
kubectl get secrets
kubectl describe secret <name>

# NetworkPolicies
kubectl get networkpolicies
kubectl describe networkpolicy <name>

# Pod Security
kubectl label namespace <ns> pod-security.kubernetes.io/enforce=restricted
kubectl label namespace <ns> pod-security.kubernetes.io/warn=baseline
kubectl label namespace <ns> pod-security.kubernetes.io/audit=restricted

# Security Context Testing
kubectl exec -it <pod> -- id
kubectl exec -it <pod> -- whoami
kubectl exec -it <pod> -- ls -la /
kubectl exec -it <pod> -- cat /proc/1/status | grep Cap
```

---

## Common Security Patterns

### Non-Root User with Read-Only Filesystem

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
volumeMounts:
- name: tmp
  mountPath: /tmp
volumes:
- name: tmp
  emptyDir: {}
```

### Dropped Capabilities

```yaml
securityContext:
  capabilities:
    drop:
    - ALL
    add:
    - NET_BIND_SERVICE
  allowPrivilegeEscalation: false
```

### Secret with Restrictive Permissions

```yaml
volumes:
- name: secret-volume
  secret:
    secretName: my-secret
    defaultMode: 0400
    items:
    - key: password
      path: db-password
```

### NetworkPolicy - Default Deny

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### Projected ServiceAccount Token

```yaml
volumes:
- name: token
  projected:
    sources:
    - serviceAccountToken:
        path: token
        expirationSeconds: 3600
        audience: api
```

---

## References

- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [SecurityContext](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Securing a Cluster](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/)
- [Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
