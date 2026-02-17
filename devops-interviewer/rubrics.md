# Scoring Rubrics

All domains use the same 0-100 scale. Each level has a label, score range, and explicit criteria.

This rubric is calibrated for a **DevOps Engineer position interview**. Scoring rewards:
- Formal, precise definitions of technologies and terms
- Clear comparison with alternatives ("why this, not that")
- Anchoring answers in the four pillars: security, scalability, availability, cost optimization
- Connecting infrastructure decisions to business impact

---

## Universal Scale

| Score Range | Label | Criteria |
|-------------|-------|----------|
| 0 | **No Answer** | Blank, refusal, or completely incorrect (dangerous misconception). |
| 1–25 | **Awareness** | Recognizes the topic exists but cannot explain mechanics. Vague or buzzword-only answer. Cannot define the technology formally. Lower end: only recognizes the name. Higher end: attempts a vague explanation. |
| 26–50 | **Competent** | Correct high-level explanation. Misses important details, edge cases, or security implications. Can define the technology but cannot compare with alternatives. Could follow a runbook but not write one. Lower end: surface-level correctness. Higher end: solid understanding with notable gaps. |
| 51–75 | **Proficient** | Accurate and detailed. Covers edge cases or failure modes. Defines technologies precisely and compares with at least one alternative. Could design and implement the solution independently. Minor omissions only. Lower end: good with a few gaps. Higher end: strong, nearly complete. |
| 76–100 | **Expert** | Deep, precise, and opinionated. Anticipates failure modes unprompted. Formally defines all terms, compares multiple alternatives with nuanced tradeoffs, and ties decisions to business/operational impact. Could mentor others. Lower end: deep knowledge with minor polish needed. Higher end: textbook-quality mastery. |

---

## Technology Justification Score (Cross-Cutting)

Every question also receives a separate **Technology Justification** score (0-100). This is tracked independently and reported in the debrief.

| Score Range | Criteria |
|-------------|----------|
| 0 | Cannot name the technology being discussed. |
| 1–25 | Names the technology but gives a vague or incorrect definition. No comparison with alternatives. |
| 26–50 | Gives a correct but informal definition. Mentions one alternative but cannot articulate why the project chose this technology. |
| 51–75 | Provides a formal, interview-ready definition. Compares with 1-2 alternatives with specific tradeoffs. Explains why this choice fits this project. |
| 76–100 | Textbook-quality definition. Compares 2+ alternatives with nuanced tradeoffs (cost, complexity, team size, ecosystem maturity). Ties the choice to project-specific constraints (team size, budget, AWS-native ecosystem, GitOps model). |

**Scoring signals:**
- Does the candidate start with "X is..." (formal definition)?
- Do they mention what X does *in this project specifically*?
- Do they name at least one alternative and explain why it wasn't chosen?
- Do they tie the choice to a best-practice pillar (security, cost, scalability, availability)?

---

## Domain-Specific Rubric Extensions

Each domain below adds **specific criteria** that map onto the universal 0-100 scale. Use these to calibrate scores within each round.

---

### 1. Terraform & IaC

| Score Range | Additional Criteria |
|-------------|-------------------|
| 0 | Cannot distinguish Terraform from other IaC tools. |
| 1–25 | Knows `terraform apply` exists. Cannot explain plan/apply cycle, state, or modules. |
| 26–50 | Understands plan/apply, remote state with locking, basic module structure (main.tf, variables.tf, outputs.tf). May not know `prevent_destroy`, lifecycle rules, or IRSA patterns. |
| 51–75 | Explains state locking, backend encryption, module composition, `prevent_destroy`, import workflows, IRSA for EKS addons. Knows when to use `data` sources vs resources. Understands blast radius of state operations. |
| 76–100 | Can design bootstrap-vs-main phase separation. Explains state file partitioning strategy, drift detection, `moved` blocks, provider aliasing, dynamic blocks for DRY code. Knows conventions: mandatory descriptions on all variables/outputs, `sensitive = true` flagging, `versions.tf` constraints, pre-commit pipeline (fmt, validate, tflint, checkov). |

**Key evaluation signals:**
- Does the candidate mention `terraform fmt -check` and `terraform validate` as mandatory pre-commit steps?
- Do they know that every variable must have a `description` and `type`?
- Do they flag secrets as `sensitive = true`?
- Can they explain remote state backend requirements (S3 + locking, encryption, versioning)?
- Do they understand module validation (required files, descriptions, examples directory)?

---

### 2. Kubernetes

| Score Range | Additional Criteria |
|-------------|-------------------|
| 0 | Cannot explain what a Pod is. |
| 1–25 | Knows Pods, Deployments, Services exist. Cannot explain scheduling, networking, or resource requests. |
| 26–50 | Understands Deployments, Services (ClusterIP/NodePort/LoadBalancer), ConfigMaps/Secrets, namespaces. Basic `kubectl` commands. May not understand RBAC, network policies, or Pod disruption budgets. |
| 51–75 | Explains resource requests/limits, liveness/readiness probes, PDBs, HPA, RBAC, node affinity/taints, init containers. Can troubleshoot CrashLoopBackOff, ImagePullBackOff, pending pods using a systematic approach. Follows the systematic diagnostic workflow: gather context → triage → deep dive → root cause → remediation → verify. |
| 76–100 | Deep knowledge of CNI plugins (VPC-CNI for EKS), IRSA, CSI drivers, admission controllers, Pod Security Standards, network policies, custom schedulers. Can explain kube-proxy modes, DNS resolution chain, and etcd consistency guarantees. Demonstrates mastery of incident response (severity classification, structured response phases, blameless post-mortems). Can diagnose performance issues across CPU, memory, network, and storage layers. Knows when to use `kubectl debug`, ephemeral containers, and node-level diagnostics. |

**Key evaluation signals:**
- Does the candidate follow a systematic troubleshooting workflow rather than ad-hoc guessing?
- Can they classify incidents by severity (SEV-1 through SEV-4) and describe appropriate response?
- Do they know the common issue diagnostic patterns: ImagePullBackOff, CrashLoopBackOff, Pending Pods, OOMKilled, Node NotReady, Disk Pressure, networking failures, PVC Pending?
- Can they diagnose performance issues across multiple layers (application, Kubernetes, infrastructure)?
- Do they reference specific kubectl commands in the correct diagnostic order?
- Can they explain the Kubernetes eviction hierarchy (QoS classes, priority classes)?
- Do they understand Helm troubleshooting patterns (stuck releases, failed upgrades, values debugging)?

---

### 3. Helm

| Score Range | Additional Criteria |
|-------------|-------------------|
| 0 | Does not know what Helm is. |
| 1–25 | Knows Helm installs "packages" into Kubernetes. Cannot explain charts, values, or releases. |
| 26–50 | Understands charts, values files, `helm install/upgrade`, template functions. May not understand values layering, hooks, or library charts. |
| 51–75 | Explains values hierarchy (chart defaults < -f values.yaml < --set), hooks (pre-install, post-upgrade), release history, rollback mechanics, subchart dependencies. Can debug template rendering with `helm template`. |
| 76–100 | Designs charts with library chart patterns, JSON schema validation for values, conditional resource inclusion, proper `_helpers.tpl` usage. Understands OCI registries, chart testing (`helm test`), semantic versioning for charts, and Helmfile/ArgoCD integration patterns. |

---

### 4. ArgoCD / GitOps

| Score Range | Additional Criteria |
|-------------|-------------------|
| 0 | Cannot explain GitOps. |
| 1–25 | Knows "Git is the source of truth." Cannot explain sync, reconciliation, or Application CRDs. |
| 26–50 | Understands Application CRD, sync policies (manual/auto), prune, self-heal. Knows about the ArgoCD UI. May not understand sync waves, app-of-apps, or ApplicationSets. |
| 51–75 | Explains sync waves, resource hooks, app-of-apps pattern, health checks, diff customization, repo credentials management (SSH keys in Secrets Manager), multi-cluster patterns. Can troubleshoot sync failures and OutOfSync states. |
| 76–100 | Designs ApplicationSet generators (Git, cluster, matrix), progressive delivery integration (Argo Rollouts), notification policies, RBAC within ArgoCD (projects, roles), custom health checks, config management plugins. Understands reconciliation loop internals, cache behavior, and sharding for scale. |

**Key evaluation signals:**
- Can they explain reconcile/drift precisely?
- Do they distinguish `selfHeal` from `prune` risk?
- Can they articulate app-of-apps vs ApplicationSet fit?
- Do they enforce Git-based promotion instead of manual cluster mutation?
- Can they reason about ArgoCD control-plane failure and recovery?

---

### 5. CI/CD

| Score Range | Additional Criteria |
|-------------|-------------------|
| 0 | Cannot explain CI/CD stages or pipeline concepts. |
| 1–25 | Understands build/test/deploy at buzzword level only. Cannot explain Jenkins pipeline types or Docker build strategy. |
| 26–50 | Can describe pipeline stages and basic promotion flow. Understands declarative Jenkinsfile basics and Docker multi-stage builds at a high level. May not understand credential management, artifact lifecycle, or CI-to-GitOps handoff. |
| 51–75 | Clearly explains CI-to-GitOps handoff, Jenkins agent architecture, credential binding patterns, Docker layer caching optimization, image scanning integration, and artifact promotion workflow. Understands why CI should NOT run `kubectl apply` directly. |
| 76–100 | Designs resilient multi-environment pipeline with governance, shared libraries, ephemeral agents, parallel stages, SBOM generation, supply chain security, and measurable deployment quality signals. Can explain Jenkins controller HA, Docker BuildKit cache mounts, and pipeline audit trail design. |

**Key evaluation signals:**
- Does the candidate understand the CI/CD-to-GitOps boundary?
- Can they explain Jenkins agent architecture (controller vs agents)?
- Do they know credential management patterns (withCredentials, IRSA for CI)?
- Can they design an image tagging strategy (commit SHA, never "latest" in prod)?
- Do they understand pipeline security (least-privilege CI service accounts, image scanning)?

---

### 6. AWS Architecture

| Score Range | Additional Criteria |
|-------------|-------------------|
| 0 | Cannot name core AWS services. |
| 1–25 | Knows EC2, S3, IAM exist. Cannot explain VPCs, subnets, or security groups. |
| 26–50 | Understands VPC/subnets, security groups, IAM policies (allow/deny), S3 buckets, Route 53 basics, ALB/NLB. May not understand IRSA, NAT gateway cost implications, or CloudFront OAC. |
| 51–75 | Explains public/private subnet design, NAT gateway tradeoffs (single vs per-AZ), IRSA for EKS, CloudFront with S3 OAC + custom origin, ACM certificate validation (DNS-01), ExternalDNS with Route 53, ECR lifecycle policies. Understands least-privilege IAM. |
| 76–100 | Designs multi-account architectures, SCPs, cross-region DR, cost allocation tagging strategies, PrivateLink/VPC endpoints, S3 bucket policies with conditions, CloudFront signed URLs/cookies, WAF integration. Can calculate cost impact of architectural choices. |

**Key evaluation signals:**
- Service selection rationale is explicit, not default-driven.
- Failure domains and acceptable risks are identified clearly.
- Candidate debugs `AccessDenied` systematically (identity → trust → permission → deny layers).
- Candidate can narrate packet/request path clearly.
- Candidate identifies network SPOFs and mitigation ROI.

---

### 7. Observability

| Score Range | Additional Criteria |
|-------------|-------------------|
| 0 | Cannot define SLO or explain why monitoring matters. |
| 1–25 | Knows "monitoring is important." Cannot define SLI/SLO/SLA or explain alerting. Cannot explain metric types or logging architecture. |
| 26–50 | Understands SLI/SLO/SLA distinction, basic Prometheus/Grafana stack, log aggregation. May not understand error budgets, alert fatigue, cardinality, or EFK pipeline details. |
| 51–75 | Explains error budgets, multi-signal alerting (latency, errors, saturation, traffic), structured logging, distributed tracing, PromQL correctness (rate/increase/histogram), recording rules, EFK pipeline flow, ILM lifecycle, Kibana troubleshooting. Can design alerting that avoids false positives. |
| 76–100 | Designs SLO-based alerting with burn-rate windows, implements canary analysis, understands Thanos/Cortex for long-term metrics, Elasticsearch shard strategy and mapping cardinality, on-call rotation design, incident classification frameworks, and cost of observability at scale. |

**Key evaluation signals:**
- Does the candidate correctly use metric types (counter, gauge, histogram)?
- Can they write correct PromQL (rate over counters, not raw division)?
- Do they understand cardinality and recording rule strategy?
- Can they debug missing logs in Kibana (pipeline localization)?
- Do they connect alerts to SLO/error-budget impact?

---

### 8. Security

| Score Range | Additional Criteria |
|-------------|-------------------|
| 0 | Cannot explain why secrets should not be in Git. |
| 1–25 | Knows "don't commit passwords." Cannot explain secret rotation, supply chain security, or network segmentation. |
| 26–50 | Understands Secrets Manager / external-secrets-operator pattern, `.gitignore` for sensitive files, non-root containers, basic network policies. May not understand IRSA trust policies, OPA/Gatekeeper, or image signing. |
| 51–75 | Explains IRSA trust policy conditions, ExternalSecrets SecretStore/ClusterSecretStore, secret rotation strategies, container image scanning (ECR scan-on-push), Pod Security Standards, network segmentation via security groups + network policies. |
| 76–100 | Designs zero-trust architectures, supply chain security (SLSA, Sigstore, admission policies for signed images), secrets rotation automation, blast radius analysis, compliance-as-code (OPA, Kyverno), audit logging (CloudTrail, Kubernetes audit logs), and incident containment procedures. |

---

## Scoring Consistency Rules

1. Use both scores for every question: **Rating** and **Tech Justification**.
2. Tie ratings to explicit rubric criteria, not subjective impression.
3. Penalize unsafe operational behavior (e.g., normalizing manual cluster mutation, broad IAM grants, disabling alerts without replacement).
4. Reward candidates who proactively address failure modes, rollback safety, and prevention.
5. Favor structured, reproducible reasoning over volume of words.
