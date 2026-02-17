# Validation Checklist: `devops-aws-interviewer`

Use this checklist to verify the new AWS interviewer skill before considering it complete.

## 1) Skill package structure

- [ ] Folder exists: `devops-aws-interviewer/`
- [ ] Required file exists: `devops-aws-interviewer/SKILL.md`
- [ ] Metadata file exists: `devops-aws-interviewer/agents/openai.yaml`
- [ ] Core interview files exist:
  - [ ] `devops-aws-interviewer/repo-intake.md`
  - [ ] `devops-aws-interviewer/question-templates.md`
  - [ ] `devops-aws-interviewer/rubrics.md`
  - [ ] `devops-aws-interviewer/example-transcript.md`
- [ ] Checklist files exist:
  - [ ] `devops-aws-interviewer/checklists/aws.md`
  - [ ] `devops-aws-interviewer/checklists/best-practices.md`
  - [ ] `devops-aws-interviewer/checklists/technology-comparisons.md`

## 2) Triggering and metadata correctness

- [ ] `SKILL.md` frontmatter contains only `name` and `description`
- [ ] Name is exactly `devops-aws-interviewer`
- [ ] Description clearly states:
  - [ ] structured repo-aware DevOps mock interview behavior
  - [ ] AWS architecture and platform depth focus
  - [ ] follow-up/scoring/debrief behavior
  - [ ] interview-practice/readiness trigger contexts
- [ ] `agents/openai.yaml` matches the skill:
  - [ ] display name
  - [ ] short description
  - [ ] default prompt using `$devops-aws-interviewer`

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

## 4) AWS repo-intake depth

- [ ] Intake scans and maps key AWS domains:
  - [ ] IAM roles, policies, trust relationships, permission boundaries
  - [ ] VPC, subnets, route tables, NAT, gateways, VPC endpoints
  - [ ] compute/runtime (EKS/ECS/Lambda/EC2)
  - [ ] edge/DNS (CloudFront, Route 53, ALB/NLB/API Gateway, ACM, WAF)
  - [ ] storage/data (S3, EBS/EFS, RDS/Aurora, DynamoDB)
  - [ ] secrets and encryption (KMS, Secrets Manager, SSM)
  - [ ] observability (CloudWatch logs/metrics/alarms/dashboards, X-Ray)
  - [ ] cost/governance indicators (tagging, lifecycle policies, budgets/anomaly controls)
- [ ] Output format matches interviewer contract:
  - [ ] layer table with artifacts + key paths
  - [ ] per-round availability classification (`Available`/`Limited`/`Generic`/`Skipped`)

## 5) Question template quality

- [ ] Every round includes all mandatory categories:
  - [ ] What breaks if...
  - [ ] How would you debug...
  - [ ] Tradeoff
  - [ ] Technology comparison
  - [ ] Best practice pillar
- [ ] AWS failure/debug scenarios are present:
  - [ ] IAM/STS assume-role failures
  - [ ] networking path failures (route/NAT/VPC endpoint/security group/NACL)
  - [ ] service-specific incidents (EKS/ECS/Lambda/EC2)
  - [ ] data/storage incidents (RDS failover/backups, DynamoDB throttling, S3 policy/encryption issues)
  - [ ] edge/DNS incidents (CloudFront/Route53/ALB/API Gateway behavior)
  - [ ] CloudWatch/X-Ray observability and alerting diagnosis
- [ ] Technology-comparison prompts require formal definitions + alternatives + project-fit justification

## 6) Rubric and checklist alignment

- [ ] `rubrics.md` includes:
  - [ ] universal score scale
  - [ ] technology-justification score criteria
  - [ ] AWS-focused domain extensions (architecture, IAM/security, networking/reliability, observability, cost/governance)
- [ ] `checklists/aws.md` is used as AWS scoring source of truth
- [ ] `checklists/best-practices.md` enforces pillar coverage expectations
- [ ] `checklists/technology-comparisons.md` supports "why this, not that" scoring

## 7) Debrief quality

- [ ] Debrief includes score summary table
- [ ] Debrief includes best-practice pillar coverage table
- [ ] Debrief includes strengths and gaps
- [ ] Debrief includes interview-ready talking points
- [ ] Debrief includes targeted drill plan with actionable exercises tied to pillars

## 8) Safety and non-destructive behavior

- [ ] Allowed commands are read-only only
- [ ] Prohibited command list explicitly blocks mutating actions
- [ ] Skill refuses prohibited requests during interview mode
- [ ] AWS mutating command families are blocked (`create-*`, `delete-*`, `put-*`, `update-*`, `modify-*`, `terminate-*`, etc.)

## 9) Example transcript gate

- [ ] Transcript shows repo intake and round availability
- [ ] Transcript includes one scored AWS architecture exchange
- [ ] Transcript includes one scored IAM/security exchange
- [ ] Transcript includes one scored reliability/observability exchange
- [ ] Transcript includes debrief excerpt
- [ ] Score blocks match rubric format and labels

## 10) Reuse congruence with `monitoring-observability` and `aws-cost-optimization`

- [ ] AWS observability/reliability sections reflect:
  - [ ] `monitoring-observability/references/tool_comparison.md` (CloudWatch/X-Ray tradeoffs)
  - [ ] `monitoring-observability/references/alerting_best_practices.md` (actionable alert quality)
  - [ ] `monitoring-observability/references/slo_sla_guide.md` (SLO/error-budget rigor)
  - [ ] `monitoring-observability/references/logging_guide.md` (CloudWatch Logs patterns)
- [ ] `question-templates.md` includes scenarios derived from script heuristics:
  - [ ] `monitoring-observability/scripts/analyze_metrics.py` (metric trend/anomaly framing)
  - [ ] `monitoring-observability/scripts/alert_quality_checker.py` (alert quality checks)
  - [ ] `monitoring-observability/scripts/slo_calculator.py` (burn-rate/severity reasoning)
  - [ ] `monitoring-observability/scripts/health_check_validator.py` (health endpoint quality)
  - [ ] `monitoring-observability/scripts/log_analyzer.py` (error-pattern diagnostics)
- [ ] Optional runbook/template references appear where relevant:
  - [ ] `monitoring-observability/assets/templates/runbooks/incident-runbook-template.md`
  - [ ] `monitoring-observability/assets/templates/prometheus-alerts/webapp-alerts.yml`
  - [ ] `monitoring-observability/assets/templates/prometheus-alerts/kubernetes-alerts.yml`
- [ ] AWS FinOps and cost-governance sections reflect:
  - [ ] `aws-cost-optimization/references/best_practices.md` (quick wins + strategic initiatives)
  - [ ] `aws-cost-optimization/references/service_alternatives.md` (cost tradeoff prompts)
  - [ ] `aws-cost-optimization/references/finops_governance.md` (monthly cadence + ownership model)
- [ ] `question-templates.md` includes scenarios derived from cost script heuristics:
  - [ ] `aws-cost-optimization/scripts/find_unused_resources.py` (waste discovery categories)
  - [ ] `aws-cost-optimization/scripts/cost_anomaly_detector.py` (anomaly triage flow)
  - [ ] `aws-cost-optimization/scripts/rightsizing_analyzer.py` (utilization-based rightsizing)
  - [ ] `aws-cost-optimization/scripts/analyze_ri_recommendations.py` (RI/SP suitability)
  - [ ] `aws-cost-optimization/scripts/spot_recommendations.py` (Spot candidate reasoning)
  - [ ] `aws-cost-optimization/scripts/detect_old_generations.py` (migration opportunities)
- [ ] Optional FinOps template references appear where relevant:
  - [ ] `aws-cost-optimization/assets/templates/monthly_cost_report.md`

## 11) Manual scenario tests

- [ ] Scenario A: AWS-rich repo with IaC + runtime + observability artifacts
  - [ ] repo map is path-accurate
  - [ ] AWS-core mode is strongly repo-grounded
- [ ] Scenario B: repo has IAM/networking only, little runtime observability
  - [ ] observability rounds become `Limited`/`Generic` without fabricated paths
- [ ] Scenario C: repo has app artifacts with minimal AWS IaC
  - [ ] AWS questions fall back to rigorous generic mode correctly
- [ ] Scenario D: sparse repo
  - [ ] full interview still completes with consistent scoring and debrief structure

## 12) Validation commands

- [ ] Run structural validation:
  - [ ] `python3 /Users/romansmirnov/.codex/skills/.system/skill-creator/scripts/quick_validate.py devops-aws-interviewer`
- [ ] Run phase and round sanity checks:
  - [ ] `rg -n "## Phase|### Rounds|### Per-Question Flow|## Tool Safety|## Behavioral Rules" devops-aws-interviewer/SKILL.md`
  - [ ] `rg -n "^## Round|### What Breaks If|### How Would You Debug|### Tradeoff|### Technology Comparison|### Best Practice Pillar" devops-aws-interviewer/question-templates.md`
  - [ ] `rg -n "IAM|VPC|NAT|CloudFront|Route 53|CloudWatch|X-Ray|KMS|S3|RDS|DynamoDB" devops-aws-interviewer/* devops-aws-interviewer/checklists/*.md`
  - [ ] `rg -n "FinOps|Savings Plans|Reserved Instances|Spot|Graviton|gp2|gp3|monthly review|cost anomaly" devops-aws-interviewer/* devops-aws-interviewer/checklists/*.md`

## Sign-off

- [ ] All checklist items passed
- [ ] Remaining gaps documented
- [ ] Plan approved before full implementation
