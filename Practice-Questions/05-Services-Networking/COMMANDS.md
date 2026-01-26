# Services & Networking - Essential Commands

## Service Creation

```bash
# Expose pod as service
kubectl expose pod nginx --port=80 --target-port=80 --name=nginx-service

# Expose deployment
kubectl expose deployment nginx --port=80 --type=ClusterIP

# Create NodePort service
kubectl expose deployment nginx --type=NodePort --port=80

# Create LoadBalancer service
kubectl expose deployment nginx --type=LoadBalancer --port=80

# Create service imperatively
kubectl create service clusterip my-service --tcp=80:8080

# Generate service YAML
kubectl expose deployment nginx --port=80 --dry-run=client -o yaml > service.yaml

# Create headless service
kubectl create service clusterip nginx --clusterip=None --tcp=80:80
```

## Service Management

```bash
# Get services
kubectl get services
kubectl get svc  # Short form
kubectl get svc -o wide

# Describe service
kubectl describe svc nginx-service

# Get service YAML
kubectl get svc nginx-service -o yaml

# Get service endpoints
kubectl get endpoints nginx-service
kubectl get ep nginx-service  # Short form

# Check service selector
kubectl get svc nginx-service -o jsonpath='{.spec.selector}'

# Get service cluster IP
kubectl get svc nginx-service -o jsonpath='{.spec.clusterIP}'

# Get NodePort
kubectl get svc nginx-service -o jsonpath='{.spec.ports[*].nodePort}'

# Delete service
kubectl delete svc nginx-service
```

## Service Types

```bash
# ClusterIP (default)
kubectl create service clusterip my-svc --tcp=80:8080

# NodePort
kubectl create service nodeport my-svc --tcp=80:8080 --node-port=30080

# LoadBalancer
kubectl create service loadbalancer my-svc --tcp=80:8080

# ExternalName
kubectl create service externalname my-svc --external-name=example.com
```

## Testing Services

```bash
# Port forward service
kubectl port-forward svc/nginx-service 8080:80

# Test from temporary pod
kubectl run test --image=busybox --rm -it --restart=Never -- wget -qO- nginx-service:80

# Test with curl
kubectl run curl --image=curlimages/curl --rm -it --restart=Never -- curl http://nginx-service

# DNS resolution test
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup nginx-service

# Test specific endpoint
kubectl run test --image=busybox --rm -it --restart=Never -- wget -qO- <pod-ip>:80
```

## Network Policies

```bash
# Get NetworkPolicies
kubectl get networkpolicies
kubectl get netpol  # Short form

# Describe NetworkPolicy
kubectl describe netpol allow-frontend

# Create basic deny-all policy
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF

# Allow from specific pods
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
EOF

# Delete NetworkPolicy
kubectl delete netpol deny-all
```

## Ingress

```bash
# Create Ingress (requires Ingress Controller)
kubectl create ingress my-ingress --rule="example.com/=service:80"

# Create with multiple paths
kubectl create ingress multi-path \
  --rule="example.com/app1=app1-svc:80" \
  --rule="example.com/app2=app2-svc:80"

# Generate YAML
kubectl create ingress my-ingress --rule="example.com/=service:80" --dry-run=client -o yaml > ingress.yaml

# Get Ingress
kubectl get ingress
kubectl get ing  # Short form

# Describe Ingress
kubectl describe ingress my-ingress

# Get Ingress address
kubectl get ingress my-ingress -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Edit Ingress
kubectl edit ingress my-ingress

# Delete Ingress
kubectl delete ingress my-ingress
```

## DNS and Service Discovery

```bash
# DNS format: <service-name>.<namespace>.svc.cluster.local

# Test service DNS (short name, same namespace)
kubectl exec <pod> -- nslookup service-name

# Test service DNS (FQDN)
kubectl exec <pod> -- nslookup service-name.default.svc.cluster.local

# Test cross-namespace DNS
kubectl exec <pod> -- nslookup service-name.other-namespace.svc.cluster.local

# Query SRV records (service discovery)
kubectl exec <pod> -- nslookup -type=srv _http._tcp.service-name.default.svc.cluster.local

# Test DNS resolution tools
kubectl run dnsutils --image=tutum/dnsutils --rm -it --restart=Never -- bash
# Then inside: nslookup, dig, host commands

# Check DNS config in pod
kubectl exec <pod> -- cat /etc/resolv.conf
```

## Endpoints

```bash
# Get endpoints for service
kubectl get endpoints service-name
kubectl get ep service-name  # Short form

# Describe endpoints
kubectl describe ep service-name

# Check if endpoints match pods
kubectl get pods -l app=nginx -o wide
kubectl get ep nginx-service

# Manually create endpoint (for external service)
kubectl apply -f - <<EOF
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service
subsets:
- addresses:
  - ip: 10.1.2.3
  ports:
  - port: 80
EOF
```

## Service Debugging

```bash
# Service has no endpoints
kubectl describe svc <service-name>
kubectl get ep <service-name>
kubectl get pods -l <selector-from-service> --show-labels

# Can't access service
kubectl run test --image=busybox --rm -it -- wget -qO- <service-name>:<port>
kubectl exec <pod> -- curl -v <service-name>:<port>

# Check service selector matches pod labels
kubectl get svc <service> -o jsonpath='{.spec.selector}'
kubectl get pods --show-labels

# Check service ports
kubectl get svc <service> -o jsonpath='{.spec.ports}'

# Check if pods are ready
kubectl get pods -l app=nginx -o jsonpath='{.items[*].status.conditions[?(@.type=="Ready")].status}'

# Test individual pod
kubectl get ep <service> -o jsonpath='{.subsets[*].addresses[*].ip}'
kubectl run test --image=busybox --rm -it -- wget -qO- <pod-ip>:<port>
```

## Network Policy Testing

```bash
# Test connectivity before policy
kubectl run test --image=busybox --rm -it -- wget -qO- backend-service:80

# Apply NetworkPolicy
kubectl apply -f network-policy.yaml

# Test connectivity after policy (should fail/succeed based on policy)
kubectl run test --image=busybox --rm -it -l app=frontend -- wget -qO- backend-service:80

# Check NetworkPolicy is applied
kubectl get netpol
kubectl describe netpol <policy-name>

# Test from pod with correct label
kubectl run allowed --image=busybox --rm -it -l app=allowed -- wget -qO- backend:80

# Test from pod without label (should fail)
kubectl run denied --image=busybox --rm -it -- wget -qO- backend:80
```

## Service Session Affinity

```bash
# Create service with session affinity
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: sticky-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600
EOF

# Test session affinity
for i in {1..10}; do kubectl run test-$i --image=curlimages/curl --rm -it --restart=Never -- curl sticky-service; done
```

## Port Forward for Testing

```bash
# Forward service port
kubectl port-forward svc/nginx-service 8080:80

# Test in another terminal
curl localhost:8080

# Forward to pod directly
kubectl port-forward pod/nginx-xxx 8080:80

# Forward multiple ports
kubectl port-forward svc/multi-port-svc 8080:80 9090:9090
```

## Ingress Testing

```bash
# Get Ingress address
kubectl get ingress

# Add to /etc/hosts (for local testing)
echo "<ingress-ip> example.com" | sudo tee -a /etc/hosts

# Test Ingress
curl http://example.com/app1
curl http://example.com/app2

# Test with host header
curl -H "Host: example.com" http://<ingress-ip>/

# Check Ingress backend
kubectl describe ingress my-ingress

# View Ingress controller logs
kubectl logs -n ingress-nginx <ingress-controller-pod>
```

## Multi-Port Services

```bash
# Create service with multiple ports
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: multi-port-svc
spec:
  selector:
    app: myapp
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: metrics
    port: 9090
    targetPort: 9090
EOF

# Test each port
kubectl run test --image=busybox --rm -it -- wget -qO- multi-port-svc:80
kubectl run test --image=busybox --rm -it -- wget -qO- multi-port-svc:9090
```

## Common Patterns

### Pattern 1: Create deployment and expose
```bash
kubectl create deployment nginx --image=nginx --replicas=3
kubectl expose deployment nginx --port=80 --target-port=80 --type=ClusterIP
kubectl get svc nginx
```

### Pattern 2: Test service connectivity
```bash
kubectl run test --image=busybox --rm -it --restart=Never -- sh
# Inside pod:
wget -qO- service-name:80
nslookup service-name
exit
```

### Pattern 3: Debug service with no endpoints
```bash
kubectl get svc <service> -o jsonpath='{.spec.selector}'  # Get selector
kubectl get pods -l <selector> --show-labels  # Check if pods match
kubectl get pods -l <selector> -o jsonpath='{.items[*].status.phase}'  # Check if running
kubectl get ep <service>  # Verify endpoints
```

### Pattern 4: Cross-namespace service access
```bash
# Service in namespace 'backend'
kubectl create service clusterip api --tcp=80:8080 -n backend

# Access from 'frontend' namespace
kubectl run test -n frontend --image=busybox --rm -it -- wget -qO- api.backend.svc.cluster.local:80
```

## Remember

- **Service selects pods by labels** - must match exactly
- **ClusterIP** - internal only
- **NodePort** - exposed on every node (30000-32767)
- **LoadBalancer** - external IP (cloud only)
- **Headless service** - clusterIP: None, returns pod IPs
- **Endpoints** - auto-created by service based on label selector
- **DNS format** - `service.namespace.svc.cluster.local`
- **Short DNS** - same namespace: just `service-name`
- **NetworkPolicy** - requires CNI plugin support
- **Default** - all traffic allowed (no NetworkPolicy)
- **Ingress** - requires Ingress Controller
- **Session affinity** - ClientIP or None
- **Service updates** - zero downtime for endpoint changes
