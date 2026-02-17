# devops-interviewer — Validation Checklist

**Date:** 2026-02-16

Use this checklist to validate each file during implementation. Cross off items as you complete them.

---

## Phase 1: Base Framework ✅

### 1.1 SKILL.md ✅

#### Structure
- [x] Frontmatter has `name: devops-interviewer` and accurate `description`
- [x] Has all major sections: Purpose, Invocation, Framing, Phase 1-4, Tool Safety, Behavioral Rules
- [x] Purpose section mentions all 6 topics (K8s, TF, GitOps, Obs, AWS, CI/CD)
- [x] Invocation section shows both explicit (`$devops-interviewer`) and natural language triggers

#### Phase 1: Repo Intake
- [x] References `repo-intake.md` correctly
- [x] Specifies Repo Map output format (table with Layer / Artifacts Found / Key Paths)
- [x] Requires per-topic availability classification (Available / Limited / Generic / Skipped)

#### Phase 2: Topic & Scope Selection
- [x] Presents topic menu with all 6 topics as multi-select checkboxes
- [x] Includes "Full Stack (all topics)" option
- [x] Presents scope menu with 3 levels (Quick/Standard/Deep)
- [x] Quick: 5-8 questions, ~20 min
- [x] Standard: 12-18 questions, ~45 min
- [x] Deep: 25-35 questions, ~90 min
- [x] Includes question distribution guidance (at least 2 per selected topic)
- [x] Explicitly says "Wait for user selection before proceeding"

#### Phase 3: Interview Loop
- [x] Lists all rounds with correct `topics/*.md` references
- [x] Per-Question Flow matches existing pattern (ask → wait → follow-up → wait → score → model answer)
- [x] Score block format matches existing: Rating, Tech Justification, Strengths, Gap, Key point
- [x] Specifies 5 mandatory question categories (failure-mode, debug, tradeoff, tech comparison, best practice pillar)
- [x] Quick mode relaxes to 3 of 5 categories
- [x] States that Security is cross-cutting (always 1-2 questions)
- [x] Repo Grounding Rules present (reference real paths, never fabricate)

#### Phase 4: Debrief
- [x] Score Summary table (per topic, not per round)
- [x] Best Practice Pillar Coverage table
- [x] Dynamic per-topic readiness sections (only for selected topics)
- [x] Technology Justification Readiness section
- [x] Strengths / Gaps sections
- [x] Interview-Ready Talking Points (top 5)
- [x] Targeted Drill Plan table (Priority, Gap, Pillar, Exercise, Resources)

#### Tool Safety
- [x] Allowed section lists read-only operations (unified superset)
- [x] Prohibited section uses principle-based approach ("no mutating operations")
- [x] Covers: Terraform, kubectl, Helm, ArgoCD, AWS CLI, Docker, Git
- [x] States refusal policy for prohibited commands

#### Behavioral Rules
- [x] All 11 rules present (never write files, never answer own question, never skip scoring, stay in character, be concise, rubric-grounded, define terminology, always probe alternatives, pivot to infrastructure, provide model answers, cover all four pillars)
- [x] No rules contradict each other
- [x] No skill-specific rules leaked in (e.g., no "correct anti-GitOps instincts" — that belongs in topic file)

### 1.2 repo-intake.md ✅

- [x] Has all 13 steps (Directory Structure through Sibling Repos)
- [x] Step 1 (Directory Structure) has comprehensive glob patterns (no duplicates)
- [x] Step 2 (Terraform) covers: root modules, child modules, bootstrap, state partitioning, key resources, provider versions, variables/outputs
- [x] Step 3 (Kubernetes) covers: EKS config, IRSA, namespaces, manifests, Kustomize, resource requests/limits, probes, network policies, RBAC, storage
- [x] Step 4 (Helm) covers: Chart.yaml, values layering, templates, ArgoCD-managed releases
- [x] Step 5 (ArgoCD/GitOps) covers: Application/ApplicationSet CRDs, app-of-apps, sync policies, sync waves, repo credentials, Helm/directory sources
- [x] Step 6 (CI/CD) covers: pipeline definition, stages, credential management, deploy strategy, test coverage, artifact management
- [x] Step 7 (Docker) covers: Dockerfiles, docker-compose, image tagging
- [x] Step 8 (AWS) covers: IAM model, VPC/networking, compute, data/storage, edge/DNS, security controls
- [x] Step 9 (Observability) covers: Prometheus, Grafana, alerting rules, log collectors, Elasticsearch, Kibana, SLO docs
- [x] Step 10 (Secrets) covers: Secrets Manager, ESO, sealed secrets, SOPS, credential management
- [x] Step 11 (DNS/CDN) covers: Route 53, CloudFront, ACM, ExternalDNS, ingress controllers
- [x] Step 12 (Scripts/Docs) covers: shell scripts, documentation, diagrams
- [x] Step 13 (Sibling Repos) covers: cross-repo scanning, relationships
- [x] Output format specified: Repo Map table + per-topic availability
- [x] "Never fabricate paths" rule present
- [x] All operations are read-only

### 1.3 rubrics.md ✅

- [x] Universal Scale table present with all 5 levels (No Answer, Awareness, Competent, Proficient, Expert)
- [x] Score ranges: 0, 1-25, 26-50, 51-75, 76-100
- [x] Each level has clear criteria text
- [x] Technology Justification Score section present with own 0-100 scale
- [x] Tech Justification includes scoring signals (formal definition, project-specific, alternatives, pillar tie-in)
- [x] Domain extensions present for all 8 domains:
  - [x] 1. Terraform & IaC
  - [x] 2. Kubernetes
  - [x] 3. Helm
  - [x] 4. ArgoCD / GitOps
  - [x] 5. CI/CD (NEW — Jenkins, Docker, pipeline security)
  - [x] 6. AWS Architecture
  - [x] 7. Observability (Prometheus, EFK, SLO)
  - [x] 8. Security
- [x] Each domain extension has criteria for all 5 score levels (0, 1-25, 26-50, 51-75, 76-100)
- [x] CI/CD domain extension covers: Jenkins pipeline maturity, Docker build quality, pipeline security, artifact management, CI/CD-GitOps integration
- [x] No domain extension references files that don't exist
- [x] Key evaluation signals listed for K8s (systematic troubleshooting, incident response, common issues)

### 1.4 checklists/best-practices.md ✅

- [x] Has all 4 pillars: Security, Scalability, Availability, Cost Optimization
- [x] Security pillar covers: IAM/RBAC, secrets management, network segmentation, container security, supply chain, audit
- [x] Scalability pillar covers: compute, network, storage, application, pipeline, observability stack scaling
- [x] Availability pillar covers: control plane, data plane, networking, application, DR, SPOF identification
- [x] Cost pillar covers: compute, network, storage, DNS, operational costs, optimization opportunities
- [x] No hardcoded dollar amounts (methodology instead)
- [x] Items are deduplicated (no item appears twice)
- [x] Items span all 6 topics (not biased toward one domain)
- [x] Cross-cutting section covers SPOF identification methodology

### 1.5 checklists/technology-comparisons.md ✅

- [x] Single table format: Category | Chosen Technology | Formal Definition | Alternatives | Why This Choice
- [x] Has entries for all 19 technologies listed in the plan (36 entries total)
- [x] Includes NEW entries for: Prometheus, Grafana, Fluent-bit, ElasticSearch+Kibana, Docker Compose
- [x] Each formal definition is 1-2 sentences, precise, interview-ready
- [x] Each entry lists 2-3 alternatives
- [x] "Why This Choice" column explains project-specific reasoning
- [x] No duplicate entries
- [x] Entries cover all 6 topics

### 1.6 example-transcript.md ✅

- [x] Shows Repo Map output (unified table)
- [x] Shows Topic & Scope selection dialogue (user picks specific topics)
- [x] Shows at least 5 question examples from different topics
- [x] Shows at least one WEAK answer (score 26-50 range) — Q4 AWS: 35/100
- [x] Shows at least one STRONG answer (score 76-100 range) — Q3 GitOps: 80/100
- [x] Score blocks match the format from SKILL.md
- [x] Model answers are 3-6 sentences each
- [x] Shows abbreviated debrief with:
  - [x] Multi-topic Score Summary table
  - [x] Pillar Coverage table
  - [x] At least 2 per-topic readiness sections
  - [x] Drill plan excerpt

### 1.7 agents/openai.yaml ✅

- [x] Has `display_name`, `short_description`, `default_prompt`
- [x] `display_name` is "DevOps Interviewer"
- [x] `default_prompt` references `$devops-interviewer`

---

## Phase 2: Topic Modules ✅

### Common Checks (apply to ALL 6 topic files) ✅

For each `topics/<topic>.md`:

- [x] Has three main sections: Enhancement Instructions, Evaluation Checklist, Question Templates
- [x] Enhancement Instructions has 3-5 bullet points specific to this topic
- [x] Evaluation Checklist has checkboxes (- [ ]) for each evaluation item
- [x] Checklist items are actionable and scorable
- [x] Question Templates has all 5 mandatory categories: What Breaks If, How Would You Debug, Tradeoff, Technology Comparison, Best Practice Pillar
- [x] Has a Bonus Pool section
- [x] Each category has at least 3 questions
- [x] Questions use `{placeholder}` syntax for repo-specific values
- [x] At least 1 question per category has a `(generic)` variant
- [x] Technology Comparison questions use `[DEFINE]` marker
- [x] Best Practice Pillar questions are tagged with pillar name
- [x] All 4 pillars are represented across the questions
- [x] Has Diagnostic Commands section with read-only commands
- [x] Has Scoring Application section with coverage-to-score mapping
- [x] No references to files that don't exist in the new structure
- [x] No duplicate questions (same question appearing in two topic files)

### 2.1 topics/kubernetes.md ✅

- [x] Enhancement Instructions cover: systematic troubleshooting workflow, common failure patterns, incident response maturity, performance troubleshooting, Helm troubleshooting
- [x] Checklist includes: 6-phase troubleshooting workflow, 14+ common issue patterns, incident response framework, performance troubleshooting, Helm troubleshooting, primitives knowledge
- [x] Includes NEW items from review: StatefulSets, Pod Security Admission, HPA troubleshooting, init container failures
- [x] Includes Helm questions in a separate subsection
- [x] Questions reference K8s-specific artifacts: EKS node groups, vpc-cni, ebs-csi-driver, IRSA
- [x] At least 18 questions total (including bonus) — **36 questions total**

### 2.2 topics/terraform.md ✅

- [x] Enhancement Instructions cover: state management, module design, lifecycle rules, drift detection, pre-commit pipeline
- [x] Checklist includes: module structure, variable/output standards, naming conventions, state management, security, lifecycle rules, provider management, import/migration, troubleshooting
- [x] Includes NEW items from review: workspaces, testing (Terratest/OPA), moved blocks workflow, dynamic blocks
- [x] Questions reference TF-specific artifacts: backend.tf, modules/, bootstrap/, prevent_destroy, state files
- [x] At least 20 questions total (including bonus) — **25 questions total**

### 2.3 topics/gitops.md ✅

- [x] Enhancement Instructions cover: OpenGitOps principles, reconciliation/drift, architecture tradeoffs, secrets strategy, ArgoCD scoring depth
- [x] Checklist includes: OpenGitOps principles, ArgoCD core mechanics, repo structure/promotion, drift troubleshooting, multi-cluster, secrets, security/governance, reliability/DR, delivery safety, ArgoCD vs Flux comparison, troubleshooting depth, secrets management comparison
- [x] Includes NEW items from review: Argo Rollouts, Argo Notifications, ArgoCD Image Updater, ArgoCD Projects depth
- [x] Questions reference GitOps-specific artifacts: Application CRDs, app-of-apps, sync waves, SSH keys
- [x] At least 18 questions total (including bonus) — **28 questions total**

### 2.4 topics/observability.md ✅

- [x] Enhancement Instructions cover: metric design (Golden Signals, RED/USE), PromQL semantics, cardinality, alert quality, structured logging, EFK pipeline, SLO/burn-rate
- [x] Checklist includes: Observability foundations, metric design, Prometheus collection, Grafana dashboards, alert quality, SLO/error budget, EFK logging pipeline, incident response, diagnostic commands
- [x] Includes NEW items from review: AWS-native observability (CloudWatch), long-term storage (Thanos/Cortex)
- [x] Questions cover all observability tools in the project: Prometheus, Grafana, Fluent-bit, ElasticSearch, Kibana
- [x] At least 22 questions total (including bonus — this topic has the most sub-domains) — **44 questions total**

### 2.5 topics/aws.md ✅

- [x] Enhancement Instructions cover: formal definitions for AWS services, blast radius reasoning, least-privilege depth, cost implications, FinOps maturity
- [x] Checklist includes: Architecture, IAM, networking, compute, data/storage/DR, edge/DNS, security/compliance, observability, cost/governance
- [x] Includes NEW items from review: EKS deep-dive, ECR, EBS CSI, ALB/NLB, Systems Manager
- [x] Questions reference AWS-specific artifacts: VPC, IAM roles, CloudFront, Route 53, NAT Gateway, Secrets Manager
- [x] At least 20 questions total (including bonus) — **37 questions total**

### 2.6 topics/cicd.md (NEW) ✅

- [x] Enhancement Instructions cover: pipeline design maturity, credential management, artifact lifecycle, CI/CD-GitOps boundary, Docker build quality
- [x] Checklist includes: Jenkins pipeline design, Docker build quality, pipeline security, artifact management, CI/CD-GitOps integration, Docker Compose
- [x] ALL content is original or adapted (no wholesale copy from existing skills)
- [x] Questions reference CI/CD-specific artifacts: Jenkinsfile, Dockerfile, docker-compose, ECR
- [x] At least 20 questions total (including bonus) — **33 questions total**
- [x] Covers both Jenkins AND Docker (not just one)
- [x] Includes CI/CD-to-GitOps handoff questions

---

## Cross-File Consistency Checks ✅

After both phases are complete, verify:

### References ✅
- [x] Every `topics/*.md` file referenced in SKILL.md actually exists
- [x] Every checklist file referenced in SKILL.md actually exists
- [x] No file references a path that doesn't exist (e.g., old `checklists/kubernetes.md` path)
- [x] `repo-intake.md` doesn't reference topic files (intake is topic-agnostic)

### Deduplication ✅
- [x] No question appears in more than one topic file
- [x] No checklist item appears in more than one topic file (best-practices items are in checklists/best-practices.md only, not duplicated in topic files)
- [x] Technology comparison entries appear only in `checklists/technology-comparisons.md`, not duplicated in topic files
- [x] Behavioral rules appear only in SKILL.md, not repeated in topic files
- [x] Tool safety appears only in SKILL.md, not repeated in topic files

### Scoring Consistency ✅
- [x] All topic files use the same score block format
- [x] All topic files reference `rubrics.md` for scoring (not inline rubrics)
- [x] Scoring Application sections in topic files use the same coverage-to-score mapping
- [x] The 5 mandatory question categories are named identically across all topic files

### Content Coverage ✅
- [x] Every technology in the student project has at least one question across all topics:
  - [x] Jenkins
  - [x] Docker / Docker Compose
  - [x] Terraform
  - [x] Kubernetes / EKS
  - [x] Helm / Helm Charts
  - [x] ArgoCD
  - [x] Prometheus
  - [x] Fluent-bit
  - [x] ElasticSearch
  - [x] Kibana
  - [x] Grafana
  - [x] AWS (VPC, IAM, S3, ECR, Route 53, CloudFront, NAT Gateway, Secrets Manager)
- [x] Every technology in the project has an entry in `checklists/technology-comparisons.md`
- [x] All 4 pillars (security, scalability, availability, cost) have questions in every topic file
- [x] The combined skill has more questions than any single existing skill (target: 120+ across all topics) — **203 questions total**

### Sizing ✅
- [x] Total combined line count is under 5,500 lines (target from feasibility analysis) — **2,914 lines total**
- [x] No single file exceeds 550 lines — **max is kubernetes.md at 321 lines**
- [x] SKILL.md is under 300 lines — **274 lines**

---

## Functional Testing (Post-Implementation)

These tests verify the skill works correctly when invoked:

### Activation Tests
- [ ] Skill activates when invoked as `$devops-interviewer`
- [ ] Skill activates on natural language: "interview me on DevOps"
- [ ] Repo intake scans all layers and produces Repo Map
- [ ] Topic selection menu appears with all 6 topics
- [ ] Scope selection menu appears with 3 levels
- [ ] "Full Stack" option selects all topics
- [ ] Skill waits for user selection (doesn't proceed automatically)

### Interview Flow Tests
- [ ] Questions reference actual repo paths when artifacts exist
- [ ] Questions fall back to generic when artifacts are missing
- [ ] Each question gets a score block after the answer
- [ ] Model answer appears after each score block
- [ ] Follow-up questions are asked (1-2 per question)
- [ ] Interviewer never answers its own questions
- [ ] Interviewer never skips scoring

### Scope Tests
- [ ] Quick mode: 5-8 questions across selected topics
- [ ] Standard mode: 12-18 questions across selected topics
- [ ] Deep mode: 25-35 questions across selected topics
- [ ] Questions are distributed across selected topics (not all from one topic)
- [ ] At least 2 questions per selected topic

### Debrief Tests
- [ ] Debrief appears after all questions
- [ ] Score Summary has rows only for selected topics
- [ ] Pillar Coverage table is complete
- [ ] Per-topic readiness sections appear only for selected topics
- [ ] Drill plan has actionable items with pillar tags
- [ ] Interview-Ready Talking Points has 5 items
