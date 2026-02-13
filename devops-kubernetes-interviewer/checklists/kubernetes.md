# Kubernetes Checklist

> **Kubernetes standards derived from `k8s-troubleshooter@devops-skills`.**
>
> This checklist is the source of truth for evaluating Kubernetes answers during the interview.
> When scoring Kubernetes questions, compare the candidate's answer against these standards.

---

## Systematic Troubleshooting Workflow (Mandatory)

Every Kubernetes troubleshooting answer MUST follow this structured approach:

1. [ ] **Gather Context** — Understand the symptom, timeline, recent changes, scope (single pod vs cluster-wide), severity
2. [ ] **Initial Triage** — Cluster-level health check: node health, system pods, pending pods, failed pods, crash loops
3. [ ] **Deep Dive** — Namespace-level or pod-level targeted diagnosis using describe, logs, events
4. [ ] **Root Cause Identification** — Correlate symptoms with known failure patterns
5. [ ] **Remediation** — Apply fix, test in non-prod first, document actions, maintain rollback plan
6. [ ] **Verify & Monitor** — Confirm resolution, monitor 15-30 minutes for recurrence, check related systems

---

## Common Issue Diagnosis (12 Core Patterns)

The candidate should be able to diagnose these common issues with correct kubectl commands and remediation steps:

### Pod Issues
- [ ] **ImagePullBackOff**: Verify image name/tag, check imagePullSecrets, verify registry access, check rate limiting
- [ ] **CrashLoopBackOff**: Check logs + previous logs, examine exit codes (137=OOM, 1=app error), verify env vars, check liveness probe timing, verify dependencies
- [ ] **Pending Pods**: Check node resources, node selectors/affinity, taints/tolerations, PVC availability, resource quotas
- [ ] **OOMKilled**: Confirm via pod status (exit code 137), check memory usage vs limits, profile for leaks, consider Guaranteed QoS

### Node Issues
- [ ] **Node NotReady**: Check kubelet status, journalctl logs, disk/memory/PID pressure, container runtime, network to API server
- [ ] **Disk Pressure**: Check disk usage (`df -h`), clean Docker/containerd cache, rotate logs, expand disk, configure kubelet GC

### Networking Issues
- [ ] **Pod-to-Pod Failure**: Check network policies, CNI plugin health, test connectivity (`ping`, `nslookup`), verify service endpoints, check Pod CIDR exhaustion
- [ ] **Service Not Accessible**: Verify endpoints exist, check pod label matching, verify readiness probes, check LoadBalancer controller, test port accessibility

### Storage Issues
- [ ] **PVC Pending**: Check StorageClass exists, verify dynamic provisioner running, check cloud credentials, verify capacity
- [ ] **Volume Mount Failure**: Check RWO multi-attach conflicts, verify PVC bound, check node-to-storage connectivity, check CSI driver health

### Resource Issues
- [ ] **Resource Quota Exceeded**: Check quota usage, identify resource consumers, right-size requests, consider namespace splitting
- [ ] **CPU Throttling**: Check usage vs limits, review throttling metrics, increase limits or remove for bursty workloads, use HPA

---

## Incident Response Framework

### Severity Classification
- [ ] **SEV-1**: Complete outage, data loss, security breach — immediate all-hands response
- [ ] **SEV-2**: Major degradation, significant user impact — response within 15 minutes
- [ ] **SEV-3**: Minor impairment, workaround available — response within 1 hour
- [ ] **SEV-4**: Cosmetic issues, minimal impact — response during business hours

### Response Phases
- [ ] **Detection**: Alerts, user reports, monitoring dashboards
- [ ] **Triage**: Assess impact, scope, gather initial data
- [ ] **Investigation**: Collect comprehensive diagnostics, identify root cause
- [ ] **Resolution**: Apply remediation, verify fix, monitor
- [ ] **Post-Incident**: Document, blameless post-mortem, preventive measures

### Common Incident Scenarios
- [ ] Complete cluster outage (API server, etcd, control plane)
- [ ] Service degradation (increased latency/errors, pod restarts)
- [ ] Node failure (NotReady, evictions, resource exhaustion)
- [ ] Storage issues (PVC pending, data access failures)
- [ ] Security incident (unauthorized access, suspicious behavior)

---

## Performance Troubleshooting

### Latency Diagnosis
- [ ] Identify the layer with latency (ingress, service, pod, dependency)
- [ ] Check CPU throttling as common latency cause
- [ ] Profile application performance (metrics endpoint, pprof)
- [ ] Test dependency connectivity and latency

### CPU Performance
- [ ] Monitor with `kubectl top nodes` and `kubectl top pods`
- [ ] Check throttling via cgroup metrics
- [ ] Understand the CPU limits debate (remove limits for bursty workloads)
- [ ] Use HPA for auto-scaling based on CPU utilization

### Memory Performance
- [ ] Identify OOMKilled pods and their patterns
- [ ] Profile memory usage over time
- [ ] Understand QoS classes: BestEffort → Burstable → Guaranteed
- [ ] Set requests = limits for Guaranteed QoS on critical workloads

### Network Performance
- [ ] DNS resolution latency (CoreDNS scaling, NodeLocal DNSCache)
- [ ] Service mesh overhead (if applicable)
- [ ] Network policy impact on performance
- [ ] Pod-to-pod latency measurement

### Storage I/O Performance
- [ ] Check IOPS limits for EBS volume type
- [ ] Monitor I/O wait on nodes
- [ ] Choose appropriate StorageClass (gp3 vs io2 vs local NVMe)
- [ ] Benchmark with fio for baseline

### Cluster-Wide Performance
- [ ] API server request latency
- [ ] etcd performance (use etcdctl check perf)
- [ ] Scheduler latency and pending pod queue
- [ ] Controller manager workqueue depth

---

## Helm Troubleshooting

### Release Management
- [ ] Diagnose stuck releases (pending-install, pending-upgrade)
- [ ] Handle failed upgrades (rollback, force delete, clean secrets)
- [ ] Debug "deployed but resources missing" scenarios
- [ ] Understand release history and revision management

### Installation & Upgrade
- [ ] Debug template rendering errors (`helm template`, `helm lint`, `--debug --dry-run`)
- [ ] Handle immutable field errors during upgrades
- [ ] Manage values layering conflicts (--set vs -f precedence)
- [ ] Use `--atomic` for safer upgrades

### Dependencies & Hooks
- [ ] Resolve dependency update failures
- [ ] Debug stuck or failed hooks (pre-install, post-upgrade)
- [ ] Configure hook deletion policies
- [ ] Manage subchart values correctly (nested under subchart name)

---

## Kubernetes Primitives Knowledge

### Workload Management
- [ ] Deployments: rolling updates, rollback, revision history
- [ ] StatefulSets: ordered deployment, stable network identity, persistent storage
- [ ] DaemonSets: per-node workloads, use cases (logging, monitoring, CNI)
- [ ] Jobs/CronJobs: batch processing, completion tracking, TTL cleanup

### Scheduling
- [ ] Resource requests and limits (CPU, memory, ephemeral storage)
- [ ] QoS classes: Guaranteed, Burstable, BestEffort
- [ ] Node selectors, affinity/anti-affinity
- [ ] Taints and tolerations
- [ ] Topology spread constraints
- [ ] Priority classes and preemption

### Networking
- [ ] Service types: ClusterIP, NodePort, LoadBalancer, ExternalName
- [ ] Ingress controllers and Ingress resources
- [ ] Network policies (ingress and egress rules)
- [ ] DNS resolution: CoreDNS, service discovery, headless services
- [ ] CNI plugins: VPC-CNI mechanics, ENI limits, prefix delegation

### Storage
- [ ] PersistentVolumes, PersistentVolumeClaims, StorageClasses
- [ ] Access modes: RWO, ROX, RWX
- [ ] Dynamic provisioning vs static provisioning
- [ ] CSI drivers: EBS CSI, EFS CSI
- [ ] Volume snapshots and backup

### Security
- [ ] RBAC: Roles, ClusterRoles, RoleBindings, ClusterRoleBindings
- [ ] Service Accounts and IRSA
- [ ] Pod Security Standards (Privileged, Baseline, Restricted)
- [ ] Security Contexts (runAsNonRoot, readOnlyRootFilesystem, capabilities)
- [ ] Network Policies for microsegmentation
- [ ] Secrets management (External Secrets Operator, Sealed Secrets)

### Observability
- [ ] Liveness probes: restart unhealthy containers
- [ ] Readiness probes: remove from service endpoints
- [ ] Startup probes: protect slow-starting containers
- [ ] Pod Disruption Budgets: protect availability during voluntary disruptions
- [ ] Resource metrics: metrics-server, `kubectl top`

---

## Diagnostic Commands (Expected Knowledge)

### Cluster Health
```bash
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods --all-namespaces | grep -v Running
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | tail -20
```

### Pod Diagnostics
```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl get pod <pod> -n <namespace> -o yaml
kubectl debug <pod> -it --image=busybox -n <namespace>
```

### Node Diagnostics
```bash
kubectl describe node <node>
kubectl top nodes
kubectl top pods --all-namespaces
kubectl get nodes -o json | jq '.items[].status.conditions'
```

### Network Diagnostics
```bash
kubectl get endpoints <service> -n <namespace>
kubectl get networkpolicies --all-namespaces
kubectl exec -it <pod> -- nslookup <service>.<namespace>.svc.cluster.local
kubectl exec -it <pod> -- nc -zv <target-ip> <port>
```

### Storage Diagnostics
```bash
kubectl get pvc,pv --all-namespaces
kubectl describe pvc <pvc> -n <namespace>
kubectl get storageclass
kubectl get volumeattachments
```

### RBAC Diagnostics
```bash
kubectl auth can-i <verb> <resource> --as=<user/sa>
kubectl auth can-i --list --as=system:serviceaccount:<ns>:<sa>
kubectl get rolebindings,clusterrolebindings --all-namespaces
```

---

## Scoring Application

When evaluating a Kubernetes answer:

1. Count how many of the relevant checklist items the candidate addressed.
2. Check whether the candidate followed the systematic troubleshooting workflow.
3. Evaluate if the candidate can classify the incident severity and describe appropriate response.
4. Flag dangerous omissions (e.g., no resource requests, no probes, no RBAC, ad-hoc debugging without methodology).
5. Apply the rubric from `rubrics.md` — Kubernetes section.

| Coverage | Score |
|----------|-------|
| 0-20% of relevant items | 1 (Awareness) |
| 20-50% | 2 (Competent) |
| 50-80%, follows systematic workflow | 3 (Proficient) |
| 80%+ with proactive depth, demonstrates incident response maturity | 4 (Expert) |
