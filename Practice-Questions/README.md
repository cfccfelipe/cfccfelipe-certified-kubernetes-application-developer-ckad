# CKAD Practice Questions - Complete Collection

## 📊 Question Count by Domain (Prioritized by Importance)

| Priority | Domain                    | Questions | Exam Weight | Folder                   |
| -------- | ------------------------- | --------- | ----------- | ------------------------ |
| **1**    | **Core Concepts**         | 15        | 20%         | 01-Core-Concepts/        |
| **2**    | **Pod Design**            | 22        | 20%         | 02-Pod-Design/           |
| **3**    | **Configuration**         | 20        | 18%         | 03-Configuration/        |
| **4**    | **Observability**         | 22        | 18%         | 04-Observability/        |
| **5**    | **Services & Networking** | 25        | 13%         | 05-Services-Networking/  |
| **6**    | **Security**              | 25        | N/A\*       | 06-Security/             |
| **7**    | **Multi-Container Pods**  | 20        | 10%         | 07-Multi-Container-Pods/ |
| **8**    | **State Persistence**     | 25        | 8%          | 08-State-Persistence/    |
|          | **TOTAL**                 | **174**   | **100%**    |                          |

\*Security topics are distributed across other CKAD exam domains but deserve focused practice.

---

## 🎯 How to Use These Questions

### Daily Practice Routine (30-60 minutes)

1. **Pick a domain** (rotate through all 8, prioritizing 1-5)
2. **Set up your environment** (kubectl, cluster access)
3. **Have cheatsheet handy** (EXAM-DAY-CHEATSHEET.md)
4. **Read 2-3 questions** from README.md
5. **Apply YAML files** if provided
6. **Read domain COMMANDS.md** first
7. **Set timer** for each question
8. **Use kubectl explain** when stuck
9. **Try to solve** without looking at solutions
10.  **Stop and Time yourself** (Goal 8-10 minutes per question)
11. **Check solutions** after attempting
12. **Mark difficult questions** for later review
13. **Clean up** resources before next question
14. After review the daily question, conclude if the domain is hard for you.

---

## 📁 Folder Structure

Each domain folder contains:

```
XX-Domain-Name/
├── README.md          # All questions for this domain
├── COMMANDS.md        # Domain-specific kubectl commands
├── *.yaml            # Example manifests and broken configs
└── solution-*.md     # Step-by-step solutions
```

---

## 🏆 Practice Goals

- [ ] Can solve 80% of the questions in less than 10 minutes?
- [ ] Can complete 15 questions in 2 hours?
- [ ] Can setup aliases and env variables?
- [ ] Know kubectl commands from memory?
- [ ] Can read/write YAML without docs?
- [ ] Completed all 174 questions at least once?
- [ ] Score 70%+ on Killer.sh simulator

---

## 🎯 Question Difficulty Levels

### Easy (5-7 minutes)

- Basic pod creation
- Simple ConfigMap/Secret
- Service exposure
- Label operations
- View logs

### Medium (8-10 minutes)

- Multi-container patterns
- Deployment updates
- NetworkPolicy creation
- PV/PVC setup
- Probe configuration

### Hard (10-12 minutes)

- Complex NetworkPolicies
- StatefulSet with storage
- Advanced troubleshooting
- Canary deployments
- Volume snapshots

---

## 📚 Domain Overview (Prioritized by Importance)

### 01-Core Concepts (15 Questions) - **PRIORITY 1**

**Focus:** Pods, namespaces, labels, resources
**Why First:** Foundation for everything in Kubernetes
**Key Skills:**

- Create/manage pods
- Use labels and selectors
- Set resource requests/limits
- Debug pending/failing pods
- Node selectors and tolerations

### 02-Pod Design (22 Questions) - **PRIORITY 2**

**Focus:** Deployments, Jobs, labels, rollouts
**Why Second:** Most common workload management, 20% of exam
**Key Skills:**

- Manage Deployments
- Perform rolling updates/rollbacks
- Create Jobs and CronJobs
- Use labels effectively
- Implement deployment strategies

### 03-Configuration (20 Questions) - **PRIORITY 3**

**Focus:** ConfigMaps, Secrets, SecurityContext, Quotas
**Why Third:** Used daily in production, 18% of exam
**Key Skills:**

- Create ConfigMaps from files/literals
- Manage Secrets securely
- Set SecurityContext for pods/containers
- Apply ResourceQuotas
- Use ServiceAccounts

### 04-Observability (22 Questions) - **PRIORITY 4**

**Focus:** Probes, logging, debugging, monitoring
**Why Fourth:** Critical for troubleshooting, 18% of exam
**Key Skills:**

- Configure all probe types
- View and filter logs
- Debug failing pods
- Use ephemeral containers
- Monitor resource usage

### 05-Services & Networking (25 Questions) - **PRIORITY 5**

**Focus:** Services, Ingress, NetworkPolicies, DNS
**Why Fifth:** Essential for app communication, 13% of exam
**Key Skills:**

- Create different service types
- Configure Ingress
- Apply NetworkPolicies
- Understand DNS resolution
- Debug connectivity issues

### 06-Security (25 Questions)

**Focus:** RBAC, SecurityContext, Pod Security, NetworkPolicies
**Why Sixth:** Critical for production, distributed across exam
**Key Skills:**

- Configure RBAC (Roles, RoleBindings, ServiceAccounts)
- Apply SecurityContext (runAsUser, capabilities, etc.)
- Use Pod Security Standards
- Secure secrets and network traffic
- Implement least privilege principle

### 07-Multi-Container Pods (20 Questions) - **PRIORITY 7**

**Focus:** Sidecar, Ambassador, Adapter, Init containers
**Why Seventh:** Specific patterns, 10% of exam
**Key Skills:**

- Implement design patterns
- Share volumes between containers
- Use init containers
- Debug multi-container issues
- Understand container lifecycle

### 08-State Persistence (25 Questions) - **PRIORITY 8**

**Focus:** PV, PVC, StatefulSets, volumes
**Why Eighth:** Important but less frequent, 8% of exam
**Key Skills:**

- Create PV and PVC
- Use StorageClasses
- Manage StatefulSets
- Mount volumes
- Handle volume operations

**Good luck! 🚀**
