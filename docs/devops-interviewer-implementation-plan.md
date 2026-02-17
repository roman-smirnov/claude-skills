# devops-interviewer — Implementation Plan

**Date:** 2026-02-16
**Goal:** Create a single combined interviewer skill from the 5 existing skills + a new CI/CD topic.
**Phases:** 2 (Base framework → Topic modules)

---

## Directory Structure (Final State)

```
devops-interviewer/
  SKILL.md                        # Unified entry point with shared framework
  repo-intake.md                  # Unified intake (all layers in one scan)
  rubrics.md                      # Universal scale + all 8 domain extensions
  example-transcript.md           # Shows topic selection + multi-topic flow
  checklists/
    best-practices.md             # Shared 4-pillar framework (all topics merged)
    technology-comparisons.md     # All tech entries in one unified table
  topics/
    kubernetes.md                 # K8s questions + checklist (merged)
    terraform.md                  # TF questions + checklist (merged)
    gitops.md                     # ArgoCD/GitOps questions + checklist (merged)
    observability.md              # Obs questions + checklist (merged)
    aws.md                        # AWS questions + checklist (merged)
    cicd.md                       # NEW: Jenkins + Docker questions + checklist
  agents/openai.yaml              # Agent interface
```

---

## Phase 1: Base Framework

Phase 1 creates the shared skeleton that all topics plug into. After Phase 1, the skill is structurally complete but has no topic content — it references `topics/*.md` files that don't exist yet.

### 1.1 Create `SKILL.md`

The main entry point. Combines the shared framework from all 5 existing SKILL.md files into one, with dynamic topic and scope selection.

**Source material:** All 5 existing SKILL.md files (they share ~80% identical structure).

**Structure:**

```
---
name: devops-interviewer
description: Conduct a structured, repo-aware DevOps mock interview across
  Kubernetes, Terraform, GitOps/ArgoCD, Observability, AWS, and CI/CD.
  Supports topic selection, configurable question count, and scoring with
  debrief across security, scalability, availability, and cost optimization.
---

# DevOps Interviewer Skill

## Purpose
## Invocation
## Interview Preparation Framing
  - Technology Functionality (why chosen, how it works, compare alternatives)
  - Best Practices (4 pillars: security, scalability, availability, cost)
  - Deployment Strategy
  - Monitoring & Observability
  - Operations & Troubleshooting

## Phase 1: Repo Intake
  - Follow `repo-intake.md`
  - Output: unified Repo Map table (all layers)
  - Classify each topic as Available / Limited / Generic

## Phase 2: Topic & Scope Selection
  - Present topic menu (multi-select checkboxes):
    [ ] Kubernetes    [ ] Terraform     [ ] GitOps/ArgoCD
    [ ] Observability [ ] AWS           [ ] CI/CD
    [ ] Full Stack (all topics)
  - Present scope menu:
    ( ) Quick    — 5-8 questions,  ~20 min, 2-3 topics max
    ( ) Standard — 12-18 questions, ~45 min, 3-5 topics
    ( ) Deep     — 25-35 questions, ~90 min, all selected topics
  - Wait for explicit user selection before proceeding
  - Calculate question distribution: distribute evenly across selected topics,
    with at least 2 questions per selected topic

## Phase 3: Interview Loop
  - Selected topics determine which rounds are active
  - Round order (when included):
    1. Terraform & IaC — `topics/terraform.md`
    2. Kubernetes — `topics/kubernetes.md`
    3. Helm (embedded in kubernetes.md and gitops.md)
    4. GitOps / ArgoCD — `topics/gitops.md`
    5. CI/CD — `topics/cicd.md`
    6. AWS Architecture — `topics/aws.md`
    7. Observability & Incident Response — `topics/observability.md`
    8. Security (cross-cutting, always included as at least 1-2 questions)

  ### Per-Question Flow
    (identical to existing skills — ask, wait, follow-up, wait, score, model answer)

  ### Question Requirements per Round
    (5 mandatory categories: failure-mode, debug, tradeoff, tech comparison, best practice pillar)
    For Quick mode: at least 3 of 5 categories per topic
    For Standard/Deep: all 5 categories per topic

  ### Topic-Specific Enhancement Sections
    Each topic file defines its own "enhancement" instructions that the skill
    applies when scoring that topic's answers. These replace the per-skill
    "Round Enhancement" sections.

  ### Repo Grounding Rules
    (identical — reference real paths, never fabricate)

## Phase 4: Debrief
  - Score Summary table (per topic)
  - Best Practice Pillar Coverage table
  - Per-Topic Readiness sections (dynamically included based on selected topics)
  - Technology Justification Readiness
  - Strengths / Gaps
  - Interview-Ready Talking Points (top 5)
  - Targeted Drill Plan

## Tool Safety
  ### Allowed (read-only)
    (unified superset of all 5 existing allowed lists)
  ### PROHIBITED
    Principle: "No mutating operations against any infrastructure, cluster,
    registry, CI system, or Git remote."
    (concise list of categories, not individual commands)

## Behavioral Rules
  (unified — 11 rules, identical across all existing skills)
```

**Key decisions:**
- Topic selection replaces the per-skill "Mode Selection" (which offered skill-specific deep-dive modes like "Kubernetes-only" or "IAM + Security"). The new system is strictly topic-based.
- Security is cross-cutting: always included as 1-2 questions regardless of topic selection, because every topic has security implications.
- Helm is not a standalone topic. Helm questions are embedded in the Kubernetes and GitOps topic files, since Helm always appears in the context of one of those.
- The debrief dynamically includes readiness sections only for selected topics (no empty "Kubernetes Readiness" section if K8s wasn't selected).

**What changes from existing SKILL.md files:**
- Frontmatter `name` and `description` are new
- Phase 2 is completely rewritten (topic + scope selection replaces fixed modes)
- Phase 3 round list references `topics/*.md` instead of skill-specific checklists
- Phase 4 debrief uses conditional sections
- Tool Safety is simplified (principle-based, not exhaustive command list)
- Round Enhancement sections are removed from SKILL.md (moved into topic files)

**What stays identical:**
- Phase 1 (Repo Intake) flow
- Per-Question Flow (ask → wait → follow-up → wait → score → model answer)
- Question Requirements (5 mandatory categories)
- Repo Grounding Rules
- Behavioral Rules (all 11)
- Score block format

### 1.2 Create `repo-intake.md`

Unified intake that merges all scan steps from the 5 existing `repo-intake.md` files.

**Source material:** All 5 existing repo-intake.md files.

**Structure:**
- Step 1: Directory Structure (from K8s skill — already comprehensive)
- Step 2: Terraform Analysis (from TF skill)
- Step 3: Kubernetes Analysis (from K8s skill)
- Step 4: Helm Analysis (from K8s skill)
- Step 5: ArgoCD / GitOps Analysis (from GitOps skill — most detailed)
- Step 6: CI/CD Analysis (from K8s skill + expand for Jenkins specifics)
- Step 7: Docker Analysis (from K8s skill)
- Step 8: AWS Infrastructure Analysis (NEW — from AWS skill's steps on IAM, networking, compute, data, edge)
- Step 9: Observability Analysis (NEW — from Obs skill's steps on Prometheus, Grafana, EFK, alerting)
- Step 10: Secrets Management (from GitOps skill)
- Step 11: DNS / CDN / Networking (from K8s skill)
- Step 12: Scripts and Docs (from K8s skill)
- Step 13: Sibling Repos (from K8s skill)
- Output Format: unified Repo Map table + per-topic availability classification

**Key merges:**
- The AWS skill's repo-intake has detailed IAM/networking/compute/data/edge scanning steps not in other skills → merge these into Step 8
- The Observability skill's repo-intake has Prometheus/Grafana/EFK-specific scanning → merge into Step 9
- The GitOps skill's repo-intake has the most detailed ArgoCD analysis → use as base for Step 5
- All other steps are already well-covered by the K8s skill's intake

**Deduplication:** Remove duplicate glob patterns that appear in multiple existing intakes. Consolidate into Step 1.

### 1.3 Create `rubrics.md`

Unified rubric with all 8 domain extensions.

**Source material:** All 5 existing rubrics.md files.

**Structure:**
- Universal Scale (identical across all skills — take from any one)
- Technology Justification Score (identical — take from any one)
- Domain-Specific Rubric Extensions:
  1. Terraform & IaC (from TF skill)
  2. Kubernetes (from K8s skill)
  3. Helm (from K8s skill)
  4. ArgoCD / GitOps (from K8s/GitOps skill)
  5. CI/CD (NEW — Jenkins, Docker, pipeline security, artifact management)
  6. AWS Architecture (from K8s skill)
  7. Observability (from K8s skill + Obs skill's more detailed version)
  8. Security (from K8s skill)

**Key additions:**
- NEW CI/CD domain extension (0-100 criteria for Jenkins, Docker, pipeline design)
- Observability domain gets expanded with Obs skill's richer criteria (metric types, EFK, SLO/burn-rate)

**Deduplication:** The universal scale and Tech Justification sections are identical across all 5 skills. Write once.

### 1.4 Create `checklists/best-practices.md`

Unified 4-pillar checklist merging all 5 existing best-practices files.

**Source material:** All 5 existing `checklists/best-practices.md` files.

**Structure:**
- Pillar 1: Security (merge all security items, deduplicate)
- Pillar 2: Scalability (merge all scalability items, deduplicate)
- Pillar 3: Availability (merge all availability items, deduplicate)
- Pillar 4: Cost Optimization (merge all cost items, deduplicate)
- Cross-Cutting: SPOF identification, monthly cost estimation methodology (not hardcoded numbers)

**Deduplication strategy:** Many items appear in 2-3 skills (e.g., IRSA in Security across K8s, TF, GitOps). Keep one instance per unique item. Group by sub-category within each pillar.

### 1.5 Create `checklists/technology-comparisons.md`

Unified technology comparison table.

**Source material:** All 5 existing `checklists/technology-comparisons.md` files.

**Structure:** Single table with columns: Category | Chosen Technology | Formal Definition | Alternatives | Why This Choice

**Categories:**
- IaC (Terraform — from TF skill)
- Container Orchestration (EKS — from K8s skill)
- CNI (VPC-CNI — from K8s skill)
- GitOps (ArgoCD — from GitOps skill)
- Helm (from K8s skill)
- CI/CD (Jenkins — from TF/K8s skills + expand)
- Edge/CDN (CloudFront — from K8s skill)
- DNS (Route 53 + ExternalDNS — from K8s skill)
- Certificates (cert-manager — from K8s skill)
- Networking (NAT Gateway — from K8s skill)
- Secrets (Secrets Manager + ESO — from K8s/GitOps skills)
- Identity (IRSA — from K8s skill)
- Container Registry (ECR — from K8s skill)
- Container Runtime (Docker multi-stage — from K8s skill)
- Monitoring (Prometheus — NEW, from Obs skill's gaps)
- Visualization (Grafana — NEW)
- Log Collection (Fluent-bit — NEW)
- Log Storage/Search (ElasticSearch + Kibana — NEW)
- Docker Compose (NEW — local dev orchestration)

**Deduplication:** Many entries appear in 3-4 skills. Keep the best version of each (longest formal definition, most alternatives listed).

### 1.6 Create `example-transcript.md`

New example showing the combined skill flow.

**Source material:** Composite of all 5 existing example-transcript.md files, rewritten.

**Structure:**
1. Repo Map output (unified table showing all layers)
2. Topic & Scope selection dialogue (NEW — shows user picking "Full Stack, Standard")
3. Question examples (5 exchanges, one from each of 5 different topics):
   - Terraform technology comparison question (score ~70, Proficient)
   - Kubernetes failure-mode question (score ~55, Proficient low-end)
   - GitOps tradeoff question (score ~80, Expert)
   - AWS debugging question (score ~35, Competent — shows a WEAK answer)
   - Observability best-practice question (score ~60, Proficient)
4. Abbreviated debrief showing:
   - Score Summary with multi-topic rows
   - Pillar Coverage
   - Per-topic readiness (only for selected topics)
   - Drill plan excerpt

**Key improvement:** Include a WEAK answer example (score 35) to demonstrate how the skill handles gaps — this was missing from all existing transcripts.

### 1.7 Create `agents/openai.yaml`

Simple agent interface.

```yaml
display_name: DevOps Interviewer
short_description: >-
  Structured DevOps mock interview across Kubernetes, Terraform, GitOps,
  Observability, AWS, and CI/CD with scoring and debrief.
default_prompt: >-
  Use $devops-interviewer to interview me using this repository.
```

---

## Phase 2: Topic Modules

Phase 2 creates the 6 topic files in `topics/`. Each file is self-contained: it includes both the **question templates** and the **evaluation checklist** for that topic, plus a short **enhancement instructions** section that tells the interviewer how to apply topic-specific scoring depth.

### Topic File Structure (common to all)

Each `topics/<topic>.md` file follows this structure:

```markdown
# <Topic> — Interview Module

## Enhancement Instructions
(2-5 bullet points telling the interviewer what to emphasize when scoring this topic)

## Evaluation Checklist
(The merged checklist items that the interviewer scores against)

## Question Templates
### What Breaks If...
### How Would You Debug...
### Tradeoff
### Technology Comparison
### Best Practice Pillar
### Bonus Pool
```

### 2.1 Create `topics/kubernetes.md`

**Source material:**
- `devops-kubernetes-interviewer/question-templates.md` → Rounds 2 (Kubernetes) and 3 (Helm)
- `devops-kubernetes-interviewer/checklists/kubernetes.md` (full)
- Enhancement instructions from K8s skill's "Kubernetes Round Enhancement" section
- Also pull Kubernetes questions from GitOps skill that are K8s-specific

**Merge strategy:**
- Questions: Take all unique questions from K8s round (both K8s and GitOps skills). Deduplicate questions that appear in both. Include Helm questions in a separate "Helm" subsection.
- Checklist: Take the K8s skill's `checklists/kubernetes.md` as the base (it's the most comprehensive). It already includes: systematic troubleshooting workflow, 14 common issues, incident response framework, performance troubleshooting, Helm troubleshooting, primitives knowledge, diagnostic commands, scoring application.
- Enhancement: Take from K8s skill's "Kubernetes Round Enhancement" (lines 126-134).

**Additions from review findings:**
- Add StatefulSet troubleshooting questions (2 questions)
- Add Pod Security Admission question (1 question)
- Add HPA troubleshooting question (1 question)
- Add CRD/operator pattern question (1 bonus question)
- Add checklist items for: StatefulSets, Pod Security Admission, HPA issues, init container failures

### 2.2 Create `topics/terraform.md`

**Source material:**
- `devops-terraform-interviewer/question-templates.md` → Round 1 (Terraform)
- `devops-terraform-interviewer/checklists/terraform.md` (full)
- Enhancement instructions from TF skill's SKILL.md (no explicit enhancement section, but implied in the question templates intro)

**Merge strategy:**
- Questions: Take all Terraform questions from TF skill. Also include Terraform questions that appear in other skills' Round 5 (AWS/IaC rounds) if unique.
- Checklist: Take the TF skill's `checklists/terraform.md` as the base.

**Additions from review findings:**
- Add Terraform workspaces question (1 question)
- Add Terraform testing question (1 question — Terratest, terraform-compliance, OPA)
- Add moved blocks workflow question (1 question)
- Add dynamic blocks question (1 bonus question)
- Add checklist items for: workspaces, testing, moved blocks workflow, CI integration (Infracost in PR comments)

### 2.3 Create `topics/gitops.md`

**Source material:**
- `devops-gitops-interviewer/question-templates.md` → Rounds 1 (ArgoCD/GitOps) and 4 (CI/CD promotion) — only the GitOps-relevant promotion questions
- `devops-gitops-interviewer/checklists/gitops.md` (full)
- Enhancement instructions from GitOps skill's "GitOps Round Enhancement"
- Also include Helm-ArgoCD integration questions (from K8s/GitOps Helm rounds)

**Merge strategy:**
- Questions: Take all ArgoCD/GitOps questions from GitOps skill's Round 1. Include Helm-ArgoCD questions from Round 3. Include CI/CD promotion questions only where they are GitOps-specific (push-vs-pull, Git-based promotion). Pure CI/CD questions go to `topics/cicd.md`.
- Checklist: Take the GitOps skill's `checklists/gitops.md` as the base (it's comprehensive: OpenGitOps principles, ArgoCD core mechanics, repo structure, drift troubleshooting, multi-cluster, secrets, security, reliability, delivery safety, diagnostic commands, ArgoCD vs Flux, troubleshooting depth, secrets comparison).

**Additions from review findings:**
- Add Argo Rollouts questions (2 questions — canary, metric-based rollback)
- Add Argo Notifications question (1 question)
- Add ArgoCD Image Updater question (1 bonus question)
- Add ArgoCD Projects/RBAC question (1 question)
- Add checklist sections for: Argo Rollouts, Argo Notifications, ArgoCD Projects depth, Image Updater

### 2.4 Create `topics/observability.md`

**Source material:**
- `devops-observability-interviewer/question-templates.md` → All 7 rounds
- `devops-observability-interviewer/checklists/monitoring.md` (full)
- `devops-observability-interviewer/checklists/logging-efk.md` (full)
- Enhancement instructions from Obs skill's "Monitoring Round Enhancement" and "Logging Round Enhancement"

**Merge strategy:**
- Questions: Take all questions from Obs skill. Also pull in observability/reliability questions from other skills' Round 6 (Reliability/Observability) that are unique.
- Checklist: Merge both `monitoring.md` and `logging-efk.md` into one combined checklist within this file. Sections: Observability Foundations → Metric Design → Prometheus → Grafana → Alert Quality → SLO/Error Budget → EFK Logging → Incident Response → Diagnostic Commands → Scoring.

**Additions from review findings:**
- Add AWS-native observability questions (2 questions — CloudWatch integration, Container Insights)
- Add long-term storage question (1 question — Thanos/Cortex/VictoriaMetrics)
- Add checklist items for: CloudWatch integration, long-term metrics storage, multi-cluster federation

### 2.5 Create `topics/aws.md`

**Source material:**
- `devops-aws-interviewer/question-templates.md` → All 7 rounds
- `devops-aws-interviewer/checklists/aws.md` (full)
- Enhancement instructions from AWS skill's "AWS Round Enhancement" and "FinOps Enhancement"
- Also pull AWS-specific questions from other skills' AWS rounds that are unique

**Merge strategy:**
- Questions: Take all questions from AWS skill. Deduplicate against questions already in other topic files (e.g., IRSA questions that appear in both K8s and AWS → keep the AWS-architecture-focused version here, the K8s-operations-focused version in kubernetes.md).
- Checklist: Take AWS skill's `checklists/aws.md` as the base.

**Additions from review findings:**
- Add EKS deep-dive questions (3 questions — control/data plane, add-ons, node groups vs Fargate)
- Add ECR questions (2 questions — image scanning, lifecycle policies)
- Add EBS CSI driver question (1 question)
- Add ALB Ingress Controller vs NLB question (1 question)
- Add AWS Systems Manager question (1 bonus question)
- Add checklist items for: EKS operations, ECR management, EBS CSI, ALB/NLB, Systems Manager

### 2.6 Create `topics/cicd.md` (NEW)

**Source material:** This is a NEW topic module. No existing skill covers CI/CD as a primary topic.

Partial source:
- CI/CD questions scattered across other skills' Round 4/6 (GitOps skill's CI/CD round, K8s skill's reliability round Jenkins questions)
- Jenkins-specific details from existing repo-intake.md files
- Docker questions from existing K8s skill's question templates

**Structure:**

```markdown
# CI/CD — Interview Module

## Enhancement Instructions
- Evaluate pipeline design maturity (stages, ordering, failure handling)
- Probe credential management (Jenkins credentials vs K8s secrets vs Secrets Manager)
- Probe artifact lifecycle (build → tag → push → scan → deploy)
- Score higher when candidate connects CI to CD boundary (push-based CI → pull-based GitOps)
- Evaluate Docker build quality (multi-stage, layer caching, non-root, security scanning)

## Evaluation Checklist

### Jenkins Pipeline Design
- [ ] Understands declarative vs scripted pipeline syntax
- [ ] Can explain stage ordering: checkout → build → test → security scan → push → deploy/trigger
- [ ] Understands shared libraries and pipeline reuse
- [ ] Can explain agent architecture (controller vs agents, ephemeral agents)
- [ ] Knows credential management patterns (Jenkins credentials store, withCredentials binding)
- [ ] Can explain when to use pipeline vs freestyle vs multibranch
- [ ] Understands Jenkinsfile triggers (SCM polling, webhooks, cron)

### Docker Build Quality
- [ ] Understands multi-stage builds and why they reduce image size
- [ ] Can explain Docker layer caching and how to optimize for CI
- [ ] Knows non-root user pattern (USER directive, numeric UID)
- [ ] Can explain .dockerignore purpose and common entries
- [ ] Understands image tagging strategy (commit SHA, semantic versioning, never "latest" in prod)
- [ ] Knows Docker Compose purpose (local dev, integration testing)
- [ ] Can explain healthcheck directive in Dockerfile vs Kubernetes probes

### Pipeline Security
- [ ] Secrets never hardcoded in Jenkinsfile or Dockerfile
- [ ] Understands image scanning integration (Trivy, Snyk, ECR scan-on-push)
- [ ] Can explain SBOM generation and software supply chain security
- [ ] Knows least-privilege for CI service accounts (Jenkins IAM role scoped to ECR push + S3)
- [ ] Can explain credential rotation impact on CI pipelines
- [ ] Understands pipeline audit trail (build logs, artifact provenance)

### Artifact Management
- [ ] Understands ECR lifecycle policies (keep N images, expire untagged)
- [ ] Can explain image immutability (once pushed, never overwrite same tag)
- [ ] Knows registry authentication patterns (ECR login, docker-credential-ecr-login)
- [ ] Can explain artifact promotion (dev → staging → prod with same image, different config)

### CI/CD Integration with GitOps
- [ ] Understands CI → CD boundary (CI pushes image, ArgoCD detects and deploys)
- [ ] Can explain why CI should NOT run `kubectl apply` directly
- [ ] Knows image tag update patterns (CI updates values.yaml/kustomization → Git commit → ArgoCD sync)
- [ ] Can explain rollback in CI+GitOps model (Git revert, ArgoCD rollback, or both)

### Docker Compose for Development
- [ ] Understands service definitions, depends_on ordering, healthchecks
- [ ] Can explain volumes and bind mounts for local development
- [ ] Knows network modes and service discovery in Compose
- [ ] Can compare Docker Compose vs Kind/Minikube/Tilt for local K8s development

### Diagnostic Commands
```bash
# Jenkins
curl -s http://jenkins:8080/api/json  # Jenkins status
# Docker
docker build --target <stage> .       # Build specific stage
docker history <image>                # Layer inspection
docker inspect <container>            # Container details
docker-compose config                 # Validate compose file
# ECR
aws ecr describe-images --repository-name <repo>
aws ecr get-lifecycle-policy --repository-name <repo>
```

## Question Templates

### What Breaks If...
1. "Your Jenkinsfile at `{path}` uses `withCredentials` to bind ECR credentials. The IAM role
   attached to the Jenkins EC2 instance loses ECR push permissions. What fails, what still works,
   and how do you diagnose it?"
2. "Your Docker multi-stage build at `{path}` uses `COPY --from=builder`. The builder stage fails
   silently (exits 0 but produces no artifact). What does the final image contain and how do you
   catch this in CI?"
3. "Someone pushes a Docker image tagged `latest` to ECR and ArgoCD's Helm values reference
   `latest`. What happens on the next ArgoCD sync? Why is this dangerous?"
4. (generic) "Your CI pipeline succeeds but the deployed application is broken. What gap in your
   testing strategy allowed this?"

### How Would You Debug...
5. "Jenkins builds are failing with 'no space left on device'. The Jenkins agent is an EC2 instance.
   Walk me through your diagnosis."
6. "A Docker image that builds locally fails in Jenkins CI with a dependency resolution error.
   What environment differences do you check?"
7. "ECR image pulls from EKS are intermittently failing with `unauthorized`. What do you check?"
8. (generic) "Your CI pipeline takes 25 minutes. Walk me through how you would optimize it."

### Tradeoff
9. "Jenkins vs GitHub Actions vs GitLab CI — you chose Jenkins. What are the tradeoffs? When
   would you switch?"
10. "Multi-stage Docker builds add complexity. When is a single-stage build acceptable?"
11. "Your CI pipeline pushes images to ECR but does NOT deploy to the cluster. ArgoCD handles
    deployment. Why this separation? What would change if you used Jenkins to deploy directly?"

### Technology Comparison
12. "[DEFINE] What is Jenkins? Why Jenkins instead of GitHub Actions, GitLab CI, or CircleCI?
    What specific advantage does Jenkins give you for this project?"
13. "[DEFINE] What is a Docker multi-stage build? Why multi-stage instead of a single Dockerfile
    with all dependencies? What problem does it solve for image size and security?"
14. "[DEFINE] What is ECR? Why ECR instead of Docker Hub, GitHub Container Registry, or Harbor?
    What AWS-native integrations does ECR provide?"

### Best Practice Pillar
15. **(Security)** "Map the credential flow in your CI pipeline: Jenkins → ECR (push), Jenkins →
    GitLab (clone), Jenkins → S3 (artifacts). For each, what authentication mechanism is used
    and what is the blast radius if Jenkins is compromised?"
16. **(Cost)** "Your Jenkins agent runs 24/7 on a t3a.medium EC2 instance. Builds happen ~10 times
    per day. What is the cost? What alternatives exist (spot, ephemeral agents, managed CI)?"
17. **(Availability)** "Jenkins is a single point of failure in your pipeline. If Jenkins goes down,
    can you still deploy? What is your Jenkins disaster recovery plan?"

### Bonus Pool
18. "What is the difference between `docker build --cache-from` and BuildKit cache mounts?
    When would you use each in CI?"
19. "Your Jenkinsfile has 8 stages. Which stages can run in parallel? What is the syntax?"
20. "How would you implement a canary deployment purely from the CI/CD perspective, before ArgoCD
    detects the change?"
```

---

## File-by-File Summary

### Phase 1 Files

| File | Action | Primary Source | Size Estimate |
|------|--------|---------------|---------------|
| `SKILL.md` | New (merge 5) | All 5 SKILL.md | ~280 lines |
| `repo-intake.md` | New (merge 5) | All 5 repo-intake.md | ~180 lines |
| `rubrics.md` | New (merge 5) | All 5 rubrics.md | ~200 lines |
| `example-transcript.md` | New (rewrite) | All 5 examples | ~200 lines |
| `checklists/best-practices.md` | New (merge 5) | All 5 best-practices.md | ~250 lines |
| `checklists/technology-comparisons.md` | New (merge 5) | All 5 tech-comparisons.md | ~180 lines |
| `agents/openai.yaml` | New | Existing agents files | ~8 lines |

### Phase 2 Files

| File | Action | Primary Source | Size Estimate |
|------|--------|---------------|---------------|
| `topics/kubernetes.md` | New (merge) | K8s questions + K8s checklist | ~450 lines |
| `topics/terraform.md` | New (merge) | TF questions + TF checklist | ~400 lines |
| `topics/gitops.md` | New (merge) | GitOps questions + GitOps checklist | ~450 lines |
| `topics/observability.md` | New (merge) | Obs questions + monitoring + logging checklists | ~500 lines |
| `topics/aws.md` | New (merge) | AWS questions + AWS checklist | ~450 lines |
| `topics/cicd.md` | New (original) | Scattered CI/CD content + new | ~350 lines |

**Total estimated size:** ~3,900 lines across 13 files (vs ~8,600 lines across 45+ files in the 5 separate skills).

---

## Decisions Log

| Decision | Rationale |
|----------|-----------|
| Helm is NOT a standalone topic | Helm always appears in context of K8s or GitOps. Splitting it would create tiny topic files with 3-4 questions each. |
| Security is cross-cutting, not a topic | Every topic has security questions. Including 1-2 security questions regardless of topic selection ensures coverage. |
| Topic files merge questions + checklist | Keeps all evaluation material for a topic in one file. Reduces file count and cross-referencing. |
| CI/CD is a NEW topic | Addresses the biggest gap found in the review (Jenkins + Docker had zero dedicated coverage). |
| Tool safety uses principle, not exhaustive list | "No mutating operations" is clearer and more maintainable than listing 40+ individual commands. |
| No hardcoded cost estimates | Dollar amounts go stale. Teach estimation methodology instead. |
| Quick mode allows 3 of 5 mandatory categories | With only 2-3 questions per topic, requiring all 5 categories is impossible. Quick mode relaxes this. |
| Keep individual skills as-is (don't delete) | The combined skill should be validated before retiring the originals. |
