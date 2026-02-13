# Scoring Rubrics

All domains use the same 0-4 scale. Each level has a label and explicit criteria.

This rubric is calibrated for a **DevOps Engineer position interview**. Scoring rewards:
- Formal, precise definitions of technologies and terms
- Clear comparison with alternatives ("why this, not that")
- Anchoring answers in the four pillars: security, scalability, availability, cost optimization
- Connecting infrastructure decisions to business impact

---

## Universal Scale

| Score | Label | Criteria |
|-------|-------|----------|
| 0 | **No Answer** | Blank, refusal, or completely incorrect (dangerous misconception). |
| 1 | **Awareness** | Recognizes the topic exists but cannot explain mechanics. Vague or buzzword-only answer. Cannot define the technology formally. |
| 2 | **Competent** | Correct high-level explanation. Misses important details, edge cases, or security implications. Can define the technology but cannot compare with alternatives. Could follow a runbook but not write one. |
| 3 | **Proficient** | Accurate and detailed. Covers edge cases or failure modes. Defines technologies precisely and compares with at least one alternative. Could design and implement the solution independently. Minor omissions only. |
| 4 | **Expert** | Deep, precise, and opinionated. Anticipates failure modes unprompted. Formally defines all terms, compares multiple alternatives with nuanced tradeoffs, and ties decisions to business/operational impact. Could mentor others. |

---

## Technology Justification Score (Cross-Cutting)

Every question also receives a separate **Technology Justification** score (0-4). This is tracked independently and reported in the debrief.

| Score | Criteria |
|-------|----------|
| 0 | Cannot name the technology being discussed. |
| 1 | Names the technology but gives a vague or incorrect definition. No comparison with alternatives. |
| 2 | Gives a correct but informal definition. Mentions one alternative but cannot articulate why the project chose this technology. |
| 3 | Provides a formal, interview-ready definition. Compares with 1-2 alternatives with specific tradeoffs. Explains why this choice fits this project. |
| 4 | Textbook-quality definition. Compares 2+ alternatives with nuanced tradeoffs (cost, complexity, team size, ecosystem maturity). Ties the choice to project-specific constraints (team size, budget, AWS-native ecosystem, GitOps model). |

**Scoring signals:**
- Does the candidate start with "X is..." (formal definition)?
- Do they mention what X does *in this project specifically*?
- Do they name at least one alternative and explain why it wasn't chosen?
- Do they tie the choice to a best-practice pillar (security, cost, scalability, availability)?

---

## Domain-Specific Rubric Extensions

Each domain below adds **specific criteria** that map onto the universal 0-4 scale. Use these to calibrate scores within each round.

---

### 1. Terraform & IaC

| Score | Additional Criteria |
|-------|-------------------|
| 0 | Cannot distinguish Terraform from other IaC tools. |
| 1 | Knows `terraform apply` exists. Cannot explain plan/apply cycle, state, or modules. |
| 2 | Understands plan/apply, remote state with locking, basic module structure (main.tf, variables.tf, outputs.tf). May not know `prevent_destroy`, lifecycle rules, or IRSA patterns. |
| 3 | Explains state locking, backend encryption, module composition, `prevent_destroy`, import workflows, IRSA for EKS addons. Knows when to use `data` sources vs resources. Understands blast radius of state operations. |
| 4 | Can design bootstrap-vs-main phase separation. Explains state file partitioning strategy, drift detection, `moved` blocks, provider aliasing, dynamic blocks for DRY code. Knows conventions: mandatory descriptions on all variables/outputs, `sensitive = true` flagging, `versions.tf` constraints, pre-commit pipeline (fmt, validate, tflint, checkov). |

---

### 2. Kubernetes

*Standards derived from `k8s-troubleshooter@devops-skills`.*

| Score | Additional Criteria |
|-------|-------------------|
| 0 | Cannot explain what a Pod is. |
| 1 | Knows Pods, Deployments, Services exist. Cannot explain scheduling, networking, or resource requests. |
| 2 | Understands Deployments, Services (ClusterIP/NodePort/LoadBalancer), ConfigMaps/Secrets, namespaces. Basic `kubectl` commands. May not understand RBAC, network policies, or Pod disruption budgets. |
| 3 | Explains resource requests/limits, liveness/readiness probes, PDBs, HPA, RBAC, node affinity/taints, init containers. Can troubleshoot CrashLoopBackOff, ImagePullBackOff, pending pods using a systematic approach. Follows the k8s-troubleshooter diagnostic workflow: gather context → triage → deep dive → root cause → remediation → verify. |
| 4 | Deep knowledge of CNI plugins (VPC-CNI for EKS), IRSA, CSI drivers, admission controllers, Pod Security Standards, network policies, custom schedulers. Can explain kube-proxy modes, DNS resolution chain, and etcd consistency guarantees. Demonstrates mastery of incident response (severity classification, structured response phases, blameless post-mortems). Can diagnose performance issues across CPU, memory, network, and storage layers. Knows when to use `kubectl debug`, ephemeral containers, and node-level diagnostics. |

**Key evaluation signals (from `k8s-troubleshooter@devops-skills`):**
- Does the candidate follow a systematic troubleshooting workflow rather than ad-hoc guessing?
- Can they classify incidents by severity (SEV-1 through SEV-4) and describe appropriate response?
- Do they know the common issue diagnostic patterns: ImagePullBackOff, CrashLoopBackOff, Pending Pods, OOMKilled, Node NotReady, Disk Pressure, networking failures, PVC Pending?
- Can they diagnose performance issues across multiple layers (application, Kubernetes, infrastructure)?
- Do they reference specific kubectl commands in the correct diagnostic order?
- Can they explain the Kubernetes eviction hierarchy (QoS classes, priority classes)?
- Do they understand Helm troubleshooting patterns (stuck releases, failed upgrades, values debugging)?

---

### 3. Helm

| Score | Additional Criteria |
|-------|-------------------|
| 0 | Does not know what Helm is. |
| 1 | Knows Helm installs "packages" into Kubernetes. Cannot explain charts, values, or releases. |
| 2 | Understands charts, values files, `helm install/upgrade`, template functions. May not understand values layering, hooks, or library charts. |
| 3 | Explains values hierarchy (chart defaults < -f values.yaml < --set), hooks (pre-install, post-upgrade), release history, rollback mechanics, subchart dependencies. Can debug template rendering with `helm template`. |
| 4 | Designs charts with library chart patterns, JSON schema validation for values, conditional resource inclusion, proper `_helpers.tpl` usage. Understands OCI registries, chart testing (`helm test`), semantic versioning for charts, and Helmfile/ArgoCD integration patterns. |

---

### 4. ArgoCD / GitOps

| Score | Additional Criteria |
|-------|-------------------|
| 0 | Cannot explain GitOps. |
| 1 | Knows "Git is the source of truth." Cannot explain sync, reconciliation, or Application CRDs. |
| 2 | Understands Application CRD, sync policies (manual/auto), prune, self-heal. Knows about the ArgoCD UI. May not understand sync waves, app-of-apps, or ApplicationSets. |
| 3 | Explains sync waves, resource hooks, app-of-apps pattern, health checks, diff customization, repo credentials management (SSH keys in Secrets Manager), multi-cluster patterns. Can troubleshoot sync failures and OutOfSync states. |
| 4 | Designs ApplicationSet generators (Git, cluster, matrix), progressive delivery integration (Argo Rollouts), notification policies, RBAC within ArgoCD (projects, roles), custom health checks, config management plugins. Understands reconciliation loop internals, cache behavior, and sharding for scale. |

---

### 5. AWS Architecture

| Score | Additional Criteria |
|-------|-------------------|
| 0 | Cannot name core AWS services. |
| 1 | Knows EC2, S3, IAM exist. Cannot explain VPCs, subnets, or security groups. |
| 2 | Understands VPC/subnets, security groups, IAM policies (allow/deny), S3 buckets, Route 53 basics, ALB/NLB. May not understand IRSA, NAT gateway cost implications, or CloudFront OAC. |
| 3 | Explains public/private subnet design, NAT gateway tradeoffs (single vs per-AZ), IRSA for EKS, CloudFront with S3 OAC + custom origin, ACM certificate validation (DNS-01), ExternalDNS with Route 53, ECR lifecycle policies. Understands least-privilege IAM. |
| 4 | Designs multi-account architectures, SCPs, cross-region DR, cost allocation tagging strategies, PrivateLink/VPC endpoints, S3 bucket policies with conditions, CloudFront signed URLs/cookies, WAF integration. Can calculate cost impact of architectural choices. |

---

### 6. Reliability / Observability / Incident Response

| Score | Additional Criteria |
|-------|-------------------|
| 0 | Cannot define SLO or explain why monitoring matters. |
| 1 | Knows "monitoring is important." Cannot define SLI/SLO/SLA or explain alerting. |
| 2 | Understands SLI/SLO/SLA distinction, basic Prometheus/Grafana stack, log aggregation. May not understand error budgets, alert fatigue, or runbook design. |
| 3 | Explains error budgets, multi-signal alerting (latency, errors, saturation, traffic), structured logging, distributed tracing, rollback procedures, blameless postmortems. Can design alerting that avoids false positives. |
| 4 | Designs SLO-based alerting with burn-rate windows, implements canary analysis, chaos engineering practices, on-call rotation design, incident classification frameworks. Understands OpenTelemetry, Thanos/Cortex for long-term metrics, and cost of observability at scale. |

---

### 7. Security

| Score | Additional Criteria |
|-------|-------------------|
| 0 | Cannot explain why secrets should not be in Git. |
| 1 | Knows "don't commit passwords." Cannot explain secret rotation, supply chain security, or network segmentation. |
| 2 | Understands Secrets Manager / external-secrets-operator pattern, `.gitignore` for sensitive files, non-root containers, basic network policies. May not understand IRSA trust policies, OPA/Gatekeeper, or image signing. |
| 3 | Explains IRSA trust policy conditions, ExternalSecrets SecretStore/ClusterSecretStore, secret rotation strategies, container image scanning (ECR scan-on-push), Pod Security Standards, network segmentation via security groups + network policies. |
| 4 | Designs zero-trust architectures, supply chain security (SLSA, Sigstore, admission policies for signed images), secrets rotation automation, blast radius analysis, compliance-as-code (OPA, Kyverno), audit logging (CloudTrail, Kubernetes audit logs), and incident containment procedures. |
