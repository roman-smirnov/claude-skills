# Implementation Plan: `devops-jenkins-interviewer`

## Goal

Create a new interviewer skill focused on **Jenkins-first DevOps interview depth with AWS integration** that:

- scans the repository and builds a Jenkins/AWS-aware repo map
- runs a structured, repo-aware mock interview loop
- evaluates Jenkins pipeline design, troubleshooting, security, reliability, and cost tradeoffs
- scores every answer and ends with a debrief + targeted drill plan

The skill should match the existing interviewer contract used by:

- `devops-kubernetes-interviewer/`
- `devops-gitops-interviewer/`
- `devops-terraform-interviewer/`
- `devops-aws-interviewer/`

---

## What Was Reviewed

### Existing interviewer skill contract (reused)

Across `devops-kubernetes-interviewer`, `devops-gitops-interviewer`, `devops-terraform-interviewer`, `devops-aws-interviewer`, and `devops-observability-interviewer`, the shared contract is:

1. Phase 1: repo intake and repo map
2. Phase 2: mode selection
3. Phase 3: interview loop (ask -> wait -> follow-ups -> score -> model answer)
4. Phase 4: debrief (summary tables, strengths/gaps, drill plan)

Common reusable assets/patterns:

- `SKILL.md` with strict behavioral and safety rules
- `repo-intake.md` with domain-specific scan workflow
- `question-templates.md` with mandatory categories per round
- `rubrics.md` with universal scale + domain extensions
- `example-transcript.md` showing expected interview behavior
- `checklists/` as scoring source of truth
- `agents/openai.yaml` metadata pattern

### `ci-cd-jenkins` and `ci-cd` reviewed for reuse

Primary reusable Jenkins/AWS inputs:

- `ci-cd-jenkins/references/jenkins.md`
  - controller vs agent model
  - declarative vs scripted pipeline tradeoffs
  - credentials + `withCredentials`
  - JCasC and plugin hygiene
  - common Jenkins failure patterns
- `ci-cd-jenkins/assets/templates/jenkins/*.Jenkinsfile`
  - representative stage design (checkout/test/build/push)
  - build hygiene (`timestamps`, `disableConcurrentBuilds`, `buildDiscarder`)
  - credentials binding patterns and artifact/archive/stash patterns
- `ci-cd-jenkins/scripts/pipeline_analyzer.py`
  - Jenkins heuristics for interview expectations: `parallel`, explicit `agent`, `timeout`, `withCredentials`, `stash/unstash`, `archiveArtifacts`, `disableConcurrentBuilds`
- `ci-cd-jenkins/references/security.md`
  - OIDC patterns, secrets handling, supply chain controls
- `ci-cd-jenkins/references/optimization.md`
  - caching, parallelization, timeout/concurrency and performance heuristics
- `ci-cd-jenkins/references/troubleshooting.md`
  - pipeline failure diagnosis patterns
- `ci-cd-jenkins/references/devsecops.md`
  - SAST/SCA/container scanning and quality/security gates
- `ci-cd/scripts/ci_health.py`
  - reusable CI health framing for reliability discussions

Additional implementation note discovered during review:

- `ci-cd-jenkins/SKILL.md` frontmatter currently has `name: ci-cd`; do not propagate that mismatch into the new interviewer skill.

---

## Planned Skill Package

Create folder:

- `devops-jenkins-interviewer/`

Planned files:

- `devops-jenkins-interviewer/SKILL.md`
- `devops-jenkins-interviewer/repo-intake.md`
- `devops-jenkins-interviewer/question-templates.md`
- `devops-jenkins-interviewer/rubrics.md`
- `devops-jenkins-interviewer/example-transcript.md`
- `devops-jenkins-interviewer/checklists/jenkins.md` (new domain checklist)
- `devops-jenkins-interviewer/checklists/aws.md` (AWS integration depth)
- `devops-jenkins-interviewer/checklists/best-practices.md`
- `devops-jenkins-interviewer/checklists/technology-comparisons.md`
- `devops-jenkins-interviewer/agents/openai.yaml`

---

## Implementation Phases

### Phase 1: Scaffold the new interviewer skill

1. Copy baseline structure from `devops-aws-interviewer/` (best AWS-aligned baseline).
2. Keep all core interviewer assets.
3. Add/replace domain checklist with `checklists/jenkins.md`.
4. Keep AWS checklist to preserve cloud-depth scoring (`checklists/aws.md`).

### Phase 2: Author `SKILL.md` for Jenkins-first + AWS interview depth

1. Set frontmatter:
   - `name: devops-jenkins-interviewer`
   - description with explicit Jenkins+AWS interview triggers
2. Keep standard interview mechanics and safety model.
3. Define Jenkins-first framing:
   - CI/CD architecture reasoning
   - controller/agent operational model
   - credentials/security and supply chain safety
   - AWS deployment integration and operational readiness
4. Define mode menu (example):
   - Full loop
   - Jenkins-only deep dive
   - Jenkins + AWS integration drill
   - Security + incident drill
   - Mixed rapid-fire (10 questions)
5. Define round order with Jenkins as primary and AWS as integrated domain.

### Phase 3: Build `repo-intake.md` for Jenkins/AWS artifact discovery

Capture and classify:

1. Jenkins pipeline artifacts:
   - `Jenkinsfile*`, shared libraries (`vars/`, `src/`, `resources/`), folder/job DSL, JCasC yaml, plugin manifests, seed jobs
2. Pipeline design patterns:
   - stages, parallel branches, caching strategy, artifact flow, test layers, deploy handoff
3. Credentials and security:
   - use of credentials binding, secret sources, OIDC/federation patterns, unsafe secret usage signals
4. AWS integration artifacts:
   - IAM roles/policies/trust, ECR/EKS/ECS/Lambda deployment targets, Route53/CloudFront references, S3 artifact/state usage
5. Operational and observability signals:
   - retry/timeout controls, logging/notifications, runbooks, incident docs, CI health trends
6. Cost/performance signals:
   - controller sizing, agent model (static/dynamic), queueing bottlenecks, compute waste patterns

### Phase 4: Create scoring checklists

1. `checklists/jenkins.md` (new)
   - controller/agent architecture correctness
   - declarative pipeline quality and stage design
   - credentials/security posture (`withCredentials`, least privilege)
   - reliability/troubleshooting methodology
   - performance and scalability controls
   - plugin/JCasC governance and upgrade safety
2. `checklists/aws.md`
   - reuse AWS scoring expectations for IAM/network/runtime/data/cost
   - focus on Jenkins-to-AWS deployment identity path and blast radius
3. `checklists/best-practices.md`
   - enforce four-pillar interview coverage
4. `checklists/technology-comparisons.md`
   - Jenkins vs GitHub Actions/GitLab CI/Tekton
   - push-based CD vs GitOps handoff
   - credential strategies, runner/agent strategies, artifact and deployment model tradeoffs

### Phase 5: Build `question-templates.md`

For each round, enforce mandatory categories:

1. What breaks if...
2. How would you debug...
3. Tradeoff
4. Technology comparison (formal definitions required)
5. Best practice pillar

Must include Jenkins-centric scenario inventory:

- agent offline/label mismatch/queue starvation
- workspace contention and concurrency failures
- missing timeout/retry behavior and hung builds
- secret leakage and credential-scope misconfiguration
- plugin drift/CVE/compatibility failures
- broken promotion flow (build passes, deploy fails)
- Jenkins-to-AWS access failures (`AccessDenied`, OIDC/trust issues)
- ECR/EKS deployment path and rollback safety

### Phase 6: Author `rubrics.md`

1. Keep universal scoring + technology justification scoring.
2. Add Jenkins-focused domain sections, for example:
   - Jenkins architecture and pipeline design
   - Jenkins security and identity boundaries
   - Jenkins troubleshooting and incident response
   - AWS integration and cloud operations alignment
   - CI/CD performance and cost governance
3. Keep rubric language aligned to existing interviewer score labels.

### Phase 7: Create `example-transcript.md`

Transcript should include:

1. repo map excerpt with Jenkins/AWS artifact availability classification
2. one scored Jenkins architecture/tradeoff exchange
3. one scored Jenkins troubleshooting exchange
4. one scored Jenkins+AWS identity/deployment exchange
5. debrief excerpt with drill plan

### Phase 8: Final metadata

Create `agents/openai.yaml` with:

- display name: `DevOps Jenkins Interviewer`
- short description: repo-aware Jenkins + AWS DevOps mock interview
- default prompt using `$devops-jenkins-interviewer`

### Phase 9: Validate and iterate

1. Run structural validation:
   - `python3 /Users/romansmirnov/.codex/skills/.system/skill-creator/scripts/quick_validate.py devops-jenkins-interviewer`
2. Run content sanity checks for phases/rounds/scoring blocks.
3. Dry-run against:
   - Jenkins-heavy AWS repo
   - AWS-heavy repo with limited Jenkins artifacts
   - sparse repo (generic fallback behavior)

---

## Non-Goals

- No mutating or deployment-executing behavior in interview mode
- No runtime dependency on `ci-cd`/`ci-cd-jenkins` scripts as required tooling (reuse patterns, not coupling)
- No changes to existing interviewer skills in this step

---

## Exit Criteria

- Skill triggers correctly for Jenkins interview-practice requests.
- Repo map reliably identifies Jenkins and AWS artifacts and references real paths.
- Questions are repo-grounded when artifacts exist; generic only when necessary.
- Every question yields score + tech justification + model answer.
- Debrief includes score summary, pillar coverage, strengths/gaps, and actionable drills.
- Safety remains strictly read-only; no mutating Jenkins/AWS/Kubernetes/terraform commands.
