# Kubernetes — Interview Module

## Enhancement Instructions

- Expect and reward **systematic troubleshooting**: context → triage → deep dive → root cause → remediation → verification. Penalize ad-hoc debugging without methodology.
- Use the **14 common issue patterns** below as the evaluation baseline. Score higher when the candidate maps symptoms to the correct pattern and names the right diagnostic commands in the right order.
- Evaluate **incident response maturity**: severity classification, containment strategy, post-incident review process, and prevention measures.
- Probe **performance troubleshooting** depth: CPU throttling, memory profiling, network latency layers, storage I/O, and cluster-wide API server health.
- When Helm questions are asked, evaluate Helm troubleshooting within the Kubernetes operational context (release recovery, values layering, hook failures).

---

## Evaluation Checklist

### Systematic Troubleshooting Workflow (Mandatory)

Every Kubernetes troubleshooting answer MUST follow this structured approach:

1. [ ] **Gather Context** — Understand the symptom, timeline, recent changes, scope (single pod vs cluster-wide), severity
2. [ ] **Initial Triage** — Cluster-level health check: node health, system pods, pending pods, failed pods, crash loops
3. [ ] **Deep Dive** — Namespace-level or pod-level targeted diagnosis using describe, logs, events
4. [ ] **Root Cause Identification** — Correlate symptoms with known failure patterns
5. [ ] **Remediation** — Apply fix, test in non-prod first, document actions, maintain rollback plan
6. [ ] **Verify & Monitor** — Confirm resolution, monitor 15-30 minutes for recurrence, check related systems

### Common Issue Diagnosis (14 Core Patterns)

#### Pod Issues
- [ ] **ImagePullBackOff**: Verify image name/tag, check imagePullSecrets, verify registry access, check rate limiting
- [ ] **CrashLoopBackOff**: Check logs + previous logs, examine exit codes (137=OOM, 1=app error), verify env vars, check liveness probe timing, verify dependencies
- [ ] **Pending Pods**: Check node resources, node selectors/affinity, taints/tolerations, PVC availability, resource quotas
- [ ] **OOMKilled**: Confirm via pod status (exit code 137), check memory usage vs limits, profile for leaks, consider Guaranteed QoS

#### Node Issues
- [ ] **Node NotReady**: Check kubelet status, journalctl logs, disk/memory/PID pressure, container runtime, network to API server
- [ ] **Disk Pressure**: Check disk usage (`df -h`), clean Docker/containerd cache, rotate logs, expand disk, configure kubelet GC

#### Networking Issues
- [ ] **Pod-to-Pod Failure**: Check network policies, CNI plugin health, test connectivity (`ping`, `nslookup`), verify service endpoints, check Pod CIDR exhaustion
- [ ] **Service Not Accessible**: Verify endpoints exist, check pod label matching, verify readiness probes, check LoadBalancer controller, test port accessibility

#### Storage Issues
- [ ] **PVC Pending**: Check StorageClass exists, verify dynamic provisioner running, check cloud credentials, verify capacity
- [ ] **Volume Mount Failure**: Check RWO multi-attach conflicts, verify PVC bound, check node-to-storage connectivity, check CSI driver health

#### Resource Issues
- [ ] **Resource Quota Exceeded**: Check quota usage, identify resource consumers, right-size requests, consider namespace splitting
- [ ] **CPU Throttling**: Check usage vs limits, review throttling metrics (`/sys/fs/cgroup/cpu/cpu.cfs_throttled_time`), increase limits or remove for bursty workloads, use HPA. **Prevention: set requests based on average, limits with 50-100% headroom.**

#### Security Issues
- [ ] **Image Vulnerability**: List running images, scan with trivy, prioritize by severity and exposure. Prevention: scan in CI, minimal base images, admission controllers
- [ ] **RBAC Permission Denied**: Check with `kubectl auth can-i`, verify Role/ClusterRole, check RoleBinding, verify service account. Prevention: least privilege, namespace-scoped roles

### StatefulSet Operations (NEW)
- [ ] Understands ordered deployment and scaling (ordinal index guarantees)
- [ ] Knows stable network identity and headless service requirement
- [ ] Can troubleshoot stuck StatefulSet rollouts (PVC binding, ordinal dependencies)
- [ ] Understands `volumeClaimTemplates` immutability and migration strategies

### Pod Security Admission (NEW)
- [ ] Understands PSA modes: enforce, audit, warn
- [ ] Knows the three policy levels: Privileged, Baseline, Restricted
- [ ] Can explain namespace-level label configuration
- [ ] Understands migration path from PodSecurityPolicy to PSA

### HPA Issues (NEW)
- [ ] Can troubleshoot HPA not scaling (metrics-server, resource targets, min/max replicas)
- [ ] Understands custom metrics HPA (Prometheus adapter, KEDA)
- [ ] Knows cooldown/stabilization windows and their impact

### Init Container Failures (NEW)
- [ ] Can diagnose init container stuck in `Init:CrashLoopBackOff`
- [ ] Understands init container ordering guarantees
- [ ] Knows common patterns: wait-for-db, config-generation, permission-setup

### Incident Response Framework

#### Severity Classification
- [ ] **SEV-1**: Complete outage, data loss, security breach — immediate all-hands response
- [ ] **SEV-2**: Major degradation, significant user impact — response within 15 minutes
- [ ] **SEV-3**: Minor impairment, workaround available — response within 1 hour
- [ ] **SEV-4**: Cosmetic issues, minimal impact — response during business hours

#### Response Phases
- [ ] **Detection**: Alerts, user reports, monitoring dashboards
- [ ] **Triage**: Assess impact, scope, gather initial data
- [ ] **Investigation**: Collect comprehensive diagnostics, identify root cause
- [ ] **Resolution**: Apply remediation, verify fix, monitor
- [ ] **Post-Incident**: Document, blameless post-mortem, preventive measures

#### Common Incident Scenarios
- [ ] **Complete cluster outage**: Check API server pods, etcd health, control plane resources, certificate expiration, cloud provider status. **Recovery: restore etcd from backup or rebuild cluster.**
- [ ] **Service degradation**: Check endpoints, pod count and restarts, recent deployments, resource usage. **Quick fix: scale up replicas or rollback bad deployment.**
- [ ] **Node failure**: Check kubelet, node conditions, disk/memory/PID pressure, container runtime. **Recovery: fix issue or cordon → drain → replace node.**
- [ ] **Storage issues**: Check PVC/PV status, StorageClass, provisioner pods, volume attachments. **Recovery: fix provisioner, manually provision PV, or restore from backup.**
- [ ] **Security incident**: Isolate affected resources, preserve evidence, rotate credentials, restore from known-good state. Check for **privileged pods, RBAC changes, suspicious service accounts.**

#### Post-Incident Review
- [ ] Blameless post-mortem within 48 hours
- [ ] Timeline documentation: detection → response → resolution → key events
- [ ] Root cause analysis with contributing factors
- [ ] Action items with owners and deadlines
- [ ] Prevention measures implemented and verified

### Performance Troubleshooting

#### Latency Diagnosis
- [ ] Identify the layer with latency (ingress, service, pod, dependency)
- [ ] Check CPU throttling as common latency cause
- [ ] Profile application performance (metrics endpoint, pprof)
- [ ] Test dependency connectivity and latency

#### CPU Performance
- [ ] Monitor with `kubectl top nodes` and `kubectl top pods`
- [ ] Check throttling via cgroup metrics
- [ ] Understand the CPU limits debate (remove limits for bursty workloads)
- [ ] Use HPA for auto-scaling based on CPU utilization

#### Memory Performance
- [ ] Identify OOMKilled pods and their patterns
- [ ] Profile memory usage over time
- [ ] Understand QoS classes: BestEffort → Burstable → Guaranteed
- [ ] Set requests = limits for Guaranteed QoS on critical workloads

#### Network Performance
- [ ] DNS resolution latency (CoreDNS scaling, NodeLocal DNSCache)
- [ ] Service mesh overhead (if applicable)
- [ ] Network policy impact on performance
- [ ] Pod-to-pod latency measurement

#### Storage I/O Performance
- [ ] Check IOPS limits for EBS volume type
- [ ] Monitor I/O wait on nodes
- [ ] Choose appropriate StorageClass (gp3 vs io2 vs local NVMe)
- [ ] Benchmark with fio for baseline

#### Cluster-Wide Performance
- [ ] API server request latency
- [ ] etcd performance
- [ ] Scheduler latency and pending pod queue
- [ ] Controller manager workqueue depth

### Profiling and Metrics (Expert-Level)

- [ ] Application profiling: Go pprof (`kubectl port-forward` + `curl /debug/pprof/profile`), Java async-profiler/jmap heap dumps
- [ ] Latency breakdown: use curl timing (`time_namelookup`, `time_connect`, `time_starttransfer`, `time_total`) to isolate DNS vs network vs application layers
- [ ] Key Prometheus queries: `histogram_quantile(0.95, ...)` for P95 latency, `rate(http_requests_total{status=~"5.."}[5m])` for error rate
- [ ] Distributed tracing: OpenTelemetry/Jaeger for end-to-end request path analysis
- [ ] Storage benchmarking: use `fio` for IOPS/throughput baselines, compare against EBS volume type limits

### Helm Troubleshooting

#### Release Management
- [ ] Diagnose stuck releases (pending-install, pending-upgrade): check `helm list --all`, look for stuck hooks, clean up with `helm uninstall --no-hooks`
- [ ] Handle failed upgrades: `helm rollback <release> 0`, or force: `helm upgrade --force`
- [ ] Debug "deployed but resources missing": compare `helm get manifest` with live resources
- [ ] Understand release history: `helm history <release>`, revision management

#### Installation & Upgrade
- [ ] Debug template rendering errors (`helm template`, `helm lint`, `--debug --dry-run`)
- [ ] Handle immutable field errors: delete resource then upgrade, or use `--force`
- [ ] Manage values layering: later `-f` files override earlier, `--set` overrides all
- [ ] Use `--atomic` for safer upgrades (auto-rollback on failure)

#### Dependencies & Hooks
- [ ] Resolve dependency update failures: `helm dependency update`, clean `charts/` and `Chart.lock`
- [ ] Debug stuck or failed hooks: check `kubectl get jobs -l helm.sh/hook`, inspect logs, use `--no-hooks` to bypass
- [ ] Configure hook deletion policies (`hook-succeeded`, `hook-failed`, `before-hook-creation`)
- [ ] Manage subchart values correctly **(nested under subchart name in parent values)**

#### Key Error Messages (Recognition)
- [ ] `cannot re-use a name that is still in use` — release exists, check `helm list --all`, uninstall first
- [ ] `chart requires kubeVersion: >=X.Y` — version mismatch, use compatible chart version
- [ ] `rendered manifests contain errors` — dry-run debug, check CRDs installed
- [ ] `immutable field` — delete resource or use `--force`

### Kubernetes Primitives Knowledge

#### Workload Management
- [ ] Deployments: rolling updates, rollback, revision history
- [ ] StatefulSets: ordered deployment, stable network identity, persistent storage
- [ ] DaemonSets: per-node workloads, use cases (logging, monitoring, CNI)
- [ ] Jobs/CronJobs: batch processing, completion tracking, TTL cleanup

#### Scheduling
- [ ] Resource requests and limits (CPU, memory, ephemeral storage)
- [ ] QoS classes: Guaranteed, Burstable, BestEffort
- [ ] Node selectors, affinity/anti-affinity
- [ ] Taints and tolerations
- [ ] Topology spread constraints
- [ ] Priority classes and preemption

#### Networking
- [ ] Service types: ClusterIP, NodePort, LoadBalancer, ExternalName
- [ ] Ingress controllers and Ingress resources
- [ ] Network policies (ingress and egress rules)
- [ ] DNS resolution: CoreDNS, service discovery, headless services
- [ ] CNI plugins: VPC-CNI mechanics, ENI limits, prefix delegation

#### Storage
- [ ] PersistentVolumes, PersistentVolumeClaims, StorageClasses
- [ ] Access modes: RWO, ROX, RWX
- [ ] Dynamic provisioning vs static provisioning
- [ ] CSI drivers: EBS CSI, EFS CSI
- [ ] Volume snapshots and backup

#### Security
- [ ] RBAC: Roles, ClusterRoles, RoleBindings, ClusterRoleBindings
- [ ] Service Accounts and IRSA
- [ ] Pod Security Standards (Privileged, Baseline, Restricted)
- [ ] Security Contexts (runAsNonRoot, readOnlyRootFilesystem, capabilities)
- [ ] Network Policies for microsegmentation
- [ ] Secrets management (External Secrets Operator, Sealed Secrets)

#### Observability
- [ ] Liveness probes: restart unhealthy containers
- [ ] Readiness probes: remove from service endpoints
- [ ] Startup probes: protect slow-starting containers
- [ ] Pod Disruption Budgets: protect availability during voluntary disruptions
- [ ] Resource metrics: metrics-server, `kubectl top`

### Diagnostic Commands

```bash
# Cluster Health
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods --all-namespaces | grep -v Running
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | tail -20

# Pod Diagnostics
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl get pod <pod> -n <namespace> -o yaml
kubectl debug <pod> -it --image=busybox -n <namespace>

# Node Diagnostics
kubectl describe node <node>
kubectl top nodes
kubectl top pods --all-namespaces
kubectl get nodes -o json | jq '.items[].status.conditions'

# Network Diagnostics
kubectl get endpoints <service> -n <namespace>
kubectl get networkpolicies --all-namespaces
kubectl exec -it <pod> -- nslookup <service>.<namespace>.svc.cluster.local
kubectl exec -it <pod> -- nc -zv <target-ip> <port>

# Storage Diagnostics
kubectl get pvc,pv --all-namespaces
kubectl describe pvc <pvc> -n <namespace>
kubectl get storageclass
kubectl get volumeattachments

# RBAC Diagnostics
kubectl auth can-i <verb> <resource> --as=<user/sa>
kubectl auth can-i --list --as=system:serviceaccount:<ns>:<sa>
kubectl get rolebindings,clusterrolebindings --all-namespaces

# Helm Diagnostics
helm list -n <ns> --all                              # See all releases including failed
helm history <release> -n <ns>                       # Check revision history
helm rollback <release> <revision> -n <ns>           # Rollback to working revision
helm uninstall <release> -n <ns> --no-hooks          # Force delete stuck release
kubectl delete secret -l owner=helm,name=<release>   # Clean orphaned helm secrets
helm diff upgrade <release> <chart> -n <ns>          # Preview changes (helm-diff plugin)
```

### Scoring Application

| Coverage | Score |
|----------|-------|
| 0-20% of relevant items | 1–25 (Awareness) |
| 20-50% | 26–50 (Competent) |
| 50-80%, follows systematic workflow | 51–75 (Proficient) |
| 80%+ with proactive depth, demonstrates incident response maturity | 76–100 (Expert) |

---

## Question Templates

### What Breaks If...

1. "Your EKS node group in `{path}` is set to `{config_detail}` nodes. One node becomes NotReady. Walk me through the systematic diagnostic workflow: what do you check at the node level (kubelet, disk pressure, memory pressure, container runtime), what happens to Pods on that node, and how does Kubernetes respond?"
2. "The `vpc-cni` addon configured in `{path}` runs out of IP addresses in the subnet. How does this manifest — what symptoms do you see at the Pod level and the node level? What are your remediation options?"
3. (generic) "A Deployment has no resource requests or limits set. What are the consequences during a node resource crunch? How does the Kubernetes QoS class system determine which Pods get evicted first?"
4. "The `aws-ebs-csi-driver` addon in `{path}` loses its IRSA permissions. What happens to existing PVCs? What happens when a new Pod tries to claim a PVC? Walk me through the troubleshooting steps."
5. (generic) "A container is repeatedly OOMKilled (exit code 137). Walk me through the systematic diagnosis: how do you confirm OOMKill, profile memory usage, determine whether it's a leak or an undersized limit, and what remediation steps do you take?"
6. (generic) "A StatefulSet is stuck during a rolling update — Pod ordinal 1 won't become Ready. What happens to ordinal 2? How does this differ from Deployment update behavior?"

### How Would You Debug...

7. "A Pod is stuck in `CrashLoopBackOff`. Following the systematic troubleshooting workflow, walk me through your debugging steps: what kubectl commands do you run, in what order, and what are you looking for at each step?"
8. "Services in namespace A cannot reach services in namespace B. Using the networking issues diagnostic approach, what do you check first, second, third? How do you distinguish between a network policy issue, a CNI issue, and a service misconfiguration?"
9. "Your EBS-backed PVC is stuck in `Pending`. Given that you have the `aws-ebs-csi-driver` addon in `{path}`, walk me through the storage troubleshooting workflow: what diagnostic commands do you run, what are the likely causes, and how do you verify each?"
10. (generic) "You suspect CPU throttling is causing latency spikes in your application. How do you confirm throttling is happening, distinguish it from other latency sources, and what are your remediation options — including the controversial choice of removing CPU limits entirely?"
11. "A Pod shows `ImagePullBackOff`. Your images are stored in ECR configured at `{path}`. Walk me through the full diagnostic checklist: image name/tag verification, registry authentication, network issues, rate limiting."
12. (generic) "An HPA is configured but Pods are not scaling up despite high CPU usage. Walk me through the diagnostic steps: metrics-server health, HPA status, resource targets, cooldown windows."

### Tradeoff

13. "You use `t3a.medium` instances for your EKS nodes. What are the tradeoffs vs `t3.medium`, `m6i.large`, or Graviton-based instances? How do burst credits affect Kubernetes workloads specifically?"
14. "Public + private endpoint access is enabled for your EKS cluster in `{path}`. Why not private-only? What are the security implications of each mode?"
15. (generic) "When would you use a DaemonSet vs a Deployment with node affinity? Give concrete examples from a production Kubernetes environment."
16. (generic) "Resource requests vs limits: some teams set requests = limits (Guaranteed QoS), others set only requests with no limits (Burstable QoS). What are the tradeoffs and when would you use each strategy?"

### Technology Comparison

17. "[DEFINE] What is EKS? Why EKS instead of self-managed Kubernetes (kubeadm), ECS, or k3s? What does the managed control plane give you that self-managed doesn't? What operational burden does it remove?"
18. "[DEFINE] What is VPC-CNI? You use the AWS VPC-CNI plugin in `{path}`. Why VPC-CNI instead of Calico, Flannel, or Cilium? How does VPC-CNI assign Pod IP addresses differently, and what are the IP exhaustion implications?"
19. "[DEFINE] What is the EBS CSI Driver? Why use the CSI driver addon instead of the in-tree EBS provisioner? What is IRSA and why does the CSI driver need it?"
20. "[DEFINE] What is CoreDNS? How does DNS resolution work for `service.namespace.svc.cluster.local`? Why CoreDNS instead of kube-dns? What performance issues can arise with DNS in large clusters?"

### Best Practice Pillar

21. **(Scalability)** "Your EKS node group is fixed at 3 nodes in `{path}`. What is a Cluster Autoscaler vs Karpenter? Why don't you have auto-scaling enabled, and what are the risks? How would you diagnose a 'Pending Pods due to insufficient resources' scenario?"
22. **(Availability)** "You have 2 AZs with 3 nodes. What happens if one AZ goes down? How many nodes survive? Can your workloads still run? What Kubernetes primitives (PDBs, topology spread constraints, pod anti-affinity) would improve availability?"
23. **(Security)** "What are Kubernetes RBAC, Pod Security Standards, and Network Policies? Which of these are implemented in your cluster? What attack vectors remain unaddressed? How would you use `kubectl auth can-i` to audit permissions?"
24. **(Cost)** "Your 3x t3a.medium nodes cost ~$92/month. How do you determine if they're right-sized? What kubectl and metrics-server commands reveal actual utilization? What cost savings could Karpenter with spot instances achieve?"
25. **(Security)** "Your cluster has no Pod Security Admission configured. What risks does this introduce? How would you implement the Restricted profile incrementally without breaking existing workloads?"

### Helm Questions

26. "A Helm release upgrade fails mid-way. What state is the release in, and what are your recovery options? How does `--atomic` change the behavior?"
27. "Someone runs `helm upgrade` with `--set` flags that conflict with the values file used by ArgoCD. What happens on the next ArgoCD sync?"
28. "[DEFINE] What is Helm? Why use Helm for packaging Kubernetes applications instead of raw YAML manifests with `kubectl apply`? What problems does Helm solve that plain manifests don't?"
29. "[DEFINE] What is a Helm chart vs a Helm release vs a Helm repository? You use ArgoCD to manage Helm releases. Why not Helmfile or Flux's HelmRelease CRD?"
30. (generic) "Helm vs Kustomize — when would you pick one over the other?"
31. (generic) "A chart subchart dependency has a breaking change in a new version. How does this propagate?"
32. "A Helm chart renders differently in CI vs locally. How do you isolate the difference? Which commands do you use?"
33. "`helm template` produces valid YAML but `helm install` fails. What are the common causes?"
34. "[SKIP if no Helm charts] The Helm chart at `{path}` uses values layering. How do you verify which values actually get applied?"
35. "ArgoCD manages your Helm releases. Why use ArgoCD's Helm support instead of raw `helm install`/`helm upgrade` in CI?"
36. "When would you use a Helm library chart vs copy-pasting templates across charts?"
37. "[DEFINE] What are Helm hooks? How do they compare to ArgoCD sync hooks and resource hooks? Which takes precedence when ArgoCD deploys a Helm chart?"
38. **(Availability)** "A Helm release upgrade fails and leaves the release in a FAILED state. What is Helm's rollback mechanism? How many release versions does Helm keep by default, and how does this affect your ability to recover?"
39. **(Security)** "Helm values files can contain secrets. How should secrets be handled in Helm charts? How does your project avoid putting secrets directly in values files?"

### Bonus Pool

40. "Walk me through the incident response framework for a SEV-1 Kubernetes outage: detection, triage, investigation, resolution, and post-incident phases. What specific kubectl commands do you run at each phase?"
41. "Explain the difference between liveness, readiness, and startup probes. When would you use each? What happens when each type fails, and how does misconfiguring them lead to cascading failures?"
42. "What is a PodDisruptionBudget and when is it critical? How does it interact with node drains, cluster autoscaler, and rolling updates?"
43. "Describe the Kubernetes eviction hierarchy: what determines which Pods get evicted first during node pressure? How do QoS classes, priority classes, and resource requests interact?"
44. "Your cluster's control plane seems slow — API server requests are taking 5+ seconds. Walk me through the performance troubleshooting approach."
45. "What is a CRD (Custom Resource Definition)? When would you write an operator vs using Helm charts? Give an example of when the operator pattern is the right choice."
