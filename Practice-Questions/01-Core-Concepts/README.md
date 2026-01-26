# Core Concepts - Practice Questions

## Question 1: Create and Manage Basic Pods

**Scenario:** You need to create a pod and perform basic operations on it.

**Tasks:**
1. Create a pod named `nginx-pod` in the `default` namespace using the `nginx:1.21` image
2. Expose port `80` in the pod
3. Add labels: `app=web`, `tier=frontend`
4. Set resource requests: `memory=128Mi`, `cpu=100m`
5. Verify the pod is running
6. Get the pod's IP address

**Initial Setup:**
```bash
# Verify namespace exists
kubectl get ns default
```

**Solution:** See `solution-01.md`

---

## Question 2: Fix Broken Pod from YAML

**Scenario:** A developer created a pod but it's not starting. Fix the YAML file.

**Tasks:**
1. Apply the pod from `broken-pod.yaml`
2. Identify why it's not starting
3. Fix the YAML file
4. Verify the pod is running

**Initial Setup:**
```bash
# Apply the broken pod
kubectl apply -f broken-pod.yaml
```

**Files:** `broken-pod.yaml`
**Solution:** See `solution-02.md`

---

## Question 3: Create Pod with Environment Variables

**Scenario:** Create a pod that uses environment variables.

**Tasks:**
1. Create a pod named `env-pod` using `busybox:1.36` image
2. Set command to: `sleep 3600`
3. Add environment variables:
   - `DB_HOST=mysql-service`
   - `DB_PORT=3306`
   - `APP_ENV=production`
4. Verify environment variables inside the pod

**Solution:** See `solution-03.md`

---

## Question 4: Create Multiple Pods with Different Restart Policies

**Scenario:** Create pods with different restart policies to understand their behavior.

**Tasks:**
1. Create pod `always-restart` with `restartPolicy: Always` that runs a failing command
2. Create pod `never-restart` with `restartPolicy: Never` that completes successfully
3. Create pod `onfailure-restart` with `restartPolicy: OnFailure` that fails
4. Observe and document the restart behavior of each pod

**Solution:** See `solution-04.md`

---

## Question 5: Create Pod from Existing Deployment

**Scenario:** A deployment exists but you need to create a standalone pod with the same configuration.

**Tasks:**
1. Check the existing deployment in `webapp-deployment.yaml`
2. Extract the pod template
3. Create a standalone pod named `webapp-standalone` with the same spec
4. Ensure it has labels: `app=webapp`, `version=v1`

**Files:** `webapp-deployment.yaml`
**Solution:** See `solution-05.md`

---

## Question 6: Pod with Resource Limits Causing OOMKilled

**Scenario:** A pod keeps getting OOMKilled. Fix the resource limits.

**Tasks:**
1. Create a pod named `memory-demo` with `nginx:1.21` image
2. Set memory request: `64Mi` and limit: `128Mi`
3. Pod will be killed if it exceeds memory
4. Check pod status and events
5. Increase memory limit to `256Mi` and recreate

---

## Question 7: Create Pod in Specific Namespace with Node Selector

**Scenario:** Create a pod that runs on nodes with specific labels.

**Tasks:**
1. List all nodes and their labels
2. Create namespace `production` if it doesn't exist
3. Create a pod named `frontend` in `production` namespace
4. Add nodeSelector to run on nodes with label `disktype=ssd`
5. Verify which node the pod is scheduled on

---

## Question 8: Pod with Custom Command and Arguments

**Scenario:** Create a pod that runs a custom command.

**Tasks:**
1. Create a pod named `custom-cmd` using `busybox:1.36`
2. Set command: `["/bin/sh"]`
3. Set args: `["-c", "echo Hello from custom pod && sleep 3600"]`
4. Verify the output in logs
5. Update the pod to print current date every 10 seconds

---

## Question 9: Debug Pending Pod

**Scenario:** A pod is stuck in Pending state. Debug and fix it.

**Tasks:**
1. Create a pod with resource requests that exceed cluster capacity
2. Check why it's pending using `kubectl describe`
3. Check events
4. Reduce resource requests to fix
5. Verify pod becomes Running

---

## Question 10: Create Static Pod

**Scenario:** Create a pod that runs directly on a node without API server.

**Tasks:**
1. Identify the static pod manifest directory
2. Create a static pod manifest for `nginx:1.21`
3. Verify the static pod is running
4. Try to delete it using `kubectl delete`
5. Observe the behavior

---

## Question 11: Pod with Multiple Containers Sharing Network

**Scenario:** Create a pod with two containers that communicate via localhost.

**Tasks:**
1. Create a pod named `network-test` with two containers:
   - Container 1: `nginx:1.21` on port 80
   - Container 2: `busybox:1.36` running `wget localhost -O-`
2. Verify container 2 can access container 1 via localhost
3. Check logs from both containers

---

## Question 12: Pod Tolerations and Taints

**Scenario:** Create a pod that can be scheduled on tainted nodes.

**Tasks:**
1. Check if any nodes have taints
2. Create a pod that tolerates the taint
3. Verify pod is scheduled on the tainted node
4. Remove toleration and observe behavior

---

## Question 13: Pod with Host Networking

**Scenario:** Create a pod that uses the host's network namespace.

**Tasks:**
1. Create a pod with `hostNetwork: true`
2. Verify the pod has the host's IP address
3. Compare with a normal pod's networking
4. Document the differences

---

## Question 14: Pod Priority and Preemption

**Scenario:** Create pods with different priorities.

**Tasks:**
1. Create PriorityClass `high-priority` with value 1000
2. Create PriorityClass `low-priority` with value 100
3. Create pods using different priority classes
4. Observe scheduling behavior

---

## Question 15: Pod with DNS Policy

**Scenario:** Configure custom DNS settings for a pod.

**Tasks:**
1. Create a pod with `dnsPolicy: Default`
2. Create another pod with `dnsPolicy: ClusterFirst`
3. Compare DNS resolution behavior
4. Add custom DNS servers using `dnsConfig`
5. Verify custom DNS settings

---

## Tips for Core Concepts

- Use `kubectl run` for quick pod creation
- Use `--dry-run=client -o yaml` to generate YAML templates
- Master these commands:
  - `kubectl get pods`
  - `kubectl describe pod <name>`
  - `kubectl logs <pod-name>`
  - `kubectl exec -it <pod-name> -- /bin/sh`
  - `kubectl delete pod <name>`
- Remember: Pods are ephemeral - they're not meant to be long-lived
- Always check `kubectl describe pod` for debugging
- Resource limits prevent pods from consuming too much
- Node selectors schedule pods on specific nodes
- Static pods are managed by kubelet, not API server

---

## References

- [Pod Overview](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [kubectl run](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run)
- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Assign Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
