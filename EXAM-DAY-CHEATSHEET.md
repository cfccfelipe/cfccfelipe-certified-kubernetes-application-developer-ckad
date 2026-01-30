# CKAD Exam Day Cheatsheet

**MEMORIZE THIS FILE** - Your foundation during the 2-hour exam

---

## ⚡ First 2 Minutes - Setup

```bash
# Aliases (essential!)
alias k=kubectl
alias kg='kubectl get'
alias kd='kubectl describe'
alias kaf='kubectl apply -f'
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"

# Vim for YAML
echo "set tabstop=2 shiftwidth=2 expandtab" >> ~/.vimrc

# Set namespace (if 3+ questions in same namespace)
k config set-context --current --namespace=<namespace>
```

---

## 🚀 Resource Generation (Fast YAML)

```bash
# Pod
k run nginx --image=nginx $do > pod.yaml

# Deployment
k create deploy nginx --image=nginx --replicas=3 $do > deploy.yaml

# Service
k expose pod nginx --port=80 --target-port=8080 $do > svc.yaml
k create svc clusterip my-svc --tcp=80:8080 $do > svc.yaml

# ConfigMap
k create cm my-config --from-literal=key=value $do > cm.yaml
k create cm web-config --from-env-file=config.env $do > cm.yaml

# Secret
k create secret generic my-secret --from-literal=pass=secret $do > secret.yaml
k create secret tls cert-secret --cert=tls.crt --key=tls.key $do > secret.yaml

# Job
k create job my-job --image=busybox $do > job.yaml

# CronJob
k create cj my-cron --image=busybox --schedule="*/5 * * * *" $do > cron.yaml
```

---

## 🔍 Quick Queries

```bash
# Get all resources
k get all -n namespace

# Wide output (IPs, nodes)
k get pods -o wide

# YAML output
k get pod nginx -o yaml

# JSONPath
k get pods -o jsonpath='{.items[*].metadata.name}'

# Field selectors
k get pods --field-selector status.phase=Running

# Label selectors
k get pods -l app=nginx,tier=frontend

# Namespace specific
k get pods -n <namespace>

# Events (check admission/quota issues)
k get events --sort-by='.lastTimestamp'

# Endpoints (check service connectivity)
k get ep <service-name>

# Internal DNS test
nslookup <svc>.<ns>.svc.cluster.local
```

---

## ⚙️ Fast Modifications

```bash
# Scale
k scale deploy nginx --replicas=5

# Set image
k set image deploy/nginx nginx=nginx:1.20

# Set resources
k set resources deploy nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi

# Expose
k expose deploy nginx --port=80 --type=NodePort

# Label
k label pod nginx tier=frontend

# Annotate
k annotate pod nginx description="web server"
```

---

## 🐛 Debugging (Critical!)

```bash
# Logs
k logs pod-name
k logs pod-name -c container-name
k logs pod-name --previous  # Crashed container

# Describe (solves 80% of issues!)
k describe pod nginx

# Events
k get events --sort-by='.lastTimestamp'

# Exec into pod
k exec -it nginx -- /bin/sh

# Debug with ephemeral container
k debug nginx -it --image=busybox

# Port forward (test service locally)
k port-forward pod/nginx 8080:80

# Test DNS
k run busybox --image=busybox --rm -it --restart=Never -- nslookup kubernetes

# Test connectivity
k run curl --image=curlimages/curl --rm -it --restart=Never -- curl http://service:80
kubectl exec <pod> -- cat /etc/resolv.conf
```

---

## 📋 Troubleshooting Checklist

When something doesn't work:

- [ ] `k get pods` - Is pod Running?
- [ ] `k describe pod` - Check Events section at bottom
- [ ] `k logs pod` - Any application errors?
- [ ] `k get svc` - Service exists?
- [ ] `k describe svc` - Endpoints populated?
- [ ] `k get ep` - Endpoints match pod IPs?
- [ ] Port-forward works? - `k port-forward pod 8080:80`
- [ ] DNS works? - `nslookup service` from test pod

---

## 🔧 Debug Container Images

| Image             | Size   | Purpose                | When to Use                     |
| ----------------- | ------ | ---------------------- | ------------------------------- |
| nicolaka/netshoot | ~450MB | Full network debugging | Network issues, best all-around |
| busybox           | ~1-5MB | Minimal utilities      | Quick tests, lightweight        |
| alpine            | ~7MB   | Basic debugging        | With apk package manager        |
| curlimages/curl   | ~10MB  | HTTP testing           | API/service testing             |

**Debug example:**

```bash
k debug -it pod-name --image=nicolaka/netshoot

# Inside container:
curl localhost:8080
netstat -tulpn
nslookup service.namespace.svc.cluster.local
tcpdump -i any port 8080
```

---

## 🔄 Common Patterns

```bash
# Create and apply in one line
k run nginx --image=nginx $do | k apply -f -

# Delete immediately
k delete pod nginx $now

# Replace (when pod is immutable)
k get pod nginx -o yaml > pod.yaml
vim pod.yaml
k replace --force -f pod.yaml

# Patch (small changes)
k patch pod nginx -p '{"spec":{"containers":[{"name":"nginx","image":"nginx:1.20"}]}}'
```

---

## 🔐 RBAC (Security)

```bash
# 1. Create ServiceAccount
k create sa monitor-sa

# 2. Create Role (namespace-scoped)
k create role pod-reader --verb=get,list,watch --resource=pods

# 3. Bind Role to ServiceAccount
k create rolebinding monitor-bind --role=pod-reader --serviceaccount=default:monitor-sa

# 4. ClusterRole (cluster-wide)
k create clusterrole deploy-manager --verb=create,delete --resource=deployments

# 5. ClusterRoleBinding
k create clusterrolebinding deploy-bind --clusterrole=deploy-manager --serviceaccount=team-a:sa-name
```

---

## 💾 Storage & Scheduling

```bash
# Check PV capacity
k describe pv | grep -i capacity

# Label node for scheduling
k label node <node-name> disk=ssd

# Check quotas (if pod stays pending)
k get quota,limitrange -n <namespace>
```

---

## 📊 JSON Data Extraction

```bash
# Get all pod IPs
k get pods -o jsonpath='{.items[*].status.podIP}' > /opt/ips.txt

# Custom columns (pod name + image)
k get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image
```

---

## 📝 YAML Snippets (Copy-Paste Ready)

### Resources

```yaml
resources:
  limits:
    cpu: 200m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 256Mi
```

### SecurityContext

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  capabilities:
    drop: ['ALL']
    add: ['NET_BIND_SERVICE']
```

### Probes

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Volume Mount

```yaml
volumeMounts:
  - name: data
    mountPath: /data
volumes:
  - name: data
    emptyDir: {}
  # OR
  - name: config-vol
    configMap:
      name: my-config
  # OR
  - name: secret-vol
    secret:
      secretName: my-secret
```

### ConfigMap/Secret in Pod

```yaml
# Load all keys
envFrom:
  - configMapRef:
      name: my-config
  - secretRef:
      name: my-secret

# OR single key
env:
  - name: KEY
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: key
```

### InitContainer

```yaml
initContainers:
  - name: init
    image: busybox
    command: ['sh', '-c', 'sleep 10']
```

### Multi-Container (Sidecar)

```yaml
containers:
  - name: app
    image: nginx
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'tail -f /var/log/app.log']
```

### Node Selector

```yaml
nodeSelector:
  disk: ssd
```

### Tolerations

```yaml
tolerations:
  - key: 'key1'
    operator: 'Equal'
    value: 'value1'
    effect: 'NoSchedule'
```

---

## 🎯 Exam Strategy

1. **Read all questions** - Flag hard ones
2. **Do easy first** - 2-3 min questions to build confidence
3. **Verify everything** - `k get` after each creation
4. **Use imperative** - Generate YAML, don't write from scratch
5. **Time tracking** - 15-17 questions, ~7-8 min per question
6. **Partial credit** - Something working > nothing
7. **Don't delete unless asked** - Keep your work visible
8. **Last 15 min** - Verify all answers

---

## ⚡ Time-Savers

1. **Tab completion** - `k get po ngi<TAB>` autocompletes
2. **Up arrow** - Reuse/modify previous commands
3. **`$do`** - Always use for YAML generation
4. **`k explain`** - Faster than docs: `k explain pod.spec.containers`
5. **Copy from docs** - Search "pod example" on kubernetes.io
6. **Delete fast** - Use `$now` when told to delete
7. **Verify immediately** - After creating, always `k get` to verify
8. **Skip hard questions** - Do easy ones first, come back later

---

**🎓 YOU GOT THIS!** - Trust your practice, use imperative commands, verify everything.
