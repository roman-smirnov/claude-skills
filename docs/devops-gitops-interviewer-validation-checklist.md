# Validation Checklist: `devops-gitops-interviewer`

Use this checklist to validate content quality and behavior before considering the new skill complete.

## 1) Skill Package Structure

- [ ] Folder exists at `devops-gitops-interviewer/`
- [ ] Required file exists: `devops-gitops-interviewer/SKILL.md`
- [ ] Metadata file exists: `devops-gitops-interviewer/agents/openai.yaml`
- [ ] Core interview assets exist:
  - [ ] `devops-gitops-interviewer/repo-intake.md`
  - [ ] `devops-gitops-interviewer/question-templates.md`
  - [ ] `devops-gitops-interviewer/rubrics.md`
  - [ ] `devops-gitops-interviewer/example-transcript.md`
- [ ] Checklist assets exist:
  - [ ] `devops-gitops-interviewer/checklists/gitops.md`
  - [ ] `devops-gitops-interviewer/checklists/kubernetes.md`
  - [ ] `devops-gitops-interviewer/checklists/best-practices.md`
  - [ ] `devops-gitops-interviewer/checklists/technology-comparisons.md`

## 2) Triggering and Metadata Correctness

- [ ] `SKILL.md` frontmatter contains only:
  - [ ] `name: devops-gitops-interviewer`
  - [ ] `description: ...`
- [ ] Description clearly includes:
  - [ ] GitOps/ArgoCD interview intent
  - [ ] Kubernetes interview intent
  - [ ] Repo-aware/mock-interview trigger contexts
- [ ] `agents/openai.yaml` values match the new skill:
  - [ ] display name
  - [ ] short description
  - [ ] default prompt with `$devops-gitops-interviewer`

## 3) Interview Flow Contract (`SKILL.md`)

- [ ] Includes the 4 main phases:
  - [ ] Repo intake
  - [ ] Mode selection
  - [ ] Interview loop
  - [ ] Debrief
- [ ] Includes mode menu with at least:
  - [ ] Full loop
  - [ ] GitOps-only
  - [ ] Kubernetes-only
  - [ ] Mixed rapid-fire
- [ ] Requires explicit user mode selection before questions begin
- [ ] Defines per-question interaction contract:
  - [ ] Ask one question
  - [ ] Wait for user answer
  - [ ] Ask 1-2 follow-ups
  - [ ] Score using rubric
  - [ ] Provide concise model answer
- [ ] Enforces repo-grounding rules (no fabricated paths)
- [ ] Keeps read-only safety and explicit prohibited command list

## 4) GitOps/ArgoCD Depth Coverage

- [ ] Repo intake captures:
  - [ ] Application/ApplicationSet
  - [ ] app-of-apps topology
  - [ ] sync policy (`automated`, `selfHeal`, `prune`)
  - [ ] sync waves/hooks
  - [ ] ArgoCD project/RBAC boundaries
  - [ ] repo credential and secret patterns
- [ ] Question templates include ArgoCD failure/debug/tradeoff scenarios:
  - [ ] OutOfSync and drift analysis
  - [ ] self-heal/prune side effects
  - [ ] app-of-apps vs ApplicationSet
  - [ ] sync ordering dependencies
  - [ ] repo credential rotation failure
  - [ ] scaling concerns (multi-cluster/sharding)
- [ ] Rubric has explicit 0-4 ArgoCD/GitOps criteria
- [ ] `checklists/gitops.md` exists and is used as scoring source of truth

## 5) Kubernetes Depth Coverage

- [ ] Kubernetes section includes troubleshooting expectations (not only theory)
- [ ] Questions include common production failures (CrashLoopBackOff, Pending, networking, storage)
- [ ] Rubric distinguishes superficial vs systematic troubleshooting
- [ ] `checklists/kubernetes.md` reflects structured diagnosis and remediation sequencing

## 6) Cross-Cutting Interview Quality

- [ ] Every round enforces all required question types:
  - [ ] What breaks if
  - [ ] How would you debug
  - [ ] Tradeoff
  - [ ] Technology comparison
  - [ ] Best-practice pillar
- [ ] Four pillars are tracked across interview:
  - [ ] Security
  - [ ] Scalability
  - [ ] Availability
  - [ ] Cost optimization
- [ ] Debrief includes:
  - [ ] score summary table
  - [ ] pillar coverage table
  - [ ] strengths and gaps
  - [ ] interview-ready talking points
  - [ ] targeted drill plan with actionable exercises

## 7) Transcript Quality Gate

- [ ] `example-transcript.md` shows realistic mode selection and flow
- [ ] Contains at least:
  - [ ] one GitOps/ArgoCD scored exchange
  - [ ] one Kubernetes scored exchange
  - [ ] one debrief excerpt
- [ ] Score blocks align with rubric language (0-4 labels)

## 8) Safety and Non-Destructive Behavior

- [ ] Allowed commands are read-only/analysis-only
- [ ] Mutating operations are explicitly prohibited (`apply`, `delete`, `sync`, destructive git/docker/aws actions)
- [ ] Skill explicitly refuses prohibited command requests during interview mode

## 9) Manual Scenario Tests

- [ ] Scenario A: Repo with rich ArgoCD + Kubernetes artifacts
  - [ ] Repo map references real paths
  - [ ] GitOps-only mode produces artifact-specific questions
- [ ] Scenario B: Repo with Kubernetes artifacts but little/no ArgoCD
  - [ ] ArgoCD round marked limited/generic
  - [ ] No fabricated ArgoCD paths
- [ ] Scenario C: Sparse repo with minimal infra
  - [ ] Skill falls back to rigorous generic questions
  - [ ] Debrief still produced in full format

## 10) Final Validation Commands

- [ ] Run quick frontmatter/package validation (if available):
  - [ ] `python3 /Users/romansmirnov/.codex/skills/.system/skill-creator/scripts/quick_validate.py devops-gitops-interviewer`
- [ ] Run content sanity checks:
  - [ ] `rg -n "## Phase|### Rounds|### Per-Question Flow|## Tool Safety|## Behavioral Rules" devops-gitops-interviewer/SKILL.md`
  - [ ] `rg -n "^## Round|### What Breaks If|### How Would You Debug|### Tradeoff|### Technology Comparison|### Best Practice Pillar" devops-gitops-interviewer/question-templates.md`
  - [ ] `rg -n "^### 1\\. ArgoCD / GitOps|^### 2\\. Kubernetes Troubleshooting and Operations" devops-gitops-interviewer/rubrics.md`

## 11) Reuse Congruence with Implementation Plan

- [ ] `checklists/gitops.md` clearly reflects source inputs from:
  - [ ] `gitops-workflows/references/best_practices.md`
  - [ ] `gitops-workflows/references/troubleshooting.md`
  - [ ] `gitops-workflows/scripts/check_argocd_health.py` logic
  - [ ] `gitops-workflows/scripts/validate_gitops_repo.py` logic
- [ ] `question-templates.md` includes scenarios derived from:
  - [ ] `gitops-workflows/references/troubleshooting.md` (OutOfSync/drift, invalid resource sync failure)
  - [ ] `gitops-workflows/references/argocd_vs_flux.md` (ArgoCD vs Flux tradeoff)
  - [ ] `gitops-workflows/references/multi_cluster.md` (ApplicationSet scale patterns)
  - [ ] `gitops-workflows/references/secret_management.md` (SOPS vs ESO vs Sealed Secrets)
- [ ] `repo-intake.md` includes artifact-discovery heuristics derived from:
  - [ ] `gitops-workflows/references/repo_patterns.md`
  - [ ] `gitops-workflows/references/best_practices.md`
  - [ ] `gitops-workflows/scripts/validate_gitops_repo.py` checks
- [ ] `example-transcript.md` includes at least one ArgoCD exchange grounded in `OutOfSync`/drift from `gitops-workflows/references/troubleshooting.md`
- [ ] Safety congruence with plan:
  - [ ] No mutating workflow copied from `gitops-workflows/SKILL.md` into interviewer behavior
  - [ ] Read-only model remains enforced
- [ ] Content hygiene checks:
  - [ ] No typo copy from `gitops-workflows/references/oci_artifacts.md` (`kind: OC IRepository`)
  - [ ] Version-specific claims from `gitops-workflows` are used as context and not hard-coded as mandatory behavior
- [ ] Optional grep checks for congruence:
  - [ ] `rg -n "OutOfSync|drift|ApplicationSet|app-of-apps|SOPS|External Secrets|Sealed Secrets" devops-gitops-interviewer/question-templates.md devops-gitops-interviewer/checklists/gitops.md devops-gitops-interviewer/example-transcript.md`
  - [ ] `rg -n "OC IRepository" devops-gitops-interviewer`

## Sign-Off

- [ ] All checklist items passed
- [ ] Remaining gaps documented
- [ ] Plan approved before implementation starts
