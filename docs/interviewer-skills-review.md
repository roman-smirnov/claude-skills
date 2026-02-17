# Interviewer Skills - In-Depth Review

**Date:** 2026-02-16
**Scope:** All 5 interviewer skills (Kubernetes, Terraform, GitOps, Observability, AWS)

---

## Executive Summary

The interviewer skills are **well-designed and comprehensive** — collectively scoring around 8/10. They share a strong common architecture (4-phase interview, dual scoring, repo-grounding, 4-pillar best practices) and are well-calibrated for mid/senior DevOps interviews. However, there are **systematic gaps** that appear across all skills, **significant cross-skill overlap**, and **missing coverage of key project stack technologies** (especially CI/CD with Jenkins/Docker, and observability tools).

---

## 1. What Works Well (Keep As-Is)

### Structural Strengths
- **4-phase interview flow** (Repo Intake → Mode Selection → Interview Loop → Debrief) is clear, professional, and mirrors real interview practices
- **Dual scoring system** (Rating + Technology Justification) is excellent — prevents candidates from scoring well on execution alone without understanding *why*
- **Repo-grounding** via intake procedure ensures questions reference actual project artifacts, not just theory
- **4-pillar framework** (Security, Scalability, Availability, Cost) aligns with AWS Well-Architected and is industry-standard
- **5 mandatory question categories** per round (failure-mode, debugging, tradeoff, tech comparison, best practice) ensure comprehensive coverage
- **Tool safety constraints** (read-only, prohibited commands) prevent accidental damage during repo analysis
- **Technology comparison tables** with formal definitions — forces precise language, which is exactly what interviewers look for

### Per-Skill Standouts
| Skill | Standout Strength |
|-------|-------------------|
| **Kubernetes** | Systematic 6-phase troubleshooting workflow, 14 common issue patterns, incident response framework |
| **Terraform** | Deep state management coverage, lifecycle rules, drift detection, import/migration workflows |
| **GitOps** | ArgoCD vs Flux comparison matrix, secrets management comparison (SOPS vs Sealed Secrets vs ESO), OpenGitOps principles |
| **Observability** | SLO/burn-rate alerting, metric cardinality control, structured logging fundamentals, ILM strategy |
| **AWS** | Multi-pillar evaluation, IAM deep dive, networking/VPC depth, cost optimization with real estimates |

---

## 2. What Should Be Removed or Simplified

### 2.1 Cross-Skill Redundancy
The **biggest structural problem** is significant content duplication across skills:

- **Kubernetes checklist** (`checklists/kubernetes.md`) appears in nearly identical form in both `devops-kubernetes-interviewer` and `devops-gitops-interviewer`. Same for `best-practices.md` and `technology-comparisons.md`.
- **Rubrics** share ~70% identical content across all 5 skills (the universal scale, Technology Justification section, and several domain extensions are copy-pasted).
- **SKILL.md** files share the same Phase 1-4 structure, behavioral rules, tool safety section, and debrief format — with only the round emphasis changing.
- **Best-practices checklists** repeat the same 4-pillar structure with overlapping items across all skills.

**Recommendation:** Extract shared content into a `_shared/` directory:
```
_shared/
  interview-framework.md      # Phases, behavioral rules, tool safety
  universal-rubric.md          # 0-100 scale, Technology Justification
  best-practices-template.md   # 4-pillar framework
  technology-comparisons-base.md  # Common tech entries
```
Each skill's SKILL.md would then `include` the shared framework and only define skill-specific overrides (round order, domain emphasis, unique checklists).

### 2.2 Overly Verbose Tool Safety Sections
Each SKILL.md lists ~20 prohibited commands individually. Replace with: "No mutating operations. Read-only tools only (`cat`, `grep`, `find`, `ls`, `kubectl get/describe/logs`)."

### 2.3 Generic Question Variants
Every question template includes both a repo-grounded and generic variant. The generic variants add bulk. Consider keeping only the repo-grounded versions and adding a single note: "If artifacts are unavailable, ask the same question in generic form without file path references."

### 2.4 Monthly Cost Estimate Tables
The specific dollar-amount tables in `best-practices.md` will go stale as AWS pricing changes. Replace with a methodology: "Candidate should be able to estimate costs using AWS Pricing Calculator or back-of-envelope math" rather than hardcoded numbers.

---

## 3. What Should Be Improved

### 3.1 Scoring Calibration (All Skills)
**Problem:** The rubrics define score ranges (0, 1-25, 26-50, 51-75, 76-100) but don't clarify borderline cases.
- What's the difference between a 49 and a 51?
- If a candidate gets 4 of 6 troubleshooting steps right, what score?
- How do round scores aggregate into an overall score?

**Fix:** Add calibration examples at each boundary (25/26, 50/51, 75/76) showing sample answers and their scores.

### 3.2 Handling Weak Candidates (All Skills)
**Problem:** All example transcripts show proficient-to-expert answers (scores 65-92). No skill demonstrates how to handle:
- "I don't know" responses
- Dangerously wrong answers (e.g., recommending `Action: *` in IAM)
- Candidates who score consistently below 25

**Fix:** Add a second example transcript per skill showing a mid-level candidate (scores 30-55) who struggles, receives hints, and partially recovers. Show scoring for weak answers.

### 3.3 Time Management (All Skills)
**Problem:** No skill specifies interview duration or per-question time limits.
- Short mode: how many questions? 10? 15?
- Full loop: 25-35 questions across 7 rounds — is that 1 hour? 2 hours?
- What if a candidate's slow answers eat into later rounds?

**Fix:** Add timing guidance to SKILL.md:
```
Short mode: ~30 min, 10-12 questions, 3-4 rounds
Full loop: ~60-90 min, 25-35 questions, 7 rounds
Per question: ~3-5 min (question + answer + follow-up + scoring)
```

### 3.4 Progressive Difficulty (All Skills)
**Problem:** Questions within each round don't have difficulty tags. The interviewer can't adapt to candidate strength.

**Fix:** Tag questions as `[MID]`, `[SENIOR]`, or `[EXPERT]`. Start with MID, escalate if the candidate scores well, de-escalate if they struggle.

### 3.5 Example Transcripts Need More Variety (All Skills)
Current transcripts only show 2-3 questions from 1-2 rounds. They should demonstrate:
- At least one question from each round type
- A technology comparison question with formal definition scoring
- A best-practice pillar question
- A follow-up chain that probes a weakness
- An "I don't know" response with graceful handling
- The full debrief with actionable drill plan

### 3.6 Soft Skills Not Assessed (All Skills)
**Problem:** No skill evaluates communication quality, explanation clarity, or mentoring ability — which are key differentiators at the senior level.

**Fix:** Add an optional "Communication Quality" modifier (+/- 5 points) that tracks:
- Can the candidate explain complex topics simply?
- Do they structure their answers logically?
- Do they acknowledge uncertainty honestly?

---

## 4. What Should Be Added

### 4.1 Missing Skill: CI/CD Interviewer (Jenkins + Docker)
**This is the single biggest gap.** The student project uses Jenkins and Docker/Docker Compose extensively, but **no dedicated interviewer skill covers CI/CD**. CI/CD appears as a minor round in other skills but is never the primary focus.

A `devops-cicd-interviewer` should cover:
- **Jenkins:** Declarative vs scripted pipelines, shared libraries, credential management, agent architecture, plugin security, pipeline-as-code
- **Docker:** Multi-stage builds, layer caching, security scanning (Trivy), image tagging strategies (SHA vs semver vs latest), compose for local dev
- **Pipeline security:** SBOM generation, supply chain attacks, secrets in CI, artifact signing
- **Integration with GitOps:** How CI triggers CD (image push → ArgoCD sync)
- **Build optimization:** Parallel stages, caching, artifact reuse
- **Pipeline observability:** Build metrics, failure rates, MTTR

### 4.2 Missing Coverage: Observability Stack in Non-Observability Skills
Prometheus, Grafana, Fluent-bit, ElasticSearch, and Kibana are in the student project but are only deeply covered in the Observability interviewer. The other 4 skills mention them in passing or label their observability round as "Limited."

**Fix:** Each skill's `technology-comparisons.md` should include entries for:
| Technology | Missing From |
|-----------|-------------|
| Prometheus | Kubernetes, Terraform, GitOps, AWS |
| Grafana | Kubernetes, Terraform, GitOps, AWS |
| Fluent-bit | Kubernetes, Terraform, GitOps, AWS |
| ElasticSearch + Kibana | Kubernetes, Terraform, GitOps, AWS |

### 4.3 Missing Topics Per Skill

#### Kubernetes Interviewer — Add:
- StatefulSet-specific troubleshooting (pod identity, PVC retention, rolling updates)
- Custom Resource Definitions (CRDs) and operator patterns
- Kubernetes version upgrades and version skew
- Pod Security Admission (replacing deprecated Pod Security Policies)
- HPA troubleshooting (scaling lag, metric collection failures)
- Multi-tenancy patterns (namespace isolation, ResourceQuotas, LimitRanges)

#### Terraform Interviewer — Add:
- Terraform workspaces (when to use vs separate state files)
- Terraform testing (Terratest, terraform-compliance, OPA/Conftest)
- Moved blocks workflow (refactoring, renaming resources)
- Dynamic blocks for DRY code
- Import workflow best practices (write config → import → plan → verify)
- CI/CD rubric for Jenkins pipeline stages

#### GitOps Interviewer — Add:
- Argo Rollouts (progressive delivery: canary, blue-green, metric-based rollback)
- Argo Notifications (Slack/email for sync success/failure)
- ArgoCD Image Updater (automated image tag updates)
- ArgoCD Projects depth (RBAC boundaries, destination/source restrictions)
- ArgoCD 3.x migration details (annotation-based tracking)
- GitOps-specific Kubernetes issues (ArgoCD-managed resource deletion, sync wave deadlocks)

#### Observability Interviewer — Add:
- Distributed tracing round (OpenTelemetry, Jaeger/Tempo, trace correlation, sampling)
- AWS-native observability (CloudWatch integration, Container Insights, X-Ray)
- Long-term metrics storage (Thanos, Cortex, VictoriaMetrics)
- Multi-cluster/multi-region federation
- Synthetic monitoring (blackbox exporter, uptime probes)
- Application instrumentation quality assessment

#### AWS Interviewer — Add:
- EKS deep dive (control/data plane, add-ons, node groups vs Fargate, VPC CNI, IRSA)
- ECR (image scanning, lifecycle policies, replication)
- EBS CSI driver and volume management for Kubernetes
- ALB Ingress Controller vs NLB tradeoffs
- AWS Systems Manager (Parameter Store, Session Manager)
- AWS Config and compliance monitoring
- GuardDuty/Security Hub for threat detection
- Multi-account governance (Organizations, SCPs)
- Container-specific security (runtime protection, pod security)

### 4.4 Cross-Skill Architecture Questions
Add an optional "Senior+ Architecture Round" to each skill for candidates scoring 75+:
- "Design a multi-cluster GitOps topology for 10 teams..."
- "Design a DR strategy with RTO < 15 min for the entire platform..."
- "Your observability costs doubled in 3 months — diagnose and propose fixes..."

These open-ended design questions separate senior from staff-level engineers.

### 4.5 Pillar Tagging in Score Blocks
Score blocks currently don't tag which pillar (security/scalability/availability/cost) a question addresses. Since the debrief requires pillar coverage stats, each score block should explicitly note: `Pillar: Cost Optimization` (or whichever applies).

---

## 5. Cross-Skill Overlap Analysis

| Content Area | K8s | TF | GitOps | Obs | AWS | Recommendation |
|-------------|-----|-----|--------|-----|-----|----------------|
| Kubernetes checklist | Full | Partial | Full (duplicate) | Partial | Partial | Shared base + skill-specific extensions |
| Best practices (4 pillars) | Full | Full | Full | Full | Full | Extract to shared template |
| Technology comparisons | 16 entries | 16 entries | 16 entries | 8 entries | 12 entries | Shared base + per-skill additions |
| Rubric universal scale | Full | Full | Full | Full | Full | Extract to shared rubric |
| Behavioral rules | Full | Full | Full | Full | Full | Extract to shared framework |
| Tool safety | Full | Full | Full | Full | Full | Extract to shared framework |
| Repo intake structure | Full | Full | Full | Full | Full | Shared template + per-skill scan steps |

**Impact of deduplication:** Would reduce total content by ~40% while making maintenance much easier (fix once, applies everywhere).

---

## 6. Prioritized Recommendations

### P0 — Critical (Address Before Use)
1. **Create CI/CD interviewer skill** — Jenkins + Docker are core project technologies with zero dedicated coverage
2. **Add observability stack entries** to all technology-comparisons files — students will be asked about Prometheus/Grafana/EFK in any interview
3. **Add weak candidate examples** to transcripts — current examples only show strong performance, making scoring calibration impossible

### P1 — High Priority
4. **Extract shared content** into `_shared/` to eliminate duplication and simplify maintenance
5. **Add scoring calibration examples** at boundary levels (25/26, 50/51, 75/76)
6. **Add timing guidance** to all SKILL.md files
7. **Add missing Kubernetes topics** (StatefulSets, CRDs, Pod Security Admission, HPA)
8. **Add missing Terraform topics** (workspaces, testing, moved blocks)
9. **Add Argo Rollouts and Notifications** to GitOps interviewer
10. **Add distributed tracing round** to Observability interviewer
11. **Add EKS/ECR deep dive** to AWS interviewer

### P2 — Medium Priority
12. **Add difficulty tags** (`[MID]`, `[SENIOR]`, `[EXPERT]`) to all question templates
13. **Add pillar tagging** to score block format
14. **Add communication quality modifier** to rubrics
15. **Add architecture round** (optional, for 75+ candidates)
16. **Simplify tool safety sections** (replace command lists with principle)
17. **Remove hardcoded cost estimates** (replace with estimation methodology)
18. **Add "I don't know" handling guidance** to all SKILL.md files

### P3 — Nice to Have
19. **Add more follow-up depth** examples to transcripts
20. **Add cross-skill scope clarification** (when to use which interviewer)
21. **Consolidate generic question variants** into a single note
22. **Add OpenTelemetry coverage** to Observability interviewer
23. **Add AWS managed services comparisons** (AMP, AMG, OpenSearch Service)
24. **Add multi-account governance** to AWS interviewer

---

## 7. Per-Skill Quality Scores

| Skill | Structure | Depth | Completeness | Project Alignment | Overall |
|-------|-----------|-------|-------------|-------------------|---------|
| Kubernetes | 10/10 | 9/10 | 8/10 | 9/10 | **9/10** |
| Terraform | 10/10 | 9/10 | 8/10 | 8/10 | **8.5/10** |
| GitOps | 10/10 | 9/10 | 7.5/10 | 8/10 | **8.5/10** |
| Observability | 9/10 | 9/10 | 7/10 | 8/10 | **8/10** |
| AWS | 9/10 | 8/10 | 7/10 | 7/10 | **7.5/10** |

**Aggregate: 8.3/10** — Strong foundation, needs targeted improvements to fully cover the student project stack.
