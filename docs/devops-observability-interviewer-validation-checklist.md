# Validation Checklist: `devops-observability-interviewer`

Use this checklist to verify the new interviewer skill before considering it complete.

## 1) Skill package structure

- [ ] Folder exists: `devops-observability-interviewer/`
- [ ] Required file exists: `devops-observability-interviewer/SKILL.md`
- [ ] Metadata file exists: `devops-observability-interviewer/agents/openai.yaml`
- [ ] Core interview files exist:
  - [ ] `devops-observability-interviewer/repo-intake.md`
  - [ ] `devops-observability-interviewer/question-templates.md`
  - [ ] `devops-observability-interviewer/rubrics.md`
  - [ ] `devops-observability-interviewer/example-transcript.md`
- [ ] Checklist files exist:
  - [ ] `devops-observability-interviewer/checklists/monitoring.md`
  - [ ] `devops-observability-interviewer/checklists/logging-efk.md`
  - [ ] `devops-observability-interviewer/checklists/best-practices.md`
  - [ ] `devops-observability-interviewer/checklists/technology-comparisons.md`

## 2) Triggering and metadata correctness

- [ ] `SKILL.md` frontmatter contains only `name` and `description`
- [ ] Name is exactly `devops-observability-interviewer`
- [ ] Description clearly states:
  - [ ] structured repo-aware mock interview behavior
  - [ ] EFK (Kibana) logging focus
  - [ ] Prometheus + Grafana monitoring focus
  - [ ] interview-practice/readiness trigger contexts
- [ ] `agents/openai.yaml` matches the skill:
  - [ ] display name
  - [ ] short description
  - [ ] default prompt using `$devops-observability-interviewer`

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

## 4) Repo intake depth

- [ ] Prometheus artifacts are discovered:
  - [ ] `Prometheus` / helm values / scrape config
  - [ ] `ServiceMonitor`, `PodMonitor`
  - [ ] `PrometheusRule` and recording rules
  - [ ] Alertmanager config/routing
  - [ ] retention and remote-write/federation settings
- [ ] Grafana artifacts are discovered:
  - [ ] dashboard JSON/provisioning
  - [ ] datasources and folders
  - [ ] alert panel links/runbook links
- [ ] EFK artifacts are discovered:
  - [ ] Fluent Bit/Fluentd config (inputs/parsers/filters/outputs)
  - [ ] Elasticsearch config/index templates/ILM
  - [ ] Kibana index patterns/saved searches (if present)
- [ ] SLO/runbook artifacts are discovered:
  - [ ] SLI/SLO definitions
  - [ ] burn-rate alert rules
  - [ ] incident runbooks/postmortems

## 5) Question template quality

- [ ] Every round includes all mandatory categories:
  - [ ] What breaks if...
  - [ ] How would you debug...
  - [ ] Tradeoff
  - [ ] Technology comparison
  - [ ] Best practice pillar
- [ ] EFK scenarios are present:
  - [ ] missing logs in Kibana
  - [ ] parser failures and dropped logs
  - [ ] ILM/retention errors and storage pressure
  - [ ] field mapping/cardinality issues
- [ ] Prometheus/Grafana scenarios are present:
  - [ ] scrape failures/stale metrics
  - [ ] histogram misuse and percentile errors
  - [ ] noisy or non-actionable alerts
  - [ ] burn-rate alert tuning
  - [ ] dashboard design and query performance tradeoffs

## 6) Rubric and checklist alignment

- [ ] `rubrics.md` has universal score definitions
- [ ] `rubrics.md` has explicit domain sections for:
  - [ ] observability strategy
  - [ ] Prometheus/PromQL
  - [ ] Grafana/Alertmanager
  - [ ] EFK/Kibana operations
  - [ ] SLO/incident response
  - [ ] security/compliance
  - [ ] cost optimization
- [ ] `checklists/monitoring.md` and `checklists/logging-efk.md` are used as scoring sources of truth

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

## 9) Example transcript gate

- [ ] Transcript shows repo intake and round availability
- [ ] Transcript includes one scored Prometheus/Grafana exchange
- [ ] Transcript includes one scored EFK/Kibana exchange
- [ ] Transcript includes debrief excerpt
- [ ] Score blocks match rubric format and labels

## 10) Reuse congruence with `monitoring-observability`

- [ ] `checklists/monitoring.md` reflects:
  - [ ] `monitoring-observability/references/metrics_design.md`
  - [ ] `monitoring-observability/references/alerting_best_practices.md`
  - [ ] `monitoring-observability/references/slo_sla_guide.md`
- [ ] `checklists/logging-efk.md` reflects:
  - [ ] `monitoring-observability/references/logging_guide.md`
- [ ] `question-templates.md` contains scenarios derived from:
  - [ ] `monitoring-observability/scripts/log_analyzer.py` heuristics
  - [ ] `monitoring-observability/scripts/alert_quality_checker.py` heuristics
  - [ ] `monitoring-observability/scripts/analyze_metrics.py` anomaly/trend logic
- [ ] Template references appear where applicable:
  - [ ] `monitoring-observability/assets/templates/prometheus-alerts/webapp-alerts.yml`
  - [ ] `monitoring-observability/assets/templates/prometheus-alerts/kubernetes-alerts.yml`
  - [ ] `monitoring-observability/assets/templates/runbooks/incident-runbook-template.md`

## 11) Manual scenario tests

- [ ] Scenario A: repo has both EFK and Prometheus/Grafana artifacts
  - [ ] repo map is path-accurate
  - [ ] questions are strongly repo-grounded
- [ ] Scenario B: only Prometheus/Grafana artifacts exist
  - [ ] EFK round becomes limited/generic without fake paths
- [ ] Scenario C: only logging artifacts exist
  - [ ] monitoring rounds become limited/generic without fake paths
- [ ] Scenario D: sparse repo
  - [ ] fallback generic interview still complete and scored

## 12) Validation commands

- [ ] Run structural validation:
  - [ ] `python3 /Users/romansmirnov/.codex/skills/.system/skill-creator/scripts/quick_validate.py devops-observability-interviewer`
- [ ] Run phase and round sanity checks:
  - [ ] `rg -n "## Phase|### Rounds|### Per-Question Flow|## Tool Safety|## Behavioral Rules" devops-observability-interviewer/SKILL.md`
  - [ ] `rg -n "^## Round|### What Breaks If|### How Would You Debug|### Tradeoff|### Technology Comparison|### Best Practice Pillar" devops-observability-interviewer/question-templates.md`
  - [ ] `rg -n "Prometheus|Grafana|EFK|Elasticsearch|Fluent Bit|Kibana|SLO|burn rate" devops-observability-interviewer/* devops-observability-interviewer/checklists/*.md`

## Sign-off

- [ ] All checklist items passed
- [ ] Remaining gaps documented
- [ ] Plan approved before full implementation
