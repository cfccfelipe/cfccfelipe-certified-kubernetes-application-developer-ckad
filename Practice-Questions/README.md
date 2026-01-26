# CKAD Practice Questions - Complete Collection

## 📊 Question Count by Domain (Prioritized by Importance)

| Priority | Domain | Questions | Exam Weight | Folder |
|----------|--------|-----------|-------------|--------|
| **1** | **Core Concepts** | 15 | 20% | 01-Core-Concepts/ |
| **2** | **Pod Design** | 22 | 20% | 02-Pod-Design/ |
| **3** | **Configuration** | 20 | 18% | 03-Configuration/ |
| **4** | **Observability** | 22 | 18% | 04-Observability/ |
| **5** | **Services & Networking** | 25 | 13% | 05-Services-Networking/ |
| **6** | **Security** | 25 | N/A* | 06-Security/ |
| **7** | **Multi-Container Pods** | 20 | 10% | 07-Multi-Container-Pods/ |
| **8** | **State Persistence** | 25 | 8% | 08-State-Persistence/ |
| | **TOTAL** | **174** | **100%** | |

*Security topics are distributed across other CKAD exam domains but deserve focused practice.

---

## 🎯 How to Use These Questions

### Daily Practice Routine (30-60 minutes)

1. **Pick a domain** (rotate through all 8, prioritizing 1-5)
2. **Read 2-3 questions** from README.md
3. **Try to solve** without looking at solutions
4. **Time yourself** (8-10 minutes per question)
5. **Apply YAML files** if provided
6. **Check solutions** after attempting
7. **Review COMMANDS.md** for that domain
8. **Clean up** resources before next question

### Weekly Schedule Example (Prioritized Order)

**Week 1:**
- Monday: Core Concepts (Q1-3)
- Tuesday: Pod Design (Q1-3)
- Wednesday: Configuration (Q1-3)
- Thursday: Observability (Q1-3)
- Friday: Services/Networking (Q1-3)
- Saturday: Security (Q1-3)
- Sunday: Multi-Container Pods (Q1-3)

**Week 2:**
- Repeat with Q4-6 from each domain

**Weeks 3-8:**
- Continue through all questions
- Redo difficult questions
- Focus on weak areas

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

## 🎓 Study Approach by Experience Level

### Beginner (New to Kubernetes)
1. Start with **Core Concepts** (all 15 questions)
2. Then **Configuration** (first 10 questions)
3. Then **Observability** (first 10 questions)
4. Build foundation before advanced topics

### Intermediate (Know Kubernetes Basics)
1. **Week 1-2:** Core Concepts + Configuration
2. **Week 3-4:** Multi-Container + Observability
3. **Week 5-6:** Pod Design + Services/Networking
4. **Week 7-8:** State Persistence + Review

### Advanced (Preparing for Exam)
1. **Week 1:** Do ALL questions once (timed)
2. **Week 2-3:** Redo questions you got wrong
3. **Week 4:** Speed practice (aim for 7 min per question)
4. **Week 5-6:** Killer.sh simulator
5. **Week 7-8:** Final review + exam

---

## 🏆 Practice Goals

### By Question Count
- [ ] Completed 20 questions (Basics covered)
- [ ] Completed 50 questions (Intermediate level)
- [ ] Completed 100 questions (Advanced level)
- [ ] Completed ALL 174 questions (Exam ready!)

### By Domain Mastery (Prioritized Order)
- [ ] Core Concepts (15/15)
- [ ] Pod Design (22/22)
- [ ] Configuration (20/20)
- [ ] Observability (22/22)
- [ ] Services & Networking (25/25)
- [ ] Security (25/25)
- [ ] Multi-Container Pods (20/20)
- [ ] State Persistence (25/25)

### By Speed
- [ ] Can solve easy questions in 5-7 minutes
- [ ] Can solve medium questions in 8-10 minutes
- [ ] Can solve hard questions in 10-12 minutes
- [ ] Can complete 15 questions in 2 hours (exam pace)

---

## 💡 Tips for Maximum Learning

### Before Starting
1. **Set up your environment** (kubectl, cluster access)
2. **Read domain COMMANDS.md** first
3. **Have cheatsheet handy** (Reference-Guides/EXAM-DAY-CHEATSHEET.md)
4. **Set timer** for each question

### During Practice
1. **Don't look at solutions immediately**
2. **Use kubectl explain** when stuck
3. **Verify with kubectl get/describe**
4. **Read error messages carefully**
5. **Try multiple approaches**

### After Each Question
1. **Check solution even if you got it right**
2. **Note different approaches**
3. **Add to your personal notes**
4. **Practice the commands again**
5. **Clean up all resources**

### Review Strategy
1. **Mark difficult questions** for later review
2. **Create personal cheat sheet** of patterns
3. **Note common mistakes** you make
4. **Track time per question type**
5. **Identify weak domains**

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

### 06-Security (25 Questions) - **PRIORITY 6 ⭐ NEW**
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

---

## 🚀 Quick Start

### First Time Here?
```bash
cd 01-Core-Concepts/
cat README.md           # Read questions
cat COMMANDS.md         # Review commands
# Try question 1
cat solution-01.md      # Check solution
```

### Daily Practice
```bash
# Pick a domain you haven't done today
cd 03-Multi-Container-Pods/
cat README.md           # Read questions 1-3
# Attempt them
cat COMMANDS.md         # Review commands
```

### Speed Practice
```bash
# Set timer: 10 minutes
# Pick random question from any domain
# Attempt to solve
# Check solution
# Repeat 6 times (1 hour practice)
```

---

## 📈 Progress Tracking

### Week 1 Checklist (Prioritized Order)
- [ ] Core Concepts: Q1-5
- [ ] Pod Design: Q1-5
- [ ] Configuration: Q1-5
- [ ] Observability: Q1-5
- [ ] Services/Network: Q1-5
- [ ] Security: Q1-5
- [ ] Multi-Container: Q1-5

### Week 2-4 Checklist
- [ ] Complete all remaining questions
- [ ] Redo difficult questions
- [ ] Review COMMANDS.md for each domain
- [ ] Focus extra time on Security domain

### Week 5-6 Checklist
- [ ] Speed practice all 174 questions
- [ ] Target: < 22 hours total
- [ ] Focus on weak areas
- [ ] Master Security questions

### Week 7-8 Checklist
- [ ] Killer.sh simulator
- [ ] Final review
- [ ] Schedule CKAD exam!

---

## 🎯 Exam Readiness Indicators

You're ready for CKAD when:
- ✅ Completed all 174 questions at least once
- ✅ Can solve 80% of questions in time limit
- ✅ Know kubectl commands from memory
- ✅ Can read/write YAML without docs
- ✅ Can debug pods quickly (< 2 minutes)
- ✅ Score 75%+ on Killer.sh simulator
- ✅ Feel confident with all 8 domains
- ✅ Understand security best practices (RBAC, SecurityContext)

---

## 📞 Need Help?

- **Stuck on a question?** Check COMMANDS.md for that domain
- **Want detailed commands?** See Reference-Guides/COMPREHENSIVE-COMMAND-REFERENCE.md
- **Need exam tips?** See Reference-Guides/CKAD Exam Tips and Shortcuts.md
- **Troubleshooting?** See Reference-Guides/Kubernetes Troubleshooting Scenarios.md

---

## 🎉 You Have 174 Questions to Master Kubernetes!

### Why This New Order?

**Prioritized by:**
1. **Foundation first** - Core Concepts you need before anything else
2. **Most used features** - Pod Design, Configuration, Observability (daily tools)
3. **Essential communication** - Services & Networking
4. **Security** - Critical for production ⭐ NEW DOMAIN
5. **Specialized patterns** - Multi-Container, State Persistence

Start here:
```bash
cd 01-Core-Concepts/
cat README.md
```

**Good luck! 🚀**
