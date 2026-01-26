# Services & Networking - Practice Questions

## Question 1: Create Different Service Types

**Scenario:** Expose applications using different service types.

**Tasks:**
1. Create a deployment named `web-app` with 3 replicas
2. Expose it as ClusterIP service on port `80`
3. Create a NodePort service for external access
4. Create a headless service for direct pod access
5. Test connectivity to each service type
6. Document the differences

**Files:** `web-app-deployment.yaml`
**Solution:** See `solution-01.md`

---

## Question 2: Network Policies - Restrict Traffic

**Scenario:** Implement network policies to control traffic between pods.

**Tasks:**
1. Create three deployments:
   - `frontend` pods
   - `backend` pods
   - `database` pods
2. Create NetworkPolicy to:
   - Allow frontend to backend
   - Allow backend to database
   - Deny frontend to database (direct access)
   - Deny all other ingress traffic
3. Test and verify the policies
4. Add egress rules for DNS and external access

**Files:** `network-policy-setup.yaml`
**Solution:** See `solution-02.md`

---

## Question 3: Ingress Resource

**Scenario:** Configure Ingress for HTTP routing to multiple services.

**Tasks:**
1. Create two deployments: `app1` and `app2`
2. Expose both as ClusterIP services
3. Create an Ingress resource with:
   - Path `/app1` routes to `app1-service`
   - Path `/app2` routes to `app2-service`
   - Default backend for other paths
4. Configure TLS termination (assume cert exists)
5. Test the routing rules

**Files:** `ingress-setup.yaml`
**Solution:** See `solution-03.md`

---

## Question 4: Service Discovery and DNS

**Scenario:** Understand how pods discover services using DNS.

**Tasks:**
1. Create a service named `backend-service`
2. Create a client pod to test DNS resolution
3. Resolve service using:
   - Short name: `backend-service`
   - FQDN: `backend-service.default.svc.cluster.local`
4. Test different namespace DNS resolution
5. Query SRV records for port discovery
6. Document DNS naming patterns

**Solution:** See `solution-04.md`

---

## Question 5: Network Policy - Namespace Isolation

**Scenario:** Isolate namespaces using network policies.

**Tasks:**
1. Create two namespaces: `production` and `development`
2. Deploy apps in both namespaces
3. Create NetworkPolicy to:
   - Deny all cross-namespace traffic
   - Allow specific production service to be accessed from development
   - Allow all intra-namespace communication
4. Test the isolation
5. Add exception for monitoring namespace

**Solution:** See `solution-05.md`

---

## Question 6: Services with Session Affinity

**Scenario:** Configure service to maintain session affinity.

**Tasks:**
1. Create a deployment with 3 replicas
2. Create a service with session affinity (ClientIP)
3. Test that requests from same client go to same pod
4. Configure session timeout
5. Compare with service without affinity

**Files:** `session-affinity.yaml`
**Solution:** See `solution-06.md`

---

## Question 7: Multi-Port Services

**Scenario:** Expose multiple ports from a single service.

**Tasks:**
1. Create a pod with multiple containers:
   - HTTP server on port 80
   - Metrics endpoint on port 9090
2. Create a service exposing both ports with names
3. Access both endpoints through the service
4. Configure different target ports for each

**Files:** `multi-port-service.yaml`
**Solution:** See `solution-07.md`

---

## Question 8: NetworkPolicy - Allow from Namespace

**Scenario:** Allow traffic from specific namespace only.

**Tasks:**
1. Create two namespaces: `frontend` and `backend`
2. Deploy app in `backend` namespace
3. Create NetworkPolicy allowing ingress only from `frontend` namespace
4. Use namespaceSelector in policy
5. Test from both namespaces
6. Verify policy enforcement

---

## Question 9: Headless Service for StatefulSet

**Scenario:** Create headless service for direct pod access.

**Tasks:**
1. Create headless service (clusterIP: None)
2. Create StatefulSet using this service
3. Each pod gets DNS: `pod-0.service-name.namespace.svc.cluster.local`
4. Test DNS resolution for each pod
5. Verify direct pod-to-pod communication

---

## Question 10: ExternalName Service

**Scenario:** Create service that points to external DNS name.

**Tasks:**
1. Create ExternalName service pointing to `example.com`
2. Pods can access via service name
3. DNS CNAME record is created
4. No proxying happens
5. Test resolution and access

---

## Question 11: Ingress with TLS Termination

**Scenario:** Configure HTTPS ingress with certificate.

**Tasks:**
1. Create TLS secret with cert and key
2. Create Ingress with tls section
3. Configure host: `myapp.example.com`
4. Test HTTPS access
5. Verify certificate is served

---

## Question 12: NetworkPolicy - Egress Rules

**Scenario:** Control outbound traffic from pods.

**Tasks:**
1. Create NetworkPolicy with egress rules
2. Allow DNS queries (port 53)
3. Allow HTTPS to external sites (port 443)
4. Deny all other egress traffic
5. Test pod can resolve DNS and access HTTPS
6. Verify other traffic is blocked

---

## Question 13: Service with Session Affinity Timeout

**Scenario:** Configure sticky sessions with custom timeout.

**Tasks:**
1. Create service with `sessionAffinity: ClientIP`
2. Set `sessionAffinityConfig.clientIP.timeoutSeconds: 3600`
3. Test that requests from same client go to same pod
4. Wait for timeout and verify new distribution
5. Understand use cases for session affinity

---

## Question 14: NodePort Service on Specific Port

**Scenario:** Expose service on specific node port.

**Tasks:**
1. Create NodePort service
2. Specify nodePort: 30080
3. Access service on any node's IP:30080
4. Understand NodePort range (30000-32767)
5. Test from external client

---

## Question 15: NetworkPolicy - Default Deny All

**Scenario:** Secure namespace with default deny policy.

**Tasks:**
1. Create NetworkPolicy that denies all ingress and egress
2. Apply to all pods in namespace (empty podSelector)
3. Verify pods can't communicate
4. Add specific allow policies incrementally
5. Test granular access control

---

## Question 16: Ingress Path Types

**Scenario:** Understand different path matching types.

**Tasks:**
1. Create Ingress with `pathType: Prefix` for `/api`
2. Create another with `pathType: Exact` for `/api/v1`
3. Create one with `pathType: ImplementationSpecific`
4. Test different URLs and observe routing
5. Understand matching behavior

---

## Question 17: Service Without Selectors (Manual Endpoints)

**Scenario:** Create service pointing to external IP.

**Tasks:**
1. Create service without pod selector
2. Manually create Endpoints object
3. Add external IP addresses
4. Pods access external service via internal service name
5. Useful for external databases

---

## Question 18: NetworkPolicy with Port Ranges

**Scenario:** Allow traffic on port range.

**Tasks:**
1. Application uses ports 8000-8010
2. Create NetworkPolicy allowing this port range
3. Use endPort field (Kubernetes 1.22+)
4. Test access to ports in range
5. Verify ports outside range are blocked

---

## Question 19: Ingress with Multiple Hosts

**Scenario:** Route different domains to different services.

**Tasks:**
1. Create Ingress with rules for multiple hosts:
   - `app1.example.com` → app1-service
   - `app2.example.com` → app2-service
2. Configure default backend for unknown hosts
3. Test routing with Host headers
4. Verify each host routes correctly

---

## Question 20: NetworkPolicy - Allow from Pod with Label

**Scenario:** Fine-grained access control using pod labels.

**Tasks:**
1. Backend pods have label `app=backend`
2. Only frontend pods with `role=api-gateway` can access backend
3. Create NetworkPolicy with podSelector for both
4. Test from pods with and without correct labels
5. Verify label-based access control

---

## Question 21: Service Discovery via Environment Variables

**Scenario:** Use environment variables for service discovery.

**Tasks:**
1. Create service before pods
2. New pods get env vars: `SERVICE_NAME_SERVICE_HOST` and `SERVICE_NAME_SERVICE_PORT`
3. List env vars in pod
4. Compare with DNS-based discovery
5. Understand both methods

---

## Question 22: LoadBalancer Service with Source IP Preservation

**Scenario:** Preserve client source IP through LoadBalancer.

**Tasks:**
1. Create LoadBalancer service
2. Set `externalTrafficPolicy: Local`
3. Pods see real client IP, not node IP
4. Understand trade-offs (traffic distribution)
5. Test with `externalTrafficPolicy: Cluster` comparison

---

## Question 23: Ingress Annotations for Custom Behavior

**Scenario:** Configure ingress controller behavior via annotations.

**Tasks:**
1. Add `nginx.ingress.kubernetes.io/rewrite-target: /`
2. Add `nginx.ingress.kubernetes.io/ssl-redirect: "true"`
3. Add `nginx.ingress.kubernetes.io/rate-limit: "100"`
4. Test custom behaviors
5. Understand controller-specific annotations

---

## Question 24: NetworkPolicy - Combined Ingress and Egress

**Scenario:** Restrict both inbound and outbound traffic.

**Tasks:**
1. Create policy with both policyTypes
2. Allow ingress from frontend
3. Allow egress to database
4. Deny all other traffic
5. Test comprehensive security policy

---

## Question 25: Debugging Service Connection Issues

**Scenario:** Service exists but pods can't connect.

**Tasks:**
1. Check if service endpoints are populated
2. Verify pod labels match service selector
3. Test DNS resolution
4. Test direct pod IP access
5. Check NetworkPolicies
6. Use `kubectl port-forward` to isolate issue

---

## Services & Networking Concepts

### Service Types

| Type | Description | Use Case |
|------|-------------|----------|
| **ClusterIP** | Internal cluster IP (default) | Internal service communication |
| **NodePort** | Exposes on each Node's IP | External access without LoadBalancer |
| **LoadBalancer** | External load balancer (cloud) | Production external access |
| **ExternalName** | DNS CNAME record | Alias to external service |
| **Headless** | No cluster IP (clusterIP: None) | StatefulSet, direct pod access |

### Service Spec

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - name: http
    protocol: TCP
    port: 80          # Service port
    targetPort: 8080  # Container port
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600
```

### Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      role: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: database
    ports:
    - protocol: TCP
      port: 5432
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
```

### DNS Service Discovery

| DNS Record | Format | Example |
|------------|--------|---------|
| Service | `<service>.<namespace>.svc.cluster.local` | `backend.default.svc.cluster.local` |
| Pod | `<pod-ip>.<namespace>.pod.cluster.local` | `10-244-1-5.default.pod.cluster.local` |
| Headless Service Pod | `<pod-name>.<service>.<namespace>.svc.cluster.local` | `web-0.nginx.default.svc.cluster.local` |

---

## Tips for Services & Networking

- ClusterIP is default service type
- NodePort range: 30000-32767 (configurable)
- LoadBalancer creates NodePort and ClusterIP automatically
- Headless services (clusterIP: None) return pod IPs directly
- Network policies are additive (whitelist model)
- Default: All traffic allowed if no policies exist
- NetworkPolicy requires CNI plugin support (Calico, Cilium, etc.)
- Ingress requires an Ingress Controller (nginx, traefik, etc.)
- Service selects pods by labels
- Endpoints are automatically created/updated by service
- Use `kubectl get endpoints` to verify service pod mapping
- Session affinity: ClientIP or None
- DNS names are automatically created for services

---

## Quick Command Reference

```bash
# Services
kubectl expose deployment <name> --port=80 --target-port=8080
kubectl expose deployment <name> --type=NodePort --port=80
kubectl expose deployment <name> --type=LoadBalancer --port=80
kubectl create service clusterip <name> --tcp=80:8080
kubectl get svc
kubectl describe svc <name>
kubectl get endpoints <service-name>

# Port forwarding (for testing)
kubectl port-forward svc/<service-name> 8080:80
kubectl port-forward pod/<pod-name> 8080:80

# Network Policies
kubectl get networkpolicies
kubectl describe networkpolicy <name>

# Ingress
kubectl get ingress
kubectl describe ingress <name>

# DNS Testing
kubectl run test-pod --image=busybox:1.36 -it --rm -- nslookup <service-name>
kubectl run test-pod --image=busybox:1.36 -it --rm -- wget -qO- <service-name>

# Connectivity Testing
kubectl exec -it <pod> -- curl http://<service-name>
kubectl exec -it <pod> -- nc -zv <service-name> <port>

# Debug Service
kubectl run tmp --image=nicolaka/netshoot -it --rm -- /bin/bash
# Inside container:
# curl http://service-name
# nslookup service-name
# dig service-name
```

---

## Common Service Patterns

### ClusterIP Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
```

### Headless Service (StatefulSet)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  clusterIP: None  # Headless
  selector:
    app: web
  ports:
  - port: 80
```

### Service with Multiple Ports
```yaml
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
```

### NetworkPolicy - Deny All Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

### NetworkPolicy - Allow from Namespace
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-namespace
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend-namespace
```

---

## References

- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Service Discovery](https://kubernetes.io/docs/concepts/services-networking/service/#discovering-services)
- [Connecting Applications with Services](https://kubernetes.io/docs/tutorials/services/connect-applications-service/)
