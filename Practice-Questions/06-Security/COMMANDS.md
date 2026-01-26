# Security - Essential Commands

## ServiceAccount Commands

```bash
# Create ServiceAccount
kubectl create serviceaccount <serviceaccount-name>
kubectl create sa <name>  # Short form

# List ServiceAccounts
kubectl get serviceaccounts
kubectl get sa

# Describe ServiceAccount
kubectl describe serviceaccount <name>
kubectl describe sa <name>

# Delete ServiceAccount
kubectl delete serviceaccount <name>

# Get ServiceAccount YAML
kubectl get sa <name> -o yaml

# Create pod with specific ServiceAccount
kubectl run <pod-name> --image=<image> --serviceaccount=<sa-name> --dry-run=client -o yaml
```

---

## RBAC - Role Commands

```bash
# Create Role
kubectl create role <role-name> --verb=<verb> --resource=<resource>
kubectl create role pod-reader --verb=get,list,watch --resource=pods
kubectl create role pod-manager --verb=get,list,create,delete --resource=pods

# Create Role with specific API group
kubectl create role deployment-manager --verb=get,list,create --resource=deployments --resource-name=<name>

# List Roles
kubectl get roles
kubectl get role <name>

# Describe Role
kubectl describe role <name>

# Delete Role
kubectl delete role <name>

# Get Role YAML
kubectl get role <name> -o yaml

# Edit Role
kubectl edit role <name>
```

### Role YAML Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]  # "" indicates core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
```

---

## RBAC - RoleBinding Commands

```bash
# Create RoleBinding
kubectl create rolebinding <binding-name> --role=<role-name> --serviceaccount=<namespace>:<sa-name>
kubectl create rolebinding read-pods --role=pod-reader --serviceaccount=default:pod-reader-sa

# Bind to User
kubectl create rolebinding <name> --role=<role> --user=<user-email>

# Bind to Group
kubectl create rolebinding <name> --role=<role> --group=<group-name>

# List RoleBindings
kubectl get rolebindings
kubectl get rolebinding <name>

# Describe RoleBinding
kubectl describe rolebinding <name>

# Delete RoleBinding
kubectl delete rolebinding <name>
```

### RoleBinding YAML Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: pod-reader-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## RBAC - ClusterRole Commands

```bash
# Create ClusterRole
kubectl create clusterrole <name> --verb=<verb> --resource=<resource>
kubectl create clusterrole node-reader --verb=get,list --resource=nodes
kubectl create clusterrole pv-reader --verb=get,list --resource=persistentvolumes

# ClusterRole for all resources (cluster-admin equivalent)
kubectl create clusterrole full-access --verb='*' --resource='*'

# List ClusterRoles
kubectl get clusterroles
kubectl get clusterrole <name>

# Describe ClusterRole
kubectl describe clusterrole <name>

# View built-in ClusterRoles
kubectl get clusterrole view -o yaml
kubectl get clusterrole edit -o yaml
kubectl get clusterrole admin -o yaml
kubectl get clusterrole cluster-admin -o yaml
```

### ClusterRole YAML Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "list"]
```

---

## RBAC - ClusterRoleBinding Commands

```bash
# Create ClusterRoleBinding
kubectl create clusterrolebinding <name> --clusterrole=<clusterrole> --serviceaccount=<namespace>:<sa>
kubectl create clusterrolebinding node-reader-binding --clusterrole=node-reader --serviceaccount=default:node-reader-sa

# Bind to User
kubectl create clusterrolebinding <name> --clusterrole=<role> --user=<user-email>

# List ClusterRoleBindings
kubectl get clusterrolebindings
kubectl get clusterrolebinding <name>

# Describe ClusterRoleBinding
kubectl describe clusterrolebinding <name>
```

---

## RBAC - Testing Permissions

```bash
# Check if you can perform an action
kubectl auth can-i create deployments
kubectl auth can-i delete pods
kubectl auth can-i get nodes

# Check permissions for another user
kubectl auth can-i get pods --as=user@example.com
kubectl auth can-i delete pods --as=user@example.com --namespace=production

# Check permissions for ServiceAccount
kubectl auth can-i list secrets --as=system:serviceaccount:default:my-sa
kubectl auth can-i create pods --as=system:serviceaccount:default:my-sa

# List all permissions for current user
kubectl auth can-i --list

# Check permissions in specific namespace
kubectl auth can-i get pods --namespace=production --as=user@example.com
```

---

## Secret Commands (Security Focus)

```bash
# Create Secret
kubectl create secret generic <name> --from-literal=key=value
kubectl create secret generic db-creds --from-literal=username=admin --from-literal=password=pass123

# Create Secret from file
kubectl create secret generic <name> --from-file=<key>=<filepath>
kubectl create secret generic ssh-key --from-file=ssh-privatekey=~/.ssh/id_rsa

# Create Docker Registry Secret
kubectl create secret docker-registry <name> \
  --docker-server=<server> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>

# List Secrets (doesn't show values)
kubectl get secrets
kubectl describe secret <name>

# View Secret data (base64 encoded)
kubectl get secret <name> -o jsonpath='{.data}'

# Decode Secret value
kubectl get secret <name> -o jsonpath='{.data.password}' | base64 --decode

# Delete Secret
kubectl delete secret <name>
```

---

## NetworkPolicy Commands

```bash
# Create NetworkPolicy (from YAML)
kubectl apply -f networkpolicy.yaml

# List NetworkPolicies
kubectl get networkpolicies
kubectl get netpol  # Short form

# Describe NetworkPolicy
kubectl describe networkpolicy <name>
kubectl describe netpol <name>

# Delete NetworkPolicy
kubectl delete networkpolicy <name>

# Get NetworkPolicy YAML
kubectl get networkpolicy <name> -o yaml
```

### NetworkPolicy - Deny All Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

---

## SecurityContext Testing Commands

```bash
# Check user inside container
kubectl exec -it <pod-name> -- id
kubectl exec -it <pod-name> -- whoami

# Check process capabilities
kubectl exec -it <pod-name> -- cat /proc/1/status | grep Cap

# Check if running as root
kubectl exec -it <pod-name> -- ps aux

# Check file permissions
kubectl exec -it <pod-name> -- ls -la /
kubectl exec -it <pod-name> -- ls -la /var/run/secrets/

# Test filesystem write access
kubectl exec -it <pod-name> -- touch /test-file
kubectl exec -it <pod-name> -- touch /tmp/test-file

# Check mounted volumes permissions
kubectl exec -it <pod-name> -- ls -la /mnt/volume
```

---

## Pod Security Standards

```bash
# Label namespace with Pod Security Standard
kubectl label namespace <namespace> pod-security.kubernetes.io/enforce=restricted
kubectl label namespace <namespace> pod-security.kubernetes.io/warn=baseline
kubectl label namespace <namespace> pod-security.kubernetes.io/audit=restricted

# View namespace labels
kubectl get namespace <namespace> --show-labels

# Remove Pod Security label
kubectl label namespace <namespace> pod-security.kubernetes.io/enforce-
```

### Pod Security Levels
- **privileged**: Unrestricted (default if not set)
- **baseline**: Minimally restrictive
- **restricted**: Heavily restricted, security best practices

### Pod Security Modes
- **enforce**: Reject non-compliant pods
- **audit**: Log violations but allow pod
- **warn**: Show warning but allow pod

---

## SecurityContext - Pod Creation Examples

### Non-Root User

```bash
kubectl run secure-pod --image=nginx:1.21 --dry-run=client -o yaml > pod.yaml
# Edit pod.yaml to add:
#   securityContext:
#     runAsUser: 1000
#     runAsNonRoot: true
kubectl apply -f pod.yaml
```

### Read-Only Root Filesystem

```yaml
securityContext:
  readOnlyRootFilesystem: true
volumeMounts:
- name: tmp
  mountPath: /tmp
volumes:
- name: tmp
  emptyDir: {}
```

### Drop All Capabilities

```yaml
securityContext:
  capabilities:
    drop:
    - ALL
  allowPrivilegeEscalation: false
```

---

## Image Security Commands

```bash
# Pull image for scanning
docker pull <image-name>

# Scan image with trivy (if installed)
trivy image <image-name>
trivy image nginx:1.21

# Check image digest
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[0].imageID}'

# Use specific image digest (immutable)
kubectl run pod --image=nginx@sha256:<digest>
```

---

## Resource Quota and LimitRange (Security)

```bash
# Create ResourceQuota
kubectl create quota <name> --hard=pods=10,cpu=4,memory=8Gi

# List ResourceQuota
kubectl get resourcequota
kubectl describe resourcequota <name>

# Create LimitRange (from YAML)
kubectl apply -f limitrange.yaml

# List LimitRange
kubectl get limitrange
kubectl describe limitrange <name>

# Check quota usage
kubectl describe resourcequota <name>
```

---

## Audit and Monitoring

```bash
# View events (security-related)
kubectl get events --sort-by=.metadata.creationTimestamp

# Check pod security events
kubectl get events --field-selector involvedObject.kind=Pod

# View API server audit logs (if enabled)
# Logs location varies: /var/log/kubernetes/audit.log or check control plane

# Monitor failed authentication
kubectl get events --field-selector reason=FailedAuthentication
```

---

## Debug Security Issues

```bash
# Check why pod is not starting (security context issues)
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --field-selector involvedObject.name=<pod-name>

# Check RBAC permission issues
kubectl auth can-i <verb> <resource> --as=<user>

# Test ServiceAccount permissions
kubectl auth can-i list pods --as=system:serviceaccount:default:<sa-name>

# Check NetworkPolicy blocking traffic
kubectl exec -it <pod> -- curl http://<service-name>
kubectl exec -it <pod> -- nc -zv <service-name> <port>
```

---

## Complete Security Example

```bash
# 1. Create ServiceAccount
kubectl create serviceaccount secure-app-sa

# 2. Create Role
kubectl create role pod-reader --verb=get,list --resource=pods

# 3. Create RoleBinding
kubectl create rolebinding read-pods --role=pod-reader --serviceaccount=default:secure-app-sa

# 4. Create Secret
kubectl create secret generic app-secret --from-literal=api-key=secret123

# 5. Create Pod with security settings
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  serviceAccountName: secure-app-sa
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx:1.21
    securityContext:
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: tmp
    emptyDir: {}
  - name: secret-volume
    secret:
      secretName: app-secret
      defaultMode: 0400
EOF

# 6. Test security
kubectl exec -it secure-app -- id
kubectl exec -it secure-app -- ls -la /etc/secrets
kubectl auth can-i list pods --as=system:serviceaccount:default:secure-app-sa
```

---

## Quick Security Checklist

```bash
# ✅ Run as non-root
runAsNonRoot: true
runAsUser: 1000

# ✅ Drop all capabilities
capabilities:
  drop: ["ALL"]

# ✅ Read-only filesystem
readOnlyRootFilesystem: true

# ✅ No privilege escalation
allowPrivilegeEscalation: false

# ✅ Seccomp profile
seccompProfile:
  type: RuntimeDefault

# ✅ Use ServiceAccount with minimal RBAC
serviceAccountName: <sa-name>

# ✅ Mount secrets as volumes with restrictive permissions
defaultMode: 0400

# ✅ Apply NetworkPolicies
# ✅ Set resource limits
# ✅ Scan images for vulnerabilities
```

---

## References

- [RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [SecurityContext](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
