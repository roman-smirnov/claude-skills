# Example Interview Transcript (Abbreviated)

This shows the expected flow for a DevOps Engineer interview preparation session. Actual questions will vary based on repo content and user answers.

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
- Kubernetes: **Available** (EKS config, IRSA, addons)
- Helm: **Limited** (ArgoCD chart only, no standalone charts)
- ArgoCD / GitOps: **Available** (app-of-apps, sync policies, SSH keys)
- AWS Architecture: **Available** (VPC, IAM, CDN, DNS, ECR)
- Reliability / Observability: **Limited** (health/metrics endpoints, no Prometheus config)
- Security: **Available** (IRSA, secrets, non-root containers, ECR scanning)
```

What interview mode would you like?

> **User selects: Full loop**

---

## Phase 3: Interview (Excerpts)

### Terraform Question — Technology Comparison

**Interviewer:**
Your entire infrastructure is defined in Terraform with custom modules under `modules/`. Let's start with fundamentals.

**What is Terraform, and why did you choose it instead of CloudFormation, Pulumi, or CDK for this project?**

> **User:** Terraform is an Infrastructure as Code tool by HashiCorp that lets you define cloud resources declaratively in HCL. You write what you want and Terraform figures out how to get there through a plan/apply lifecycle. I chose it over CloudFormation because Terraform has better state management, a richer module ecosystem, and it's cloud-agnostic even though I'm only using AWS. Pulumi lets you use real programming languages but I preferred the declarative approach — it's easier to review in PRs. CDK generates CloudFormation under the hood and I didn't want that abstraction layer.

**Follow-up:** You mentioned state management. Your state backend is S3 with native locking at `backend.tf`. What is Terraform state locking, and why did you choose S3 native locking instead of the traditional S3 + DynamoDB approach?

> **User:** State locking prevents two people from running apply at the same time and corrupting the state file. Traditionally you'd use a DynamoDB table alongside S3 for locking. S3 native locking was introduced in Terraform 1.6 — it uses S3 conditional writes so you don't need DynamoDB at all. I chose it because it's simpler, cheaper — no DynamoDB table to manage — and one fewer resource to maintain.

### Score: Terraform — Technology Choice & State Backend
```
Rating: 3/4 — Proficient
Tech Justification: 4/4 — Formal definition, compared 3 alternatives with specific tradeoffs,
  tied choice to project needs (declarative, PR-reviewable, simpler state management).
Strengths: Clean formal definition of Terraform. Good comparison across CloudFormation
  (vendor lock-in), Pulumi (imperative vs declarative), CDK (abstraction concern).
  Correctly identified S3 native locking introduction in Terraform 1.6 and the DynamoDB
  cost/complexity elimination.
Gap: Did not mention that Terraform's `plan` output is more readable than CFN changesets,
  which matters for the PR review workflow. Did not mention the `iac-terraform@devops-skills`
  convention of pinning provider versions in `versions.tf`.
Key point for interview: "I chose Terraform for its declarative plan/apply lifecycle,
  strong module system, and cloud-agnostic design. The S3 backend with native locking
  simplified state management by eliminating DynamoDB."
```

**Model answer:** Terraform is an open-source IaC tool by HashiCorp that uses declarative HCL configuration and a plan/apply lifecycle to provision cloud resources. In this project, it manages the complete AWS stack: VPC, EKS, IAM, Route 53, CloudFront, and Secrets Manager across a bootstrap phase and a main phase. I chose Terraform over CloudFormation because of its explicit `terraform plan` review step (crucial for PR-based workflows), reusable module system (I wrote 5 custom modules), and provider-agnostic design. CloudFormation changesets are harder to read, CDK adds an abstraction layer that obscures what's actually deployed, and Pulumi's imperative model makes infrastructure harder to review in diffs. The S3 backend uses native locking (Terraform 1.6+), which eliminates the DynamoDB table the traditional setup required — saving both cost and operational complexity.

---

### Kubernetes Question — Best Practice Pillar (Scalability)

**Interviewer:**
Your EKS node group in `modules/eks/main.tf` is configured with `desired_size = 3`, `min_size = 3`, `max_size = 3` — a fixed-size cluster.

**What is a Cluster Autoscaler and what is Karpenter? Why don't you have auto-scaling enabled, and what are the risks of running fixed-size nodes?**

> **User:** Cluster Autoscaler is a Kubernetes component that watches for pending pods that can't be scheduled due to insufficient resources, and scales up the node group by increasing the ASG desired count. Karpenter is a newer AWS-native node provisioning tool that's faster because it provisions nodes directly through EC2 fleet API instead of going through ASGs. I don't have auto-scaling because this is a small project with predictable load — 3 nodes is enough. The risk is if load spikes or if one node goes down, I might not have enough capacity. With 3 fixed nodes, I'm also paying for compute I might not need during low traffic.

**Follow-up 1:** If you were to add auto-scaling, would you choose Cluster Autoscaler or Karpenter? What's the tradeoff?

> **User:** Karpenter is probably the better choice for EKS because it's faster at provisioning — it can launch the right instance type for the workload instead of just scaling the existing node group. Cluster Autoscaler is simpler and more battle-tested. For a small project, either works, but Karpenter is the direction AWS is pushing.

**Follow-up 2:** From a cost optimization perspective, your nodes are `t3a.medium` (burstable, AMD). What happens if your workloads consistently use more CPU than the burst baseline? And would Graviton-based instances (t4g) be a better choice?

> **User:** T3a instances have burst credits — if you exceed the baseline consistently you'll deplete credits and get throttled. You'd need to monitor CPU credit balance. Graviton t4g instances are ARM-based and about 20% cheaper for similar performance. The tradeoff is you'd need to build ARM Docker images, and some base images might not support ARM. For this project it's probably worth it since Python runs fine on ARM.

### Score: Kubernetes — Node Scaling & Cost
```
Rating: 3/4 — Proficient
Tech Justification: 3/4 — Good definitions of Cluster Autoscaler and Karpenter with
  accurate comparison. Missed Karpenter's consolidation feature (bin-packing existing nodes).
Strengths: Correctly identified the fixed-size risk, understood burst credit mechanics,
  knew Graviton savings percentage, and flagged the ARM image build requirement.
Gap: Did not mention spot instances (up to 90% savings with Karpenter's spot support).
  Did not discuss node affinity or taints for mixed instance types. An expert would
  calculate: 3x t3a.medium at $0.0376/hr = ~$81/month vs 3x t4g.medium at $0.0336/hr = ~$73/month.
Key point for interview: "I chose fixed-size t3a.medium nodes for predictable cost on a
  small project. For production scale, I'd add Karpenter for right-sized provisioning
  with spot instances, and migrate to Graviton for the 20% cost reduction."
```

**Model answer:** A Cluster Autoscaler is a Kubernetes controller that monitors pending Pods and adjusts the node group size via Auto Scaling Groups. Karpenter is AWS's next-generation node provisioner that bypasses ASGs, directly launching right-sized EC2 instances and bin-packing workloads for efficiency. In this project, fixed-size nodes (3x t3a.medium, ~$92/month) were chosen for simplicity and cost predictability. The risk is twofold: no headroom if a node fails (2/3 capacity), and wasted compute during low traffic. To optimize, I'd add Karpenter with spot instance support (up to 90% savings for fault-tolerant workloads) and migrate to Graviton t4g.medium instances for ~20% cost reduction. The t3a burst credit model means sustained CPU above the 20% baseline will throttle performance — monitoring `CPUCreditBalance` is essential.

---

### ArgoCD Question — Failure Mode

**Interviewer:**
ArgoCD is configured in `modules/argocd/main.tf` with `selfHeal: true` and `prune: true` on the root Application. The SSH key for the GitOps repo comes from Secrets Manager secret `gitlab-cowsay3d-gitops-key`.

**ArgoCD's sync policy has `selfHeal: true`. First, define what self-heal means in ArgoCD. Then: someone kubectl-applies a hotfix directly to the cluster at 2 AM. What happens?**

> **User:** Self-heal in ArgoCD means that if the live cluster state drifts from what's defined in the Git repository, ArgoCD will automatically revert the change to match Git. So if someone applies a hotfix at 2 AM with kubectl, ArgoCD will detect the drift within its reconciliation interval — default is 3 minutes — and overwrite the hotfix with whatever's in Git. The hotfix is lost.

**Follow-up:** So how should that 2 AM emergency be handled properly within your GitOps workflow?

> **User:** The engineer should commit the hotfix to the GitOps repo and let ArgoCD sync it. If it's truly urgent, they could commit to Git and then manually trigger an ArgoCD sync to skip the polling interval. After the incident, the fix should be cherry-picked or properly merged into the main branch of the application repo so it's included in the next pipeline build.

### Score: ArgoCD — Self-Heal Drift
```
Rating: 3/4 — Proficient
Tech Justification: 3/4 — Good definition of self-heal with reconciliation interval.
  Could have defined ArgoCD itself more formally at the start.
Strengths: Correctly explained self-heal behavior, knew the 3-minute default interval,
  described proper GitOps emergency workflow (commit to Git, manual sync, backport).
Gap: Did not mention that `prune: true` means ArgoCD will also DELETE resources that
  exist in the cluster but not in Git — a separate and dangerous behavior from self-heal.
  Did not discuss ArgoCD's sync windows (can restrict auto-sync during maintenance windows)
  or notification hooks to alert when drift is detected. An expert would also note that
  kubectl-applied resources with ArgoCD's tracking label will be reverted, but resources
  without the label won't be touched.
Key point for interview: "Self-heal with prune ensures the cluster always matches Git.
  Emergency fixes must go through Git — even at 2 AM — because ArgoCD will revert
  direct kubectl changes within the reconciliation interval."
```

**Model answer:** ArgoCD self-heal is the automatic reconciliation of cluster state back to the desired state in Git when drift is detected. In this project, self-heal is enabled alongside prune on the root Application, meaning ArgoCD will both revert modified resources AND delete resources that exist in the cluster but not in Git — this is a critical distinction. If an engineer kubectl-applies a hotfix at 2 AM, ArgoCD's reconciliation loop (default: 3 minutes) detects the drift and overwrites it. The correct emergency workflow: commit to the GitOps repo, optionally trigger `argocd app sync` to bypass polling, then backport the fix into the application repo. For added safety, ArgoCD sync windows can restrict auto-sync during maintenance periods, and notification integrations can alert the team when drift is detected.

---

## Phase 4: Debrief (Excerpt)

```
## Interview Debrief

### Score Summary
| Round | Questions | Avg Score | Tech Justification Avg | Level |
|-------|-----------|-----------|----------------------|-------|
| Terraform & IaC | 5 | 3.2 | 3.6 | Proficient |
| Kubernetes | 4 | 3.0 | 3.0 | Proficient |
| Helm | 3 | 2.3 | 2.0 | Competent |
| ArgoCD / GitOps | 4 | 3.5 | 3.3 | Proficient+ |
| AWS Architecture | 4 | 2.8 | 2.5 | Competent+ |
| Reliability | 3 | 2.0 | 1.7 | Competent |
| Security | 3 | 3.0 | 2.7 | Proficient |

**Overall: 2.8 / 4.0 — Solid Competent+, approaching Proficient**

### Best Practice Pillar Coverage
| Pillar | Questions Asked | Avg Score | Strongest Area | Weakest Area |
|--------|----------------|-----------|----------------|--------------|
| Security | 6 | 3.0 | IRSA, secrets management | Supply chain, image signing |
| Scalability | 4 | 2.5 | Node scaling concepts | VPC-CNI IP limits, HPA |
| Availability | 4 | 2.3 | ArgoCD self-heal | SPOF identification, DR plans |
| Cost Optimization | 3 | 2.7 | Instance type awareness | Monthly cost estimation, VPC endpoints |

### Technology Justification Readiness
- Well-justified: Terraform vs alternatives, S3 native locking, ArgoCD vs Flux, EKS managed control plane, IRSA vs alternatives
- Struggled to compare: Helm vs Kustomize (vague), VPC-CNI vs Cilium (no comparison), cert-manager DNS-01 vs HTTP-01 (not discussed)
- Missing formal definitions: CloudFront OAC, ExternalDNS, Pod Disruption Budget

### Strengths
- Strong GitOps understanding: sync waves, app-of-apps, drift handling, emergency workflow
- Good Terraform state management: bootstrap separation, native locking, prevent_destroy rationale
- Solid Kubernetes debugging methodology: systematic, uses right tools
- Security-conscious: mentions IRSA, least privilege, non-root containers unprompted
- Good technology justification for core tools (Terraform, ArgoCD, EKS)

### Gaps
1. Helm deep mechanics: hooks, library charts, release history internals
2. AWS networking edge cases: ENI limits, NAT gateway failover, VPC endpoint costs
3. Observability: no mention of SLO/error budget frameworks, alerting strategy
4. Terraform CI enforcement: knows checkov exists but hasn't defined blocking policies
5. Cost estimation: could not approximate monthly AWS spend
6. Formal definitions missing for several secondary technologies

### Interview-Ready Talking Points
Top 5 things to confidently say in an actual interview:
1. "I designed a two-phase Terraform architecture — bootstrap for long-lived resources with prevent_destroy, main phase for destroyable infra — to minimize blast radius during infrastructure changes."
2. "I chose ArgoCD's pull-based GitOps model over push-based CI deploys because it provides continuous reconciliation, drift detection via self-heal, and a complete audit trail through Git history."
3. "All AWS access from Kubernetes uses IRSA — IAM Roles for Service Accounts via OIDC federation — which provides pod-level, least-privilege access using short-lived STS tokens instead of long-lived credentials."
4. "The CI/CD pipeline separates concerns: Jenkins handles build and test (push-based CI), then writes the image tag to a GitOps repo, and ArgoCD handles deployment (pull-based CD) with ordered sync waves."
5. "CloudFront serves the frontend from S3 via OAC and proxies API requests to the backend NLB, providing global caching, HTTPS termination, and HTTP/2+3 — all configured in Terraform with automated DNS via ExternalDNS."

### Targeted Drill Plan
| Priority | Gap | Pillar | Suggested Exercise | Resources |
|----------|-----|--------|-------------------|-----------|
| 1 | Cost estimation | Cost | Calculate the full monthly AWS cost of this stack. Document it. Compare t3a.medium vs t4g.medium vs spot pricing. | AWS Pricing Calculator, `checklists/best-practices.md` cost section |
| 2 | Observability | Availability | Add Prometheus + Grafana via ArgoCD. Define 3 SLIs for the backend (/health, /api latency, error rate). Set up alerts. | SRE Workbook Ch. 4-5, Prometheus Operator Helm chart |
| 3 | Missing definitions | All | Write a one-sentence formal definition for: OAC, ExternalDNS, PDB, CSI Driver, sync wave, OIDC federation, S3 native locking. Practice saying them aloud. | `checklists/technology-comparisons.md` |
| 4 | Helm deep dive | Scalability | Create a library chart for shared labels. Use JSON schema for values validation. Deploy via ArgoCD. | Helm docs: Library Charts |
| 5 | VPC endpoints | Cost | Add VPC endpoints for S3, ECR, and STS. Calculate NAT cost savings. | AWS VPC Endpoints documentation |
| 6 | Terraform CI | Security | Add pre-commit hook: fmt -check -> validate -> tflint -> checkov. Make checkov blocking. | `iac-terraform@devops-skills` CI/CD templates |
```
