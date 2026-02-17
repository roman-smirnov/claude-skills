# Example Interview Transcript (Abbreviated)

This shows the expected flow for the unified DevOps Interviewer skill. Actual questions will vary based on repo content and user answers.

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
| ArgoCD / GitOps | App-of-apps pattern, auto-sync + prune + self-heal, 5 sync waves | modules/argocd/main.tf |
| CI/CD | Jenkinsfile (7 stages: build, unit, integration, e2e, tag, publish, deploy) | ../cowsay3d/Jenkinsfile |
| Docker | Multi-stage backend, nginx, MongoDB, 2 compose files | ../cowsay3d/Dockerfile, ../cowsay3d/docker-compose.yaml |
| AWS Infrastructure | VPC, IAM roles, ECR, CloudFront, Route 53, Secrets Manager | modules/vpc/, modules/eks/, modules/cdn/, bootstrap/ |
| Observability | Health/metrics endpoints, no Prometheus/Grafana config | ../cowsay3d/backend/ |
| Secrets Management | 6 Secrets Manager secrets, ESO IRSA role, SSH key for GitOps | bootstrap/main.tf, modules/eks/main.tf |
| DNS / CDN / Networking | Route 53 zone, CloudFront (S3 OAC + custom origin), ExternalDNS, cert-manager | modules/cdn/main.tf, bootstrap/main.tf |
| Scripts / Docs | bootstrap.sh, apply.sh, destroy.sh, kubeconfig.sh, architecture docs | scripts/, docs/ |

### Topic Availability

- Terraform & IaC: **Available**
- Kubernetes: **Available**
- GitOps / ArgoCD: **Available**
- CI/CD: **Available**
- AWS Architecture: **Available**
- Observability: **Limited** (health endpoints only, no monitoring stack)
```

---

## Phase 2: Topic & Scope Selection

**Interviewer:**
Select topics and scope for your interview:

```
Topics:
[x] Kubernetes         [x] Terraform         [x] GitOps/ArgoCD
[x] Observability      [x] AWS               [x] CI/CD
[x] Full Stack (all topics)

Scope:
( ) Quick    — 5-8 questions,  ~20 min
(x) Standard — 12-18 questions, ~45 min
( ) Deep     — 25-35 questions, ~90 min
```

> **User selects: Full Stack, Standard mode**

**Interviewer:**
Full Stack, Standard mode. I'll ask 12-18 questions distributed across all topics, plus cross-cutting security questions. Let's begin.

---

## Phase 3: Interview (Excerpts)

### Question 1: Terraform — Technology Comparison

**Interviewer:**
Your entire infrastructure is defined in Terraform with custom modules under `modules/`. Let's start with fundamentals.

**[DEFINE] What is Terraform, and why did you choose it instead of CloudFormation, Pulumi, or CDK?**

> **User:** Terraform is an Infrastructure as Code tool by HashiCorp that uses declarative HCL to provision cloud resources through a plan/apply lifecycle. I chose it over CloudFormation because Terraform has better state management, a richer module ecosystem, and it's cloud-agnostic. Pulumi uses real programming languages but I preferred declarative — easier to review in PRs. CDK generates CloudFormation under the hood and I didn't want that abstraction layer.

**Follow-up:** Your state backend uses S3 native locking at `backend.tf`. What is state locking, and why native S3 locking instead of S3 + DynamoDB?

> **User:** State locking prevents two people from running apply concurrently and corrupting state. S3 native locking, introduced in Terraform 1.6, uses conditional writes so you don't need DynamoDB at all. Simpler, cheaper, one fewer resource.

### Score: Terraform — Technology Choice & State Backend
```
Rating: 72/100 — Proficient
Tech Justification: 92/100 — Formal definition, compared 3 alternatives with specific tradeoffs,
  tied choice to project needs (declarative, PR-reviewable, simpler state management).
Strengths: Clean formal definition. Good comparison across CFN, Pulumi, CDK. Correctly
  identified S3 native locking in Terraform 1.6 and DynamoDB cost elimination.
Gap: Did not mention that Terraform's plan output is more readable than CFN changesets.
  Did not mention provider version pinning in versions.tf.
Key point for interview: "I chose Terraform for its declarative plan/apply lifecycle,
  strong module system, and cloud-agnostic design. S3 native locking simplified state
  management by eliminating DynamoDB."
```

**Model answer:** Terraform is an open-source IaC tool by HashiCorp that uses declarative HCL and a plan/apply lifecycle to provision cloud resources. In this project, it manages the complete AWS stack: VPC, EKS, IAM, Route 53, CloudFront, and Secrets Manager across a bootstrap phase and a main phase. I chose Terraform over CloudFormation because of its explicit `terraform plan` review step (crucial for PR-based workflows), reusable module system (5 custom modules), and provider-agnostic design. CDK adds an abstraction layer that obscures what's deployed, and Pulumi's imperative model makes diffs harder to review. The S3 backend uses native locking (Terraform 1.6+), which eliminates the DynamoDB table — saving cost and complexity.

---

### Question 2: Kubernetes — Failure Mode (CrashLoopBackOff)

**Interviewer:**
Your EKS cluster runs workloads across 3 nodes configured in `modules/eks/main.tf`.

**A Pod is stuck in `CrashLoopBackOff`. Walk me through your debugging steps: what kubectl commands do you run, in what order, and what are you looking for at each step?**

> **User:** First I'd check pod status with `kubectl describe pod <name> -n <namespace>` to see events and container states. I'd look at exit code — 137 means OOMKilled, 1 means app error. Then logs with `kubectl logs <pod>` and `kubectl logs <pod> --previous`. I'm looking for startup errors, missing env vars, or connection failures. If logs don't help, I'd check liveness probe configuration and resource limits.

**Follow-up:** Logs show the app can't connect to MongoDB in a different namespace. How do you determine if this is DNS, network policy, or MongoDB availability?

> **User:** I'd test DNS with `kubectl exec <pod> -- nslookup mongodb.database.svc.cluster.local`. If DNS resolves but connection fails, check network policies in both namespaces. Then verify MongoDB endpoints: `kubectl get endpoints mongodb -n database`. If endpoints exist, test connectivity: `kubectl exec <pod> -- nc -zv <ip> 27017`.

### Score: Kubernetes — CrashLoopBackOff Troubleshooting
```
Rating: 70/100 — Proficient
Tech Justification: 68/100 — Good use of kubectl in correct diagnostic order.
Strengths: Correct exit code interpretation (137 = OOM). Good cross-namespace DNS debugging.
  Systematic workflow: describe → logs → previous logs → targeted diagnosis.
Gap: Did not mention checking namespace-level events (`kubectl get events -n <ns>`).
  Did not mention init containers for dependency checking as prevention.
Key point for interview: "I follow a systematic troubleshooting workflow: describe for
  state and events, logs for app errors, then targeted network/storage/resource diagnostics."
```

**Model answer:** CrashLoopBackOff means the container repeatedly starts and crashes, with Kubernetes applying exponential backoff. The systematic workflow starts with `kubectl describe pod` for exit codes, states, and events. Exit code 137 = OOMKill (kernel killed), non-zero = application error. Next, `kubectl logs --previous` captures the last container's output. For dependency failures, DNS resolution (`nslookup`), network policy inspection, and endpoint verification isolate the layer. Always debug the root cause service first (MongoDB), not the dependent.

---

### Question 3: GitOps — Tradeoff

**Interviewer:**
ArgoCD is configured with `selfHeal: true` and `prune: true` in `modules/argocd/main.tf`. The SSH key comes from Secrets Manager.

**Define what self-heal means in ArgoCD. Then: someone kubectl-applies a hotfix at 2 AM. What happens?**

> **User:** Self-heal means ArgoCD automatically reverts cluster drift to match Git. If someone applies a hotfix with kubectl, ArgoCD detects drift within its 3-minute reconciliation interval and overwrites the hotfix. The change is lost.

**Follow-up:** So how should the 2 AM emergency be handled within GitOps?

> **User:** Commit the fix to the GitOps repo, merge, and let ArgoCD sync. For urgency, manually trigger `argocd app sync` after the commit. Then backport the fix to the application repo.

### Score: ArgoCD — Self-Heal and Drift
```
Rating: 80/100 — Expert
Tech Justification: 75/100 — Good definition with reconciliation interval detail.
Strengths: Correctly explained self-heal, knew 3-minute default, described proper
  emergency workflow (commit → sync → backport).
Gap: Did not mention prune means ArgoCD also DELETES resources not in Git — a distinct
  and dangerous behavior separate from self-heal. Did not discuss sync windows.
Key point for interview: "Self-heal with prune ensures the cluster matches Git. Emergency
  fixes must go through Git — ArgoCD reverts direct kubectl changes within 3 minutes."
```

**Model answer:** ArgoCD self-heal is automatic reconciliation of cluster state back to Git's desired state when drift is detected. In this project, self-heal is enabled alongside prune, meaning ArgoCD both reverts modified resources AND deletes resources that exist in the cluster but not in Git. If an engineer kubectl-applies a hotfix, the reconciliation loop (default 3 minutes) overwrites it. The correct workflow: commit to GitOps repo, optionally trigger `argocd app sync`, then backport. Sync windows can restrict auto-sync during maintenance periods for added safety.

---

### Question 4: AWS — Debugging (WEAK answer example)

**Interviewer:**
Your EKS cluster uses IRSA for the external-secrets-operator, configured in `modules/eks/main.tf`.

**The external-secrets-operator suddenly reports `AccessDenied` when reading from Secrets Manager. Walk me through your debugging approach.**

> **User:** I'd check if the IAM policy is correct... maybe the permissions got changed. I'd look at the policy in the AWS console and make sure it has Secrets Manager read access.

**Follow-up:** That's a start, but there are multiple layers that could cause `AccessDenied`. Can you describe a more systematic approach?

> **User:** Um, I'd also check the trust policy I guess. And maybe the service account annotation? I'm not sure of the exact order.

### Score: AWS — IRSA AccessDenied Debugging
```
Rating: 35/100 — Competent (low end)
Tech Justification: 25/100 — Did not define IRSA formally. Could not describe the
  identity chain or systematic debugging approach.
Strengths: Recognized IAM policy and trust policy as potential causes.
Gap: No systematic debugging order. Did not start with identity verification
  (`aws sts get-caller-identity`). Did not check OIDC issuer/audience conditions.
  Did not mention explicit denies from permission boundaries or SCPs. Could not
  articulate the full IRSA identity chain.
Key point for interview: "For AccessDenied, always verify identity first (sts
  get-caller-identity), then trust policy, then permission policy, then deny layers."
```

**Model answer:** IRSA (IAM Roles for Service Accounts) maps a Kubernetes service account to an IAM role via OIDC federation, granting Pods scoped temporary STS credentials. When debugging `AccessDenied`: (1) verify pod identity with `aws sts get-caller-identity` from within the pod, (2) confirm the service account annotation matches the IAM role ARN, (3) inspect the trust policy for correct OIDC issuer URL and audience/subject conditions, (4) verify the permission policy has the required Secrets Manager actions scoped to the correct resource ARNs, (5) check for explicit denies from permission boundaries or SCPs. This systematic order avoids guesswork and isolates the failing layer.

---

### Question 5: Observability — Best Practice Pillar (Availability)

**Interviewer:**
Your project has health endpoints but no Prometheus or Grafana deployed.

**(Availability)** You're on-call and the application is slow. Without a monitoring stack, how do you diagnose the problem? What observability would you add first, and what SLIs would you define?

> **User:** Without monitoring, I'd start with kubectl: `kubectl top pods` for resource usage, `kubectl logs` for error patterns, and `kubectl describe` for events. I'd check if it's a single pod or cluster-wide. For observability, I'd add Prometheus with ServiceMonitors for the backend, define SLIs for request latency (p99 from histogram), error rate (5xx / total), and availability (successful health checks / total). I'd set SLOs like 99.9% availability and p99 latency under 500ms, then create Grafana dashboards and Alertmanager rules based on burn rate.

**Follow-up:** You mentioned burn-rate alerting. Explain what that is and why it's better than a simple threshold alert.

> **User:** Burn-rate alerting measures how fast you're consuming your error budget relative to the SLO window. A 1x burn rate means you'll exactly exhaust the budget by end of window. A 14.4x burn rate in a 5-minute window means a severe incident. It's better than threshold alerts because it accounts for the acceptable error rate — a 0.1% error rate might be fine if your SLO is 99.9%, but a threshold alert at 0.1% would fire constantly.

### Score: Observability — SLI/SLO Design
```
Rating: 60/100 — Proficient
Tech Justification: 58/100 — Good definitions of SLI/SLO and burn rate. Compared
  burn-rate to threshold alerting with concrete reasoning.
Strengths: Correct kubectl-based diagnosis without monitoring. Good SLI selection
  (latency, error rate, availability). Accurate burn-rate explanation with numbers.
Gap: Did not mention Golden Signals or RED method as frameworks. Did not discuss
  recording rules for expensive queries. Missing: what Alertmanager routing looks
  like (severity, grouping, receivers).
Key point for interview: "I'd define SLIs for latency, error rate, and availability,
  set SLOs, and use burn-rate alerting to detect incidents based on budget consumption
  rather than arbitrary thresholds."
```

**Model answer:** Without monitoring, kubectl provides initial diagnosis: `kubectl top` for resource utilization, logs for error patterns, and describe for scheduling/probe events. The first observability addition should be Prometheus (time-series monitoring that scrapes metrics via a pull model) with Grafana (multi-datasource visualization platform). Define SLIs using the Golden Signals: latency (histogram-based p99), error rate (5xx/total using `rate()` over counters), and availability. Set SLOs (e.g., 99.9% availability = 43 minutes/month error budget) and implement burn-rate alerting, which detects rapid error-budget consumption over multi-window periods. A 14.4x burn rate in 5 minutes signals a severe incident, while a 1x rate is nominal — this avoids the noise of static threshold alerts that don't account for acceptable error rates.

---

## Phase 4: Debrief (Excerpt)

```
## Interview Debrief

### Score Summary
| Topic | Questions | Avg Score | Tech Justification Avg | Level |
|-------|-----------|-----------|----------------------|-------|
| Terraform & IaC | 3 | 74 | 82 | Proficient |
| Kubernetes | 5 | 72 | 70 | Proficient |
| GitOps / ArgoCD | 3 | 78 | 72 | Proficient |
| AWS Architecture | 4 | 55 | 48 | Competent |
| Observability | 2 | 60 | 58 | Proficient |
| Security (cross-cutting) | 1 | 68 | 65 | Proficient |

**Overall: 67 / 100 — Solid Competent+, approaching Proficient**

### Best Practice Pillar Coverage
| Pillar | Questions Asked | Avg Score | Strongest Area | Weakest Area |
|--------|----------------|-----------|----------------|--------------|
| Security | 4 | 68 | IRSA concepts, secrets management | IAM debugging methodology |
| Scalability | 3 | 65 | Node scaling concepts | HPA, VPC-CNI IP limits |
| Availability | 4 | 72 | ArgoCD self-heal, AZ reasoning | SPOF identification depth |
| Cost Optimization | 3 | 55 | Instance type awareness | Monthly cost estimation |

### Kubernetes Readiness
- Systematic troubleshooting workflow: **demonstrated**
- Incident response awareness: **adequate**
- Common issue diagnosis: **7/12 common issues covered confidently**
- Performance troubleshooting depth: **needs work**

### AWS Readiness
- Service-selection rationale: **adequate**
- Failure-domain and HA reasoning: **adequate**
- IAM debugging methodology: **needs work** (could not articulate systematic AccessDenied debugging)
- Cost governance: **needs work**

### Technology Justification Readiness
- Well-justified: Terraform vs alternatives, ArgoCD sync model, EKS managed control plane
- Struggled to compare: VPC-CNI vs alternatives (not tested), CloudFront OAC vs OAI (vague)
- Missing formal definitions: IRSA (used correctly but not defined in AWS question), PDB, topology spread

### Strengths
- Strong Terraform fundamentals and state management knowledge
- Good GitOps understanding: self-heal, drift, emergency workflow
- Solid Kubernetes troubleshooting methodology
- Good SLI/SLO/burn-rate conceptual knowledge

### Gaps
1. IAM debugging methodology (AccessDenied — could not articulate identity chain)
2. IRSA formal definition and trust policy mechanics
3. Cost estimation and FinOps awareness
4. Missing formal definitions for several technologies
5. Observability stack implementation experience (conceptual only)

### Interview-Ready Talking Points
Top 5 things to confidently say in an actual interview:
1. "I designed a two-phase Terraform architecture — bootstrap for long-lived resources with prevent_destroy, main phase for destroyable infra — to minimize blast radius."
2. "ArgoCD self-heal with prune ensures the cluster always matches Git. Emergency fixes must go through Git — even at 2 AM — because ArgoCD reverts direct kubectl changes within the reconciliation interval."
3. "I follow a systematic Kubernetes troubleshooting workflow: describe for state and events, logs for app errors, then targeted diagnostics — always identifying cascading failures early."
4. "For observability, I'd define SLIs for latency, error rate, and availability, set SLOs, and use burn-rate alerting to detect incidents based on error-budget consumption rather than static thresholds."
5. "S3 native locking in Terraform 1.6 eliminated the need for DynamoDB, reducing both cost and operational complexity for state management."

### Targeted Drill Plan
| Priority | Gap | Pillar | Suggested Exercise | Resources |
|----------|-----|--------|-------------------|-----------|
| 1 | IAM debugging | Security | Practice IRSA AccessDenied debugging: verify identity → trust → permission → deny layers. Write the exact commands. | AWS IAM troubleshooting docs, `modules/eks/main.tf` IRSA config |
| 2 | Missing definitions | All | Write formal 1-sentence definitions for: IRSA, OAC, PDB, topology spread, VPC-CNI prefix delegation, burn-rate alerting. Practice saying them aloud. | `checklists/technology-comparisons.md` |
| 3 | Cost estimation | Cost | Calculate full monthly AWS cost of this stack. Compare t3a vs t4g vs spot. Identify top 3 optimization opportunities. | AWS Pricing Calculator, `checklists/best-practices.md` |
| 4 | Observability setup | Availability | Add Prometheus + Grafana via ArgoCD. Define 3 SLIs, create dashboards and burn-rate alerts. | Prometheus Operator Helm chart, SRE Workbook Ch. 4-5 |
| 5 | K8s performance | Scalability | Deploy metrics-server, run `kubectl top`, create HPA, load-test to observe scaling. | `topics/kubernetes.md` performance section |
```
