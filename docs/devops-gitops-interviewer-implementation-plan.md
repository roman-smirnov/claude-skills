# Implementation Plan: `devops-gitops-interviewer`

## Goal

Create a new interviewer skill focused on **GitOps depth (ArgoCD + Kubernetes)** that preserves the interview mechanics and quality bar used by:

- `devops-kubernetes-interviewer`
- `devops-terraform-interviewer`

The skill must thoroughly scan a project, ask repo-aware questions, score answers, and provide a structured debrief.

## Scope

- Create a new skill folder: `devops-gitops-interviewer/`
- Reuse existing interviewer structure (phases, scoring loop, debrief, safety rules)
- Shift domain depth toward:
  - ArgoCD/GitOps (primary)
  - Kubernetes operations/troubleshooting (secondary but deep)
- Keep cross-domain coverage (Helm, Terraform, AWS, reliability, security) for full-loop interviews

## Non-Goals

- No runtime command integrations beyond existing read-only workflow
- No changes to existing `devops-kubernetes-interviewer` or `devops-terraform-interviewer`
- No attempt to merge/interlink skills at runtime; this is a standalone skill package

## Source Baseline

Use `devops-terraform-interviewer/` as the structural template because it already includes:

- interview phases
- repo intake
- mode selection
- round-based question flow
- scoring + debrief
- safety/behavioral rules

Inject depth from:

- `gitops-workflows/` for ArgoCD/GitOps standards
- `k8s-troubleshooter/` for Kubernetes troubleshooting standards

## Reusable Parts from `gitops-workflows`

Reuse these files as source material when writing the new interviewer skill artifacts.

### Primary references (directly reusable as content input)

- `gitops-workflows/references/troubleshooting.md`
  - Reuse for ArgoCD failure/debug question scenarios (`OutOfSync`, invalid resources, tracking migration).
  - Reuse command sequences as expected diagnostic flow in `checklists/gitops.md`.
- `gitops-workflows/references/best_practices.md`
  - Reuse OpenGitOps principles and ArgoCD 3.x operational practices for rubric/checklist criteria.
  - Reuse anti-patterns (`manual kubectl apply`, `latest` tags, secrets in Git) for failure-mode questions.
- `gitops-workflows/references/repo_patterns.md`
  - Reuse app-of-apps, monorepo/polyrepo, and promotion strategies for architecture tradeoff questions.
  - Reuse structure expectations in `repo-intake.md` artifact discovery.
- `gitops-workflows/references/multi_cluster.md`
  - Reuse ApplicationSet generator concepts and multi-cluster tradeoffs for advanced GitOps round questions.
- `gitops-workflows/references/secret_management.md`
  - Reuse SOPS vs Sealed Secrets vs ESO comparison for security pillar questions and rubric depth.
- `gitops-workflows/references/argocd_vs_flux.md`
  - Reuse comparison matrix for “Technology Comparison” prompts and Tech Justification scoring cues.
- `gitops-workflows/references/progressive_delivery.md`
  - Reuse Argo Rollouts scenarios for reliability/availability extensions in full-loop mode.

### Script logic to reuse as interviewer heuristics

These scripts should not be copied as runtime dependencies for the interviewer skill, but their logic should be reused as interview expectations and checklist checks:

- `gitops-workflows/scripts/check_argocd_health.py`
  - Reuse health/sync classification patterns for “How would you debug” answer evaluation.
- `gitops-workflows/scripts/validate_gitops_repo.py`
  - Reuse static anti-pattern checks (`:latest`, missing limits, plain secrets) as intake + scoring signals.
- `gitops-workflows/scripts/secret_audit.py`
  - Reuse secret risk categories and recommended remediation language.
- `gitops-workflows/scripts/applicationset_generator.py`
  - Reuse ApplicationSet structure expectations for design/tradeoff questioning.

### Assets/templates to reference

- `gitops-workflows/assets/applicationsets/cluster-generator.yaml`
  - Reuse as a concrete example for ApplicationSet and sync policy interview prompts.
- `gitops-workflows/assets/argocd/install-argocd-3.x.yaml`
  - Reuse ArgoCD 3.x config concerns (tracking method, resource exclusions, exposure) for architecture questions.
- `gitops-workflows/assets/progressive-delivery/argo-rollouts-canary.yaml`
  - Reuse for progressive delivery questions in reliability rounds.
- `gitops-workflows/assets/secrets/sops-age-config.yaml`
  - Reuse for GitOps secrets management scenario questions.

### Explicit non-reuse / caution items

- Do not copy mutating command behavior from `gitops-workflows/SKILL.md` into interviewer runtime behavior (`kubectl apply`, `argocd app sync`, `argocd app delete`).
- Keep interviewer safety model strictly read-only.
- Treat version-specific claims in `gitops-workflows` as input context, not hard requirements.
- Fix known typo if content is reused from `gitops-workflows/references/oci_artifacts.md` (`kind: OC IRepository`).

## Planned Work

### Phase 1: Scaffold New Skill Folder

1. Copy baseline folder to `devops-gitops-interviewer/`.
2. Keep canonical files:
   - `SKILL.md`
   - `repo-intake.md`
   - `question-templates.md`
   - `rubrics.md`
   - `example-transcript.md`
   - `checklists/best-practices.md`
   - `checklists/technology-comparisons.md`
   - `agents/openai.yaml`
3. Add GitOps depth checklist:
   - `checklists/gitops.md`
4. Keep Kubernetes depth checklist:
   - `checklists/kubernetes.md` (port from `devops-kubernetes-interviewer`)
5. During scaffold, stage reusable source references list (from section above) so each downstream file update has a specific input source.

### Phase 2: Rewrite `SKILL.md` for GitOps-First Interviewing

1. Update frontmatter:
   - `name: devops-gitops-interviewer`
   - description explicitly mentioning ArgoCD + Kubernetes interview intent and triggers.
2. Update skill identity and invocation examples.
3. Update mode menu to prioritize GitOps/Kubernetes:
   - `GitOps-only` deep mode
   - `Kubernetes-only` deep mode
   - `Full loop`
   - `Mixed rapid-fire`
4. Reorder rounds so GitOps and Kubernetes are first.
5. Keep strict per-question flow:
   - ask one question
   - wait for answer
   - follow-ups
   - score block
   - model answer
6. Preserve debrief schema and four-pillars coverage tracking.
7. Preserve read-only tool safety + prohibited commands.

### Phase 3: Strengthen Repo Intake for ArgoCD + Kubernetes

Update `repo-intake.md` to extract GitOps-critical artifacts:

1. ArgoCD:
   - `Application`/`ApplicationSet`
   - app-of-apps hierarchy
   - sync policy (`automated`, `prune`, `selfHeal`)
   - sync waves/hooks
   - projects/RBAC boundaries
   - source types (Helm/Kustomize/plain manifests)
   - repo credential patterns and secret references
2. Kubernetes:
   - workloads, namespaces, services/ingress
   - probes, requests/limits, PDBs
   - RBAC, network policies, storage classes/PVC
3. GitOps pipeline flow:
   - CI image build/tag strategy
   - Git promotion mechanism
   - pull-based deployment handoff to ArgoCD
4. Seed repo-intake heuristics from:
   - `gitops-workflows/references/repo_patterns.md`
   - `gitops-workflows/references/best_practices.md`
   - `gitops-workflows/scripts/validate_gitops_repo.py` logic

### Phase 4: Rebuild Question Bank with GitOps/K8 Depth

Update `question-templates.md`:

1. Round 1 (GitOps/ArgoCD) must include:
   - drift and reconciliation behavior
   - self-heal/prune failure modes
   - app-of-apps vs ApplicationSet tradeoffs
   - sync wave ordering rationale
   - repo credential rotation failures
   - multi-cluster scaling and controller sharding
2. Round 2 (Kubernetes) must include systematic troubleshooting prompts.
3. Maintain required categories per round:
   - What breaks if
   - How would you debug
   - Tradeoff
   - Technology comparison
   - Best practice pillar
4. Keep path placeholders (`{path}`, `{module}`, `{config_detail}`) for repo grounding.
5. Pull concrete scenario seeds from:
   - `gitops-workflows/references/troubleshooting.md`
   - `gitops-workflows/references/argocd_vs_flux.md`
   - `gitops-workflows/references/multi_cluster.md`
   - `gitops-workflows/references/secret_management.md`

### Phase 5: Align Rubrics and Checklists

1. `rubrics.md`
   - make ArgoCD/GitOps rubric the deepest section
   - keep explicit 0-4 criteria + technology justification scoring
   - keep Kubernetes criteria strong (systematic troubleshooting expectations)
2. `checklists/gitops.md` (new)
   - reconciliation model
   - sync policy safety
   - promotion strategy
   - secrets in GitOps
   - disaster recovery for ArgoCD control plane
   - operational debugging commands and triage sequence
   - derive checklist expectations from:
     - `gitops-workflows/references/best_practices.md`
     - `gitops-workflows/references/troubleshooting.md`
     - `gitops-workflows/scripts/check_argocd_health.py` logic
     - `gitops-workflows/scripts/validate_gitops_repo.py` logic
3. `checklists/kubernetes.md`
   - port/enhance from existing Kubernetes interviewer for depth continuity

### Phase 6: Update Examples and Metadata

1. `example-transcript.md`
   - include at least one strong ArgoCD exchange and one Kubernetes troubleshooting exchange
   - include realistic score block + model answer + debrief excerpt
   - include at least one exchange sourced from `OutOfSync`/drift scenarios in `gitops-workflows/references/troubleshooting.md`
2. `agents/openai.yaml`
   - display name: DevOps GitOps Interviewer
   - short description centered on repo-aware ArgoCD/Kubernetes interview
   - default prompt referencing `$devops-gitops-interviewer`

### Phase 7: Validate and Iterate

1. Run structural validation for frontmatter and naming.
2. Run manual dry-runs using at least:
   - GitOps-rich repo
   - Kubernetes-only artifacts
   - sparse repo (generic fallback behavior)
3. Fix gaps and tighten prompts/rubrics where scoring ambiguity appears.

## Deliverables

- `devops-gitops-interviewer/SKILL.md`
- `devops-gitops-interviewer/repo-intake.md`
- `devops-gitops-interviewer/question-templates.md`
- `devops-gitops-interviewer/rubrics.md`
- `devops-gitops-interviewer/example-transcript.md`
- `devops-gitops-interviewer/checklists/gitops.md` (new)
- `devops-gitops-interviewer/checklists/kubernetes.md`
- `devops-gitops-interviewer/checklists/best-practices.md`
- `devops-gitops-interviewer/checklists/technology-comparisons.md`
- `devops-gitops-interviewer/agents/openai.yaml`

## Exit Criteria

- Skill triggers correctly for GitOps/ArgoCD/Kubernetes interview requests.
- Repo intake consistently discovers and maps ArgoCD/Kubernetes artifacts.
- Interview rounds are repo-aware where artifacts exist and generic only when necessary.
- Every question receives score + tech justification + model answer.
- Debrief includes score summary, pillar coverage, strengths/gaps, and actionable drills.
- Safety rules remain read-only and prohibit state-mutating commands.
