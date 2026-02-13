# Example Interview Transcript (Abbreviated)

This shows the expected flow for a DevOps Engineer interview preparation session with Kubernetes depth. Actual questions will vary based on repo content and user answers.

---

## Phase 1: Repo Intake

```
## Repo Map

| Layer | Artifacts Found | Key Paths |
|-------|----------------|-----------|
| Terraform (bootstrap) | S3 backend, ECR, Route 53, 6 Secrets Manager secrets | bootstrap/main.tf, bootstrap/variables.tf |
| Terraform (root modules) | VPC, EKS, IAM, ArgoCD, CDN (5 modules) | main.tf, modules/vpc/, modules/eks/, modules/argocd/, modules/cdn/ |
| Kubernetes | EKS 1.31, 3x t3a.medium, 4 addons, 4 IRSA roles | modules/eks/main.tf |
| Helm | ArgoCD chart v7.8.0, local app-of-apps chart | modules/argocd/main.tf |
| ArgoCD | App-of-apps pattern, auto-sync + prune + self-heal, 5 sync waves | modules/argocd/main.tf |
| CI/CD | Jenkinsfile (7 stages: build, unit, integration, e2e, tag, publish, deploy) | ../cowsay3d/Jenkinsfile |
| Docker | Multi-stage backend, nginx, MongoDB, 2 compose files | ../cowsay3d/Dockerfile, ../cowsay3d/docker-compose.yaml |
| DNS / CDN | Route 53 zone, CloudFront (S3 OAC + custom origin), ExternalDNS, cert-manager | modules/cdn/main.tf, bootstrap/main.tf |
| Scripts | bootstrap.sh, apply.sh, destroy.sh, kubeconfig.sh | scripts/ |
| Docs | Planning docs, validation docs, architecture diagram | docs/ |

### Round Availability

- Terraform & IaC: **Available** (rich artifacts)
- Kubernetes: **Available** (EKS config, IRSA, addons) — enhanced with k8s-troubleshooter depth
- Helm: **Limited** (ArgoCD chart only, no standalone charts)
- ArgoCD / GitOps: **Available** (app-of-apps, sync policies, SSH keys)
- AWS Architecture: **Available** (VPC, IAM, CDN, DNS, ECR)
- Reliability / Observability: **Limited** (health/metrics endpoints, no Prometheus config)
- Security: **Available** (IRSA, secrets, non-root containers, ECR scanning)
```

What interview mode would you like?

> **User selects: Kubernetes-only**

---

## Phase 3: Interview (Excerpts)

### Kubernetes Question — Troubleshooting (CrashLoopBackOff)

**Interviewer:**
Your EKS cluster runs workloads across 3 nodes configured in `modules/eks/main.tf`. Let's start with a common production issue.

**A Pod is stuck in `CrashLoopBackOff`. Following the k8s-troubleshooter systematic workflow, walk me through your debugging steps: what kubectl commands do you run, in what order, and what are you looking for at each step?**

> **User:** First I'd check the pod status with `kubectl describe pod <name> -n <namespace>` to see the events and container states. I'd look at the exit code — 137 means OOMKilled, 1 means application error. Then I'd check current logs with `kubectl logs <pod>` and previous container logs with `kubectl logs <pod> --previous` since the container keeps restarting. I'm looking for startup errors, missing env vars, or connection failures to dependencies. If the logs don't show the issue, I'd check if the liveness probe is configured too aggressively — maybe the app needs more time to start. I'd also verify resource limits aren't too tight by checking the container's resource spec.

**Follow-up 1:** Good systematic approach. The logs show the application can't connect to MongoDB. The MongoDB service is in a different namespace. What specific diagnostics do you run to determine if this is a DNS issue, a network policy issue, or a MongoDB availability issue?

> **User:** I'd first check if DNS resolution works: `kubectl exec <pod> -- nslookup mongodb.database.svc.cluster.local`. If DNS resolves but connection fails, I'd check network policies: `kubectl get networkpolicies -n database` and `kubectl get networkpolicies -n <app-namespace>`. Then I'd verify the MongoDB service has endpoints: `kubectl get endpoints mongodb -n database`. If endpoints exist but connection still fails, I'd test direct connectivity: `kubectl exec <pod> -- nc -zv <mongodb-pod-ip> 27017`. This helps isolate whether it's DNS, network policy, service config, or MongoDB itself.

**Follow-up 2:** The MongoDB pod itself is also in CrashLoopBackOff. How does this change your diagnosis, and what does this tell you about the failure pattern?

> **User:** This is a cascading failure — the app is crashing because its dependency is down. I'd pivot to debugging the MongoDB pod first. Same workflow: describe, logs, previous logs. Common MongoDB startup failures include: volume mount issues if PVC isn't bound, insufficient memory causing OOMKill, or configuration errors. I'd also check if this started after a recent deployment by looking at `kubectl rollout history`. If it's an OOMKill, I need to increase memory limits. If it's a PVC issue, I'd check `kubectl get pvc -n database` and follow the storage troubleshooting path.

### Score: Kubernetes — CrashLoopBackOff Troubleshooting
```
Rating: 3/4 — Proficient
Tech Justification: 3/4 — Good use of kubectl commands in correct diagnostic order.
  Demonstrated the systematic workflow: describe → logs → previous logs → targeted diagnosis.
Strengths: Correct exit code interpretation (137 = OOM, 1 = app error). Good cross-namespace
  DNS debugging sequence. Correctly identified cascading failure pattern and pivoted to root
  cause (MongoDB). Knew to check rollout history for recent changes.
Gap: Did not mention checking events at the namespace level (`kubectl get events -n <ns>`)
  for broader context. Did not reference init containers for dependency checking as a
  prevention mechanism. An expert would also check if the restart policy and backoff
  interval are appropriate, and mention `kubectl debug` for ephemeral container debugging.
Key point for interview: "I follow a systematic troubleshooting workflow: describe for
  state and events, logs for application errors, then targeted network/storage/resource
  diagnostics depending on the symptoms. Always identify cascading failures early."
```

**Model answer:** CrashLoopBackOff means the container repeatedly starts and crashes, with Kubernetes applying an exponential backoff delay between restarts. The systematic diagnostic workflow starts with `kubectl describe pod` to examine exit codes, container states, and events. Exit code 137 indicates OOMKill (kernel killed the process), while non-zero codes indicate application errors. Next, `kubectl logs --previous` captures the last container's output before the crash. For dependency failures, DNS resolution (`nslookup`), network policy inspection, and endpoint verification isolate the layer. In this case, identifying the cascading failure (MongoDB down → app can't connect → app crashes) is critical — always debug the root cause service first, not the dependent.

---

### Kubernetes Question — Technology Comparison (VPC-CNI)

**Interviewer:**
Your EKS cluster uses the AWS VPC-CNI plugin, configured as an addon in `modules/eks/main.tf`.

**[DEFINE] What is VPC-CNI? Why VPC-CNI instead of Calico, Flannel, or Cilium? How does VPC-CNI assign Pod IP addresses differently, and what are the IP exhaustion implications for your /24 subnets?**

> **User:** VPC-CNI is AWS's Container Network Interface plugin for EKS. It assigns real VPC IP addresses to each Pod by attaching secondary IP addresses to the node's Elastic Network Interfaces. This means Pods are first-class VPC citizens — they can be addressed directly by security groups, VPC routing rules, and AWS load balancers without any overlay network.

> Compared to Calico or Flannel, which use overlay networks (VXLAN or IP-in-IP encapsulation), VPC-CNI has no overlay overhead — better network performance and simpler debugging since Pod IPs are routable within the VPC. Cilium uses eBPF and is more feature-rich for observability and network policy, but it's not the default for EKS and adds operational complexity.

> The tradeoff is IP consumption. Each t3a.medium node can have 3 ENIs with 6 IPs each, so roughly 17 Pod IPs per node (minus some for system use). With /24 subnets (256 IPs each), 3 nodes consume maybe 50-60 IPs. That's fine now, but if I scaled to 10+ nodes or ran many Pods per node, I could hit IP exhaustion. The mitigation is prefix delegation, which assigns /28 prefixes instead of individual IPs, dramatically increasing Pod density.

**Follow-up:** You mentioned prefix delegation. What is it exactly, and what would you need to change in your VPC-CNI configuration to enable it?

> **User:** Prefix delegation changes VPC-CNI from allocating individual secondary IPs to allocating /28 prefixes (16 IPs each) to ENIs. This means each ENI slot gives you 16 IPs instead of 1, massively increasing the number of Pods per node. For t3a.medium, you'd go from ~17 to potentially over 100 Pod IPs per node. To enable it, you'd set the `ENABLE_PREFIX_DELEGATION` environment variable to true in the VPC-CNI DaemonSet, and ensure your subnets have enough contiguous /28 blocks available.

### Score: Kubernetes — VPC-CNI Technology Comparison
```
Rating: 4/4 — Expert
Tech Justification: 4/4 — Textbook definition. Compared 3 alternatives (Calico, Flannel,
  Cilium) with specific technical tradeoffs. Tied choice to project constraints (EKS default,
  AWS integration, performance). Calculated IP consumption for the specific instance type.
Strengths: Precise definition of VPC-CNI's IP assignment mechanism (secondary IPs on ENIs).
  Correctly contrasted overlay vs native networking. Knew ENI/IP limits for t3a.medium.
  Unprompted mention of prefix delegation as the scaling solution. Specific about the
  /28 prefix and 16 IPs per slot.
Gap: None significant. Could have mentioned that Cilium can run on EKS as a replacement
  CNI and provides eBPF-based network policy enforcement without iptables, which is an
  emerging trend. Could also mention the WARM_IP_TARGET and MINIMUM_IP_TARGET env vars
  for tuning IP pre-allocation.
Key point for interview: "VPC-CNI assigns real VPC IPs to Pods, making them first-class
  network citizens with no overlay overhead. The tradeoff is IP consumption, which I'd
  address with prefix delegation for higher Pod density."
```

**Model answer:** VPC-CNI (Virtual Private Cloud Container Network Interface) is AWS's CNI plugin that assigns real VPC IP addresses to Kubernetes Pods by attaching secondary IPs to node ENIs, making Pods directly routable within the VPC without overlay networks. In this project, VPC-CNI is the natural choice for EKS because it provides native AWS integration: security groups can target Pod IPs, AWS load balancers route directly to Pods, and there's zero overlay encapsulation overhead. Calico and Flannel use overlay networks (VXLAN/IP-in-IP) adding latency and complexity; Cilium offers eBPF-based observability but isn't the EKS default and adds operational overhead. The key tradeoff is IP consumption: t3a.medium nodes support ~17 Pod IPs with standard allocation. With /24 subnets, this is sufficient for 3 nodes but won't scale — prefix delegation (/28 prefixes per ENI slot) increases capacity to 100+ Pods per node.

---

### Kubernetes Question — Best Practice Pillar (Availability)

**Interviewer:**
Your EKS cluster has 3 nodes spread across 2 AZs, configured in `modules/eks/main.tf`.

**You have 2 AZs with 3 nodes. What happens if one AZ goes down? How many nodes survive? Can your workloads still run? What Kubernetes primitives would improve availability?**

> **User:** With 3 nodes across 2 AZs, the distribution is likely 2 nodes in one AZ and 1 in the other. If the AZ with 2 nodes goes down, I'm left with 1 node. That single node has to handle all workloads, and if the pods were spread across the failed nodes, Kubernetes will try to reschedule them — but only if there's enough capacity on the surviving node.

> If the AZ with 1 node goes down, I keep 2 nodes, which is more manageable. Either way, it's risky. To improve this, I'd use Pod Disruption Budgets to ensure minimum replicas during disruptions. Topology spread constraints would ensure pods are distributed evenly across AZs. Pod anti-affinity rules would prevent multiple replicas from landing on the same node. And ideally, I'd add a third AZ for true multi-AZ resilience.

**Follow-up:** With only 1 surviving node and multiple workloads competing for resources, what determines which Pods get scheduled first and which get evicted?

> **User:** Kubernetes uses Priority Classes to determine scheduling order — higher priority pods get scheduled first. For eviction, it follows the QoS class hierarchy: BestEffort pods (no requests or limits) get evicted first, then Burstable (requests < limits), and Guaranteed pods (requests = limits) are evicted last. Within the same QoS class, pods using more resources relative to their requests get evicted first. System-critical pods in kube-system typically have high priority and Guaranteed QoS, so they survive longest.

### Score: Kubernetes — Multi-AZ Availability
```
Rating: 3/4 — Proficient
Tech Justification: 3/4 — Good definitions of PDBs, topology spread constraints,
  and QoS classes. Correctly described the eviction hierarchy.
Strengths: Correctly analyzed the uneven AZ distribution (2+1). Identified both failure
  scenarios and their different impacts. Named the right Kubernetes primitives for
  improvement (PDBs, topology spread, anti-affinity, third AZ). Accurate QoS eviction
  hierarchy explanation.
Gap: Did not mention that the node-group's AZ distribution depends on the subnet
  configuration in `modules/eks/main.tf` — the actual distribution might differ from
  the assumed 2+1. Did not discuss pod preemption (when a high-priority pod can evict
  a lower-priority one to get scheduled). An expert would calculate whether a single
  t3a.medium (2 vCPU, 4 GiB) has enough resources for all workloads.
Key point for interview: "With 3 nodes in 2 AZs, an AZ failure leaves at most 2 nodes.
  I'd use topology spread constraints and PDBs to ensure workload survivability, and
  add a third AZ for production-grade resilience."
```

**Model answer:** With 3 nodes across 2 AZs, the worst-case AZ failure leaves 1 surviving node. Whether workloads survive depends on: (1) available capacity on the remaining node(s) — a t3a.medium has 2 vCPU and 4 GiB RAM, (2) Pod resource requests — Kubernetes only schedules Pods if their requests fit, and (3) Priority Classes — higher-priority Pods preempt lower-priority ones. The eviction order follows QoS classes: BestEffort first, then Burstable, then Guaranteed. To improve availability: topology spread constraints distribute replicas across AZs, PodDisruptionBudgets prevent voluntary disruptions from reducing replicas below a threshold, and pod anti-affinity ensures replicas don't co-locate. For production, a third AZ and Karpenter for dynamic node provisioning would provide true multi-AZ resilience.

---

## Phase 4: Debrief (Excerpt)

```
## Interview Debrief

### Score Summary
| Round | Questions | Avg Score | Tech Justification Avg | Level |
|-------|-----------|-----------|----------------------|-------|
| Kubernetes | 8 | 3.3 | 3.4 | Proficient+ |

**Overall: 3.3 / 4.0 — Strong Proficient**

### Best Practice Pillar Coverage
| Pillar | Questions Asked | Avg Score | Strongest Area | Weakest Area |
|--------|----------------|-----------|----------------|--------------|
| Security | 2 | 3.0 | RBAC, Pod Security | Network policies |
| Scalability | 2 | 3.0 | Node scaling concepts | HPA, VPA |
| Availability | 2 | 3.0 | AZ failure analysis | PDB configuration |
| Cost Optimization | 2 | 2.5 | Instance type awareness | Spot pricing, right-sizing |

### Kubernetes Troubleshooting Readiness (from k8s-troubleshooter standards)
- Systematic troubleshooting workflow: **demonstrated** — consistently followed describe → logs → targeted diagnosis
- Incident response awareness: **adequate** — understands severity but didn't mention post-mortem process
- Common issue diagnosis: **9/12 common issues covered confidently** (missed: Disk Pressure, Resource Quota Exceeded, CPU Throttling)
- Performance troubleshooting depth: **adequate** — understands resource contention but lacks metrics-server profiling
- Helm troubleshooting: **needs work** — not tested in Kubernetes-only mode

### Technology Justification Readiness
- Well-justified: VPC-CNI vs alternatives, EKS managed control plane, IRSA vs node roles, QoS classes
- Struggled to compare: CoreDNS vs kube-dns (not discussed), CSI driver vs in-tree (vague)
- Missing formal definitions: PodDisruptionBudget (used correctly but not defined), topology spread constraints

### Strengths
- Strong systematic troubleshooting methodology — follows diagnostic workflow consistently
- Excellent VPC-CNI understanding including IP allocation mechanics and prefix delegation
- Good cascading failure identification and root cause pivoting
- Accurate QoS class and eviction hierarchy knowledge
- Security-conscious — mentions RBAC auditing and Pod Security Standards

### Gaps
1. Disk pressure and node-level troubleshooting (kubelet logs, container runtime diagnostics)
2. Performance profiling with metrics-server and `kubectl top`
3. CPU throttling diagnosis and the debate around removing CPU limits
4. Formal definitions missing for some Kubernetes primitives (PDB, topology spread)
5. Incident response post-mortem process not articulated
6. Cost estimation and right-sizing methodology

### Interview-Ready Talking Points
Top 5 things to confidently say in an actual interview:
1. "I follow a systematic Kubernetes troubleshooting workflow: describe for state and events, logs for application errors, then targeted diagnostics for network, storage, or resource issues — always identifying cascading failures early."
2. "VPC-CNI assigns real VPC IPs to Pods, making them first-class network citizens. The tradeoff is IP consumption, which I'd address with prefix delegation for higher Pod density in scaling scenarios."
3. "For multi-AZ resilience, I use topology spread constraints to distribute replicas across AZs and PodDisruptionBudgets to maintain minimum availability during voluntary disruptions."
4. "Kubernetes evicts Pods by QoS class: BestEffort first, then Burstable, then Guaranteed. I set resource requests equal to limits for critical workloads to ensure Guaranteed QoS."
5. "When diagnosing CrashLoopBackOff, I check exit codes first (137 = OOM, non-zero = app error), then previous container logs, then dependency connectivity — always solving the root cause service before dependent services."

### Targeted Drill Plan
| Priority | Gap | Pillar | Suggested Exercise | Resources |
|----------|-----|--------|-------------------|-----------|
| 1 | Node-level troubleshooting | Availability | SSH into an EKS node and practice: `journalctl -u kubelet`, `df -h`, `free -m`, `docker stats`. Simulate disk pressure by filling a volume. | k8s-troubleshooter common_issues.md: Node Issues |
| 2 | Performance profiling | Scalability | Deploy metrics-server, run `kubectl top nodes` and `kubectl top pods`. Create an HPA and load-test to observe scaling behavior. | k8s-troubleshooter performance_troubleshooting.md |
| 3 | Missing definitions | All | Write formal 1-sentence definitions for: PDB, topology spread constraint, CPU throttling, QoS class, prefix delegation, init container, ephemeral container. Practice saying them. | `checklists/technology-comparisons.md` |
| 4 | CPU throttling | Cost | Deploy a CPU-intensive workload with tight limits. Monitor throttling via cgroup metrics. Test the "remove CPU limits" approach and measure the impact. | k8s-troubleshooter performance_troubleshooting.md: CPU section |
| 5 | Incident response | Availability | Write a mock SEV-1 incident report for "all Pods evicted after node disk pressure." Include timeline, root cause, remediation, and prevention. | k8s-troubleshooter incident_response.md |
| 6 | Cost right-sizing | Cost | Run `kubectl top` for 1 week, export metrics, calculate actual CPU/memory utilization vs requests. Compare t3a.medium cost with t4g.medium and spot pricing. | AWS Pricing Calculator, `checklists/best-practices.md` |
```
