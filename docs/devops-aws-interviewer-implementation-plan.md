# Implementation Plan: `devops-aws-interviewer`

## Goal

Create a new interviewer skill focused on **AWS platform depth** that:

- scans the repository and builds a repo map of AWS-relevant artifacts
- interviews the user in a structured, repo-aware loop
- evaluates AWS architecture, operations, security, reliability, and cost tradeoffs
- scores every answer and ends with a debrief + targeted drill plan

The skill should match the existing interviewer contract used by:

- `devops-kubernetes-interviewer/`
- `devops-gitops-interviewer/`
- `devops-terraform-interviewer/`

---

## What Was Reviewed

### Existing interviewer skill contract (reused)

All existing interviewer skills share the same mechanics:

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

### Monitoring skill content reviewed for reuse

Primary source: `monitoring-observability/`

Reusable references for AWS interviewer rounds:

- `monitoring-observability/references/tool_comparison.md`
  - reuse CloudWatch + X-Ray vs Prometheus/Datadog tradeoff framing
- `monitoring-observability/references/alerting_best_practices.md`
  - reuse actionable alert quality criteria and runbook expectations
- `monitoring-observability/references/slo_sla_guide.md`
  - reuse SLI/SLO/error-budget interview depth for reliability rounds
- `monitoring-observability/references/logging_guide.md`
  - reuse CloudWatch Logs query patterns and logging pipeline evaluation signals
- `monitoring-observability/references/metrics_design.md`
  - reuse metric design and cardinality expectations for AWS observability decisions

Reusable scripts (as optional read-only analysis helpers and scoring heuristics):

- `monitoring-observability/scripts/analyze_metrics.py`
  - includes CloudWatch mode; reuse for metric trend/anomaly reasoning
- `monitoring-observability/scripts/alert_quality_checker.py`
  - reuse rule-quality expectations for alerting questions
- `monitoring-observability/scripts/slo_calculator.py`
  - reuse error-budget and burn-rate reasoning patterns
- `monitoring-observability/scripts/health_check_validator.py`
  - reuse health endpoint quality expectations in reliability rounds
- `monitoring-observability/scripts/log_analyzer.py`
  - reuse structured logging and error-pattern analysis logic

Optional templates to reference in drills/examples:

- `monitoring-observability/assets/templates/runbooks/incident-runbook-template.md`
- `monitoring-observability/assets/templates/prometheus-alerts/webapp-alerts.yml`
- `monitoring-observability/assets/templates/prometheus-alerts/kubernetes-alerts.yml`

### AWS cost optimization skill content reviewed for reuse

Primary source: `aws-cost-optimization/`

Reusable references for FinOps and cost-governance rounds:

- `aws-cost-optimization/references/best_practices.md`
  - reuse quick wins (cleanup, lifecycle, gp2 -> gp3) and strategic initiatives (Spot, Graviton, commitments)
- `aws-cost-optimization/references/service_alternatives.md`
  - reuse service-level cost tradeoff prompts (compute, storage, networking, data)
- `aws-cost-optimization/references/finops_governance.md`
  - reuse monthly cadence, tagging/accountability, budget governance, and KPI prompts

Reusable scripts (as read-only analysis heuristics and interview expectations):

- `aws-cost-optimization/scripts/find_unused_resources.py`
  - reuse unused-resource classification and safety framing for cleanup decisions
- `aws-cost-optimization/scripts/cost_anomaly_detector.py`
  - reuse anomaly-triage flow and top-driver analysis
- `aws-cost-optimization/scripts/rightsizing_analyzer.py`
  - reuse utilization-based rightsizing expectations
- `aws-cost-optimization/scripts/analyze_ri_recommendations.py`
  - reuse RI/Savings Plans suitability logic
- `aws-cost-optimization/scripts/spot_recommendations.py`
  - reuse Spot suitability scoring heuristics
- `aws-cost-optimization/scripts/detect_old_generations.py`
  - reuse old-generation migration and Graviton opportunity framing

Optional template to reference in drills/examples:

- `aws-cost-optimization/assets/templates/monthly_cost_report.md`

---

## Planned Skill Package

Create folder:

- `devops-aws-interviewer/`

Planned files:

- `devops-aws-interviewer/SKILL.md`
- `devops-aws-interviewer/repo-intake.md`
- `devops-aws-interviewer/question-templates.md`
- `devops-aws-interviewer/rubrics.md`
- `devops-aws-interviewer/example-transcript.md`
- `devops-aws-interviewer/checklists/aws.md` (new domain checklist)
- `devops-aws-interviewer/checklists/best-practices.md`
- `devops-aws-interviewer/checklists/technology-comparisons.md`
- `devops-aws-interviewer/agents/openai.yaml`

---

## Implementation Phases

### Phase 1: Scaffold the new interviewer skill

1. Copy baseline structure from `devops-terraform-interviewer/` (closest to AWS-heavy shape).
2. Keep all core interviewer assets.
3. Add/replace domain checklist with `checklists/aws.md`.

### Phase 2: Author `SKILL.md` for AWS-first interviewing

1. Set frontmatter:
   - `name: devops-aws-interviewer`
   - description that clearly triggers on AWS interview practice requests
2. Keep standard interview mechanics and safety model.
3. Make AWS depth explicit in framing:
   - architecture choices
   - IAM and security boundaries
   - reliability and incident readiness
   - cost optimization and governance
4. Define rounds with AWS-first ordering, for example:
   - AWS architecture and networking foundations
   - IAM, identity federation, and access boundaries
   - compute/runtime platform (EKS/ECS/Lambda/EC2) tradeoffs
   - data/storage and resilience strategy
   - IaC + CI/CD + GitOps integration on AWS
   - observability and incident response on AWS
   - security/compliance and cost optimization
5. Define mode menu (full loop, AWS-core, IAM/security, reliability drill, rapid-fire).

### Phase 3: Build `repo-intake.md` for AWS artifact discovery

Capture and classify:

1. Account and governance signals:
   - IAM roles/policies, permission boundaries, org/SCP evidence
2. Networking:
   - VPCs, subnets, route tables, NAT, IGW, VPC endpoints, Transit Gateway
3. Compute/runtime:
   - EKS/ECS/Lambda/EC2 configs, scaling policies, launch templates
4. Edge and DNS:
   - ALB/NLB/API Gateway, CloudFront, Route 53, ACM, WAF
5. Storage/data:
   - S3, EBS/EFS, RDS/Aurora, DynamoDB, backup/restore controls
6. Security:
   - KMS usage, Secrets Manager/SSM, CloudTrail, GuardDuty/Security Hub signals
7. Observability:
   - CloudWatch metrics/alarms/log groups/dashboards, X-Ray traces
8. Cost controls:
   - tagging strategy, lifecycle policies, budgets/anomaly detection, spot/savings plans hints

### Phase 4: Create scoring checklists

1. `checklists/aws.md` (new)
   - core AWS architecture expectations
   - IAM and least-privilege patterns
   - networking and failure-domain reasoning
   - service tradeoffs and incident-debug methodology
2. `checklists/best-practices.md`
   - keep four-pillar framing; adapt entries to AWS interviewer context
3. `checklists/technology-comparisons.md`
   - keep comparison matrix pattern; emphasize AWS service choices and alternatives

### Phase 5: Build `question-templates.md`

For each round, enforce mandatory categories:

1. What breaks if...
2. How would you debug...
3. Tradeoff
4. Technology comparison (formal definitions required)
5. Best practice pillar

Must include AWS-specific scenario inventory:

- IAM trust-policy and permission-boundary failures
- NAT/VPC endpoint/route-table failure scenarios
- EKS/ECS/Lambda/EC2 tradeoff and incident scenarios
- CloudFront/ALB/Route53 debugging flows
- RDS/DynamoDB/S3 resilience and backup/restore tradeoffs
- CloudWatch/X-Ray/alarms/runbook questions (using monitoring-observability heuristics)
- FinOps scenarios (using aws-cost-optimization heuristics):
  - quick-win cleanup prioritization
  - RI/Savings Plans baseline suitability
  - Spot candidate selection boundaries
  - old-generation and Graviton migration decisions
  - monthly governance cadence and accountability

### Phase 6: Author `rubrics.md`

1. Keep universal scoring + technology justification scoring.
2. Add AWS-focused domain sections:
   - architecture and service-selection depth
   - IAM/security and blast-radius control
   - networking and reliability diagnostics
   - observability and incident response on AWS
   - cost/governance optimization maturity
3. Keep rubric language aligned to interviewer score labels used in existing skills.

### Phase 7: Create `example-transcript.md`

Transcript should include:

1. repo map excerpt with AWS artifact availability classification
2. one scored AWS architecture exchange
3. one scored IAM/security exchange
4. one scored reliability/observability exchange
5. debrief excerpt with drill plan

### Phase 8: Final metadata

Create `agents/openai.yaml` with:

- display name: `DevOps AWS Interviewer`
- short description: repo-aware AWS-focused DevOps mock interview
- default prompt using `$devops-aws-interviewer`

### Phase 9: Validate and iterate

1. Run structural validation:
   - `python3 /Users/romansmirnov/.codex/skills/.system/skill-creator/scripts/quick_validate.py devops-aws-interviewer`
2. Run content sanity checks for phases/rounds/scoring blocks.
3. Dry-run against:
   - AWS-rich infra repo
   - partial AWS artifact repo
   - sparse repo (generic fallback behavior)

---

## Exit Criteria

- Skill triggers correctly for AWS interview-practice requests.
- Repo map reliably identifies AWS architecture artifacts and references real paths.
- Questions are repo-grounded when artifacts exist; generic only when necessary.
- Every question yields score + tech justification + model answer.
- Debrief includes score summary, pillar coverage, strengths/gaps, and actionable drills.
- Safety remains strictly read-only; no mutating cloud or cluster commands.
- Monitoring-observability reuse is visible in observability/incident expectations without changing interviewer read-only behavior.
- aws-cost-optimization reuse is visible in FinOps and cost-governance expectations without adding mutating automation behavior.
