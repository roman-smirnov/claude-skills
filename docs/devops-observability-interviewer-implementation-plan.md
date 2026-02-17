# Implementation Plan: `devops-observability-interviewer`

## Goal

Create a new interviewer skill focused on **DevOps observability depth** that:

- scans the repository and builds a repo map
- interviews the candidate in a structured loop
- emphasizes **Kibana-based EFK logging** and **Prometheus + Grafana monitoring**
- scores each answer, tracks gaps, and ends with a debrief + drill plan

The skill should match the existing interviewer style used by:

- `devops-kubernetes-interviewer/`
- `devops-gitops-interviewer/`
- `devops-terraform-interviewer/`

---

## What Was Reviewed

### Existing interviewer skill contract (reused)

All three interviewer skills share the same contract:

1. Phase 1: repo intake and repo map
2. Phase 2: mode selection
3. Phase 3: interview loop (ask -> wait -> follow-ups -> score -> model answer)
4. Phase 4: debrief (summary tables, strengths/gaps, drill plan)

Common reusable files/patterns:

- `SKILL.md` with strict behavioral and safety rules
- `repo-intake.md` with domain-specific scan workflow
- `question-templates.md` with mandatory categories per round
- `rubrics.md` with universal scale + domain extensions
- `example-transcript.md` showing expected format
- `checklists/` used as source of truth for scoring
- `agents/openai.yaml` for display metadata and default prompt

### Monitoring skill content reviewed for reuse

Primary source: `monitoring-observability/`

Reusable references:

- `monitoring-observability/references/metrics_design.md`
- `monitoring-observability/references/logging_guide.md`
- `monitoring-observability/references/alerting_best_practices.md`
- `monitoring-observability/references/slo_sla_guide.md`
- `monitoring-observability/references/tool_comparison.md`
- `monitoring-observability/references/tracing_guide.md` (secondary)

Reusable scripts (as interview expectations and optional read-only analysis helpers):

- `monitoring-observability/scripts/analyze_metrics.py`
- `monitoring-observability/scripts/log_analyzer.py`
- `monitoring-observability/scripts/alert_quality_checker.py`
- `monitoring-observability/scripts/slo_calculator.py`
- `monitoring-observability/scripts/health_check_validator.py`

Reusable templates/assets:

- `monitoring-observability/assets/templates/prometheus-alerts/webapp-alerts.yml`
- `monitoring-observability/assets/templates/prometheus-alerts/kubernetes-alerts.yml`
- `monitoring-observability/assets/templates/runbooks/incident-runbook-template.md`

Scope filtering for this interviewer:

- prioritize EFK and Prometheus/Grafana
- keep Loki/Datadog content only for comparison questions, not primary interview depth

---

## Planned Skill Package

Create folder:

- `devops-observability-interviewer/`

Planned files:

- `devops-observability-interviewer/SKILL.md`
- `devops-observability-interviewer/repo-intake.md`
- `devops-observability-interviewer/question-templates.md`
- `devops-observability-interviewer/rubrics.md`
- `devops-observability-interviewer/example-transcript.md`
- `devops-observability-interviewer/checklists/monitoring.md`
- `devops-observability-interviewer/checklists/logging-efk.md`
- `devops-observability-interviewer/checklists/best-practices.md`
- `devops-observability-interviewer/checklists/technology-comparisons.md`
- `devops-observability-interviewer/agents/openai.yaml`

---

## Implementation Phases

### Phase 1: Scaffold the new interviewer skill

1. Create `devops-observability-interviewer/` using one existing interviewer skill as baseline.
2. Keep all required files from the interviewer contract.
3. Rename and rewrite domain-specific assets for observability-first behavior.

### Phase 2: Author `SKILL.md` for observability-first interviewing

1. Set frontmatter:
   - `name: devops-observability-interviewer`
   - description that clearly triggers on observability interview requests
2. Keep standard interview mechanics (phases, question loop, scoring, debrief).
3. Define rounds with observability priority:
   - observability architecture and telemetry strategy
   - Prometheus (instrumentation, scrape model, recording rules, cardinality)
   - Grafana and alerting (dashboards, Alertmanager routing, burn-rate rules)
   - EFK logging pipeline (Fluent Bit/Fluentd, Elasticsearch, Kibana)
   - SLO/error budget and incident response
   - security/compliance and cost optimization of observability stack
4. Keep read-only safety policy and prohibited command list.

### Phase 3: Build `repo-intake.md` for observability artifact discovery

Capture and classify:

1. Prometheus stack artifacts:
   - `Prometheus`, `ServiceMonitor`, `PodMonitor`, `PrometheusRule`, `AlertmanagerConfig`
   - scrape configs, recording rules, retention, remote write/federation
2. Grafana artifacts:
   - dashboards (JSON/provisioning), data sources, folder strategy, annotations
3. EFK artifacts:
   - Fluent Bit/Fluentd DaemonSets and parser/filter/output configs
   - Elasticsearch resources/index templates/ILM policies
   - Kibana saved objects, index patterns, alerting artifacts (if present)
4. SLO/runbook artifacts:
   - SLI/SLO definitions, burn-rate rules, runbooks, incident docs
5. Security/cost signals:
   - PII redaction controls, retention windows, index lifecycle and storage cost controls

### Phase 4: Create scoring checklists

1. `checklists/monitoring.md`
   - golden signals, RED/USE, metric type correctness, label cardinality, recording rules
2. `checklists/logging-efk.md`
   - structured logs, parser quality, pipeline reliability, index lifecycle, Kibana query workflow
3. `checklists/best-practices.md`
   - four pillars: security, scalability, availability, cost optimization for observability stack
4. `checklists/technology-comparisons.md`
   - EFK vs Loki
   - Prometheus/Grafana vs Datadog
   - recording rules vs raw expensive queries
   - Fluent Bit vs Fluentd
   - Elasticsearch ILM tiers and retention tradeoffs

### Phase 5: Build `question-templates.md`

For each round include mandatory categories:

1. What breaks if...
2. How would you debug...
3. Tradeoff
4. Technology comparison (formal definitions required)
5. Best practice pillar

EFK-specific prompt inventory must include:

- mapping explosion and high-cardinality fields
- ingest pipeline/parser failures
- shard/segment pressure and query latency
- missing logs in Kibana due to timestamp/index pattern mismatch
- retention/ILM misconfiguration impacts

Prometheus/Grafana-specific prompt inventory must include:

- wrong histogram usage and broken p95 interpretation
- scrape gaps and stale metrics
- alert fatigue and missing runbook links
- burn-rate alert tuning and false positives
- dashboard anti-patterns (inconsistent windows, no context panels)

### Phase 6: Author `rubrics.md`

Add universal scoring + domain rubric extensions:

1. Observability architecture and telemetry design
2. Prometheus and PromQL depth
3. Grafana dashboards and alerting quality
4. EFK logging operations and Kibana debugging
5. SLO/error budgets and incident response discipline
6. Security/compliance of telemetry data
7. Cost/performance optimization of observability platform

### Phase 7: Create `example-transcript.md`

Must include:

1. repo map excerpt
2. one Prometheus/Grafana scored exchange
3. one EFK/Kibana scored exchange
4. one debrief excerpt with drill plan

### Phase 8: Final metadata

Create `agents/openai.yaml` with:

- display name: `DevOps Observability Interviewer`
- short description: repo-aware observability interview focused on EFK + Prometheus/Grafana
- default prompt using `$devops-observability-interviewer`

### Phase 9: Validate and iterate

1. Run structural validation:
   - `python3 /Users/romansmirnov/.codex/skills/.system/skill-creator/scripts/quick_validate.py devops-observability-interviewer`
2. Run content sanity greps (phases, rounds, score blocks).
3. Dry-run on:
   - observability-rich repo
   - monitoring-only repo with weak logging artifacts
   - sparse repo for generic fallback behavior

---

## Exit Criteria

- Skill triggers on observability interview requests.
- Repo map reliably identifies EFK and Prometheus/Grafana artifacts.
- Interview questions are repo-grounded when artifacts exist.
- Every question receives score + tech justification + model answer.
- Debrief contains score summary, pillar coverage, strengths/gaps, and actionable drill plan.
- Safety remains read-only with explicit prohibition of mutating commands.
