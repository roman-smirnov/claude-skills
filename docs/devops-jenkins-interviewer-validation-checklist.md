# Validation Checklist: `devops-jenkins-interviewer`

Use this checklist to verify the new Jenkins interviewer skill before considering it complete.

## 1) Skill package structure

- [ ] Folder exists: `devops-jenkins-interviewer/`
- [ ] Required file exists: `devops-jenkins-interviewer/SKILL.md`
- [ ] Metadata file exists: `devops-jenkins-interviewer/agents/openai.yaml`
- [ ] Core interview files exist:
  - [ ] `devops-jenkins-interviewer/repo-intake.md`
  - [ ] `devops-jenkins-interviewer/question-templates.md`
  - [ ] `devops-jenkins-interviewer/rubrics.md`
  - [ ] `devops-jenkins-interviewer/example-transcript.md`
- [ ] Checklist files exist:
  - [ ] `devops-jenkins-interviewer/checklists/jenkins.md`
  - [ ] `devops-jenkins-interviewer/checklists/aws.md`
  - [ ] `devops-jenkins-interviewer/checklists/best-practices.md`
  - [ ] `devops-jenkins-interviewer/checklists/technology-comparisons.md`

## 2) Triggering and metadata correctness

- [ ] `SKILL.md` frontmatter contains only `name` and `description`
- [ ] Name is exactly `devops-jenkins-interviewer`
- [ ] Description clearly states:
  - [ ] structured repo-aware DevOps mock interview behavior
  - [ ] Jenkins-first interview depth
  - [ ] AWS integration and cloud operations context
  - [ ] follow-up/scoring/debrief behavior
- [ ] `agents/openai.yaml` matches the skill:
  - [ ] display name
  - [ ] short description
  - [ ] default prompt using `$devops-jenkins-interviewer`

## 3) Interview flow contract (`SKILL.md`)

- [ ] Includes all phases:
  - [ ] repo intake
  - [ ] mode selection
  - [ ] interview loop
  - [ ] debrief
- [ ] Mode selection requires explicit user choice before questions start
- [ ] Per-question flow enforced:
  - [ ] ask one question
  - [ ] wait for answer
  - [ ] ask 1-2 follow-ups
  - [ ] score via rubric
  - [ ] provide concise model answer
- [ ] Repo-grounding rule enforced (no fabricated paths)
- [ ] Four-pillar coverage required across full interview

## 4) Jenkins repo-intake depth

- [ ] Intake discovers and maps Jenkins artifacts:
  - [ ] `Jenkinsfile*`
  - [ ] shared libraries (`vars/`, `src/`, `resources/`)
  - [ ] JCasC files (`jenkins.yaml`, `jcasc/*.yaml` patterns)
  - [ ] plugin manifests (`plugins.txt`, `plugins.yaml`, Helm values)
  - [ ] pipeline job DSL/seed jobs (if present)
- [ ] Intake extracts Jenkins design signals:
  - [ ] stage structure and ordering
  - [ ] parallelization and fan-in/fan-out behavior
  - [ ] timeout/retry/disableConcurrentBuilds/buildDiscarder controls
  - [ ] artifact flow (`stash/unstash`, archive)
- [ ] Intake captures Jenkins security signals:
  - [ ] credentials usage and scope
  - [ ] `withCredentials` usage and masking behavior
  - [ ] secret anti-patterns (plain text env vars, checked-in secrets)
- [ ] Intake captures AWS integration signals:
  - [ ] IAM role/trust model
  - [ ] deployment targets (ECR/EKS/ECS/Lambda/EC2)
  - [ ] artifact registries and release path
- [ ] Output format matches interviewer contract:
  - [ ] layer table with artifacts + key paths
  - [ ] per-round availability classification (`Available`/`Limited`/`Generic`/`Skipped`)

## 5) Jenkins domain depth (`checklists/jenkins.md` + templates)

- [ ] Candidate evaluation includes controller vs agent model correctness
- [ ] Declarative vs scripted pipeline tradeoffs are assessed
- [ ] Multibranch and Jenkinsfile-as-code expectations are assessed
- [ ] Queueing/executor/agent capacity reasoning is assessed
- [ ] Jenkins reliability and troubleshooting flow is explicit and ordered
- [ ] Plugin and JCasC governance expectations are present
- [ ] Security expectations include credentials binding, least privilege, and supply chain controls

## 6) AWS integration depth (`checklists/aws.md` + templates)

- [ ] Jenkins-to-AWS identity model is covered (OIDC/assume-role/static credential tradeoffs)
- [ ] IAM trust/policy debugging questions exist (`AccessDenied`, wrong principal, boundary/SCP effects)
- [ ] Deployment integration questions exist (ECR push/pull, EKS deploy handoff, rollback controls)
- [ ] AWS cost/risk implications of CI architecture are included (agent compute, egress, registry/storage)

## 7) Question template quality

- [ ] Every round includes all mandatory categories:
  - [ ] What breaks if...
  - [ ] How would you debug...
  - [ ] Tradeoff
  - [ ] Technology comparison
  - [ ] Best practice pillar
- [ ] Jenkins failure/debug scenarios are present:
  - [ ] agent offline / label mismatch / queue starvation
  - [ ] workspace contention and concurrent build corruption
  - [ ] missing timeout/retry and hung pipelines
  - [ ] credential injection failures / secret exposure risks
  - [ ] plugin incompatibility or upgrade regression
- [ ] Technology-comparison prompts require formal definitions + alternatives + project-fit justification

## 8) Rubric and checklist alignment

- [ ] `rubrics.md` includes:
  - [ ] universal score scale
  - [ ] technology-justification score criteria
  - [ ] Jenkins-focused domain extensions
  - [ ] AWS integration scoring extensions
- [ ] `checklists/jenkins.md` is used as Jenkins scoring source of truth
- [ ] `checklists/aws.md` is used where cloud-depth scoring is required
- [ ] `checklists/best-practices.md` enforces four-pillar coverage
- [ ] `checklists/technology-comparisons.md` supports "why this, not that" scoring

## 9) Debrief quality

- [ ] Debrief includes score summary table
- [ ] Debrief includes best-practice pillar coverage table
- [ ] Debrief includes strengths and gaps
- [ ] Debrief includes interview-ready talking points
- [ ] Debrief includes targeted drill plan with actionable exercises tied to pillars

## 10) Safety and non-destructive behavior

- [ ] Allowed commands are read-only only
- [ ] Prohibited command list explicitly blocks mutating actions
- [ ] Skill refuses prohibited requests during interview mode
- [ ] AWS mutating command families are blocked (`create-*`, `delete-*`, `put-*`, `update-*`, `modify-*`, `terminate-*`, etc.)
- [ ] Jenkins mutating operations are blocked in interview mode (job reconfigure/restart/replay/build trigger)

## 11) Example transcript gate

- [ ] Transcript shows repo intake and round availability
- [ ] Transcript includes one scored Jenkins architecture exchange
- [ ] Transcript includes one scored Jenkins troubleshooting exchange
- [ ] Transcript includes one scored Jenkins+AWS identity/deployment exchange
- [ ] Transcript includes debrief excerpt
- [ ] Score blocks match rubric format and labels

## 12) Reuse congruence with `ci-cd-jenkins` and `ci-cd`

- [ ] Jenkins checklist/question content reflects:
  - [ ] `ci-cd-jenkins/references/jenkins.md`
  - [ ] `ci-cd-jenkins/assets/templates/jenkins/*.Jenkinsfile`
  - [ ] Jenkins heuristics from `ci-cd-jenkins/scripts/pipeline_analyzer.py`
- [ ] Security and DevSecOps prompts reflect:
  - [ ] `ci-cd-jenkins/references/security.md`
  - [ ] `ci-cd-jenkins/references/devsecops.md`
- [ ] Reliability and optimization prompts reflect:
  - [ ] `ci-cd-jenkins/references/troubleshooting.md`
  - [ ] `ci-cd-jenkins/references/optimization.md`
  - [ ] CI health framing from `ci-cd/scripts/ci_health.py`
- [ ] No frontmatter/name mismatch is copied from `ci-cd-jenkins/SKILL.md` (`name: ci-cd`)

## 13) Manual scenario tests

- [ ] Scenario A: Jenkins-heavy repo with clear AWS deployment integration
  - [ ] repo map is path-accurate
  - [ ] Jenkins-only mode is strongly repo-grounded
- [ ] Scenario B: AWS-rich repo with limited Jenkins artifacts
  - [ ] Jenkins rounds become `Limited`/`Generic` without fabricated paths
  - [ ] AWS integration round remains strong
- [ ] Scenario C: repo with Jenkinsfile but no AWS artifacts
  - [ ] AWS-focused questions remain rigorous but generic
  - [ ] debrief still reports missing evidence correctly
- [ ] Scenario D: sparse repo
  - [ ] full interview still completes with consistent scoring and debrief structure

## 14) Validation commands

- [ ] Run structural validation:
  - [ ] `python3 /Users/romansmirnov/.codex/skills/.system/skill-creator/scripts/quick_validate.py devops-jenkins-interviewer`
- [ ] Run phase and round sanity checks:
  - [ ] `rg -n "## Phase|### Rounds|### Per-Question Flow|## Tool Safety|## Behavioral Rules" devops-jenkins-interviewer/SKILL.md`
  - [ ] `rg -n "^## Round|### What Breaks If|### How Would You Debug|### Tradeoff|### Technology Comparison|### Best Practice Pillar" devops-jenkins-interviewer/question-templates.md`
  - [ ] `rg -n "Jenkins|Jenkinsfile|agent|withCredentials|JCasC|plugin|OIDC|IAM|ECR|EKS" devops-jenkins-interviewer/* devops-jenkins-interviewer/checklists/*.md`

## Sign-off

- [ ] All checklist items passed
- [ ] Remaining gaps documented
- [ ] Plan approved before full implementation
