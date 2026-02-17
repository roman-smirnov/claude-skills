# Observability — Interview Module

## Enhancement Instructions

- Evaluate **metric design** maturity: Golden Signals, RED/USE methods, counter vs gauge vs histogram correctness, PromQL semantics, cardinality awareness.
- Probe **alert quality** discipline: actionable alerts, severity routing, `for` clauses, runbook links, grouping/inhibition, and the difference between symptom and cause alerts.
- Evaluate **structured logging** depth: EFK pipeline understanding, collector pipeline (Fluent Bit/Fluentd), Elasticsearch index management, ILM lifecycle, and Kibana investigation workflows.
- Score higher when the candidate connects observability to **SLO/error budget** operations: burn-rate alerting, multi-window strategies, and error budget policy decisions.
- Probe **AWS-native observability** integration: CloudWatch metrics and alarms, Container Insights, and when to use CloudWatch vs Prometheus/Grafana.

---

## Evaluation Checklist

### Observability Foundations (Mandatory)

- [ ] Clear distinction between observability and basic monitoring
- [ ] End-to-end telemetry path understanding (emit → collect → store → query → alert)
- [ ] Cross-signal reasoning (metrics + logs + traces) during incidents
- [ ] User-impact first mindset (symptoms before internal causes)

### Metric Design Fundamentals

#### Golden Signals and RED/USE
- [ ] Golden Signals defined correctly: latency, traffic, errors, saturation
- [ ] RED method used for request-driven services (Rate, Errors, Duration)
- [ ] USE method used for infrastructure components (Utilization, Saturation, Errors)
- [ ] Signals are linked to business/user impact

#### Metric Type Correctness
- [ ] Counter vs Gauge vs Histogram explained correctly
- [ ] Counter queries use `rate()` or `increase()`
- [ ] Histogram queries use proper `histogram_quantile()` patterns
- [ ] Candidate can explain why summaries are less flexible than histograms

#### Naming and Labeling
- [ ] Prometheus naming conventions understood
- [ ] Label taxonomy is consistent (service/env/region/team)
- [ ] High-cardinality labels are avoided (user_id, request_id as metric labels)
- [ ] Candidate can explain cardinality impact and mitigation techniques

### Prometheus Collection and Reliability

#### Scrape and Discovery
- [ ] Candidate can explain target discovery flow (`ServiceMonitor`/`PodMonitor`/static)
- [ ] Scrape interval/timeout tradeoff is understood
- [ ] Can debug missing targets via service discovery and scrape status
- [ ] External labels and relabeling implications are understood

#### Rule Design
- [ ] Recording rules used for expensive recurring queries
- [ ] Alert rules have clear and testable expressions
- [ ] Rule naming is consistent and meaningful
- [ ] Evaluation interval and performance implications understood

#### Scale and Resilience
- [ ] Can explain single-instance limitations
- [ ] Federation/remote-write tradeoffs are understood
- [ ] Retention and storage planning is discussed with constraints
- [ ] Alerting continuity strategy exists for partial scrape/data loss

### Grafana Dashboard Quality

#### Information Architecture
- [ ] Top-level health panels (request rate, error rate, latency, saturation)
- [ ] Supporting context panels included (resources, dependencies, deployment markers)
- [ ] Time windows are consistent across related panels
- [ ] Dashboard ownership and naming conventions are clear

#### Query Quality
- [ ] Dashboard queries avoid unnecessary high-cost scans
- [ ] Recording rules used where dashboard query cost is high
- [ ] Refresh interval impact on backend cost/performance understood

#### Operability
- [ ] Dashboards support incident triage, not only executive reporting
- [ ] Links to logs/runbooks are present
- [ ] Can prioritize which panels to check first during incidents

### Alert Quality and Alertmanager Operations

#### Alert Rule Design
- [ ] Alerts are actionable (clear response expected)
- [ ] Alerts focus on symptoms/user impact where appropriate
- [ ] Severity labels are present and rational
- [ ] `for` clauses are used to reduce flapping
- [ ] Runbook links are included for operational alerts
- [ ] Summaries/descriptions are clear and contextual
- [ ] Candidate avoids paging for non-actionable informational events

#### Alert Noise Control
- [ ] Can explain grouping and inhibition concepts
- [ ] Duplicate/noisy alert patterns are identified and mitigated
- [ ] Alert fatigue prevention is explicitly discussed

#### Routing and Escalation
- [ ] Severity-based routing understood
- [ ] Team/service ownership mapping is clear
- [ ] Escalation path and fallback receiver logic understood

### SLO and Error Budget Operations

#### Definitions and Policy
- [ ] SLI/SLO/SLA defined correctly
- [ ] Error budget defined as allowable unreliability tied to SLO target
- [ ] Can explain how error budget affects release decisions
- [ ] SLO target realism discussed

#### SLO Reporting and Review
- [ ] Candidate can describe periodic SLO review cadence
- [ ] Incident/postmortem learnings feed SLO/rule refinement
- [ ] Candidate avoids creating too many low-value SLOs

#### Burn Rate and Alerting
- [ ] Burn rate concept explained correctly
- [ ] Multi-window burn-rate alerting strategy is understood
- [ ] Can interpret fast-burn vs slow-burn implications
- [ ] Can connect burn events to incident priorities/actions

### EFK Logging Pipeline

#### Structured Logging Anti-Patterns
- [ ] Plaintext PII/secrets in logs
- [ ] Inconsistent formats between services
- [ ] Multi-destination uncontrolled logging without purpose
- [ ] Excessive debug volume in production by default

#### Structured Logging Quality (Mandatory)
- [ ] Timestamp in consistent machine-parseable format (ISO 8601)
- [ ] Log level normalized (`DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`)
- [ ] Human-readable message
- [ ] Service/component identity field
- [ ] Environment field (`dev`, `staging`, `prod`)
- [ ] Correlation field (`request_id` or `trace_id`)

#### Collector Pipeline (Fluent Bit / Fluentd)
- [ ] Input source and scope (node logs, container logs, app logs)
- [ ] Parser behavior and failure handling
- [ ] Filter/enrichment/redaction stages
- [ ] Output routing to Elasticsearch
- [ ] Retry/buffering/backpressure behavior

#### Collector Operational Reliability
- [ ] Candidate can diagnose dropped logs vs delayed logs
- [ ] Candidate can explain collector resource constraints and tuning
- [ ] Candidate can identify parser mismatch and timestamp parse failures
- [ ] Candidate can explain how to verify collector health and throughput

#### Elasticsearch Design and Operations
- [ ] Index naming strategy is consistent (service/environment/date pattern)
- [ ] Mapping strategy is controlled (dynamic mapping risk understood)
- [ ] ILM phases (hot/warm/cold/delete) are understood
- [ ] Retention windows justified by use case/compliance
- [ ] Rollover criteria and shard sizing understood

#### Elasticsearch Query Performance
- [ ] Candidate understands expensive wildcard/regex query risks
- [ ] Candidate can diagnose slow query behavior via index/shard/query pattern
- [ ] Candidate can differentiate indexing throughput issues from query latency issues

#### Kibana Investigation Workflow
1. [ ] Confirm logs exist at source (`kubectl logs`, app stdout/stderr)
2. [ ] Check collector ingestion status and parser errors
3. [ ] Validate Elasticsearch index presence and document counts
4. [ ] Verify timestamp field and timezone assumptions
5. [ ] Verify Kibana data view/index pattern and query filters
6. [ ] Confirm query time range and field filters are not excluding results

#### Kibana Expected Competence Signals
- [ ] Candidate can explain why logs may exist in ES but not appear in Kibana
- [ ] Candidate can isolate data latency vs true data loss
- [ ] Candidate can explain saved search/dashboard filter inheritance pitfalls

#### Common EFK Failure Patterns
- [ ] Missing logs for one namespace/service only
- [ ] Timestamp mismatch causing logs to appear outside expected window
- [ ] Parser failures after format/schema change
- [ ] High index growth due to cardinality explosion
- [ ] Shard imbalance causing uneven query performance
- [ ] Collector restart loops causing ingestion gaps
- [ ] Elasticsearch read-only index due to disk watermarks

### Incident Response Discipline

1. [ ] Detect and classify impact/scope
2. [ ] Triage with highest-signal dashboards/alerts
3. [ ] Investigate with cross-signal evidence
4. [ ] Mitigate with risk-aware action
5. [ ] Verify recovery and monitor recurrence risk
6. [ ] Record follow-up prevention actions

#### Incident Response Evaluation Signals
- [ ] Diagnostic order is explicit and justified
- [ ] Candidate distinguishes symptom from root cause
- [ ] Candidate states verification criteria before declaring resolved
- [ ] Candidate proposes prevention changes, not only immediate fixes

### AWS-Native Observability (NEW)

- [ ] Understands CloudWatch metrics, alarms, and composite alarms
- [ ] Can explain Container Insights for EKS monitoring (node, pod, container metrics)
- [ ] Knows when CloudWatch is sufficient vs when Prometheus/Grafana is needed
- [ ] Understands CloudWatch Logs integration with EFK pipeline

### Long-Term Storage (NEW)

- [ ] Understands Thanos for long-term Prometheus metrics storage and global querying
- [ ] Knows Cortex/VictoriaMetrics as alternatives to Thanos
- [ ] Can explain when single Prometheus retention is insufficient
- [ ] Understands remote-write patterns for multi-cluster federation

### Cost and Scalability Controls for Logging

- [ ] Candidate can identify top log cost drivers (volume, retention, indexing strategy)
- [ ] Sampling/aggregation strategy is discussed for high-volume logs
- [ ] Candidate can justify hot vs warm/cold storage policy
- [ ] Candidate can explain tradeoff between diagnostic depth and storage spend

### Security and Compliance for Observability

- [ ] Treats logs as sensitive data store, not debug dump
- [ ] PII/secret redaction strategy is explicit
- [ ] Access controls for Kibana/Elasticsearch/Grafana are least-privilege
- [ ] Auditability of access and config changes is discussed

### Diagnostic Commands

```bash
# Prometheus and Rules
promtool check rules /path/to/rules.yml
curl -s http://prometheus:9090/api/v1/targets
curl -s http://prometheus:9090/api/v1/rules
curl -s http://alertmanager:9093/api/v2/alerts

# Kubernetes Observability Components
kubectl get pods -n monitoring
kubectl describe pod <pod> -n monitoring
kubectl logs <pod> -n monitoring

# Elasticsearch / Logging
kubectl get pods -n logging
kubectl logs <collector-pod> -n logging
curl -s http://elasticsearch:9200/_cat/indices?v
curl -s http://elasticsearch:9200/_cat/shards?v
curl -s http://elasticsearch:9200/<index>/_mapping

# CloudWatch (read-only)
aws cloudwatch describe-alarms
aws cloudwatch get-metric-data --metric-data-queries file://queries.json --start-time <iso> --end-time <iso>
aws logs filter-log-events --log-group-name <group> --start-time <ms>
```

### Scoring Application

| Coverage | Score |
|----------|-------|
| 0-20% of relevant items | 1–25 (Awareness) |
| 20-50% | 26–50 (Competent) |
| 50-80% | 51–75 (Proficient) |
| 80%+ with proactive depth | 76–100 (Expert) |

---

## Question Templates

### What Breaks If...

1. "Your telemetry path depends on `{resource}` in `{path}`. It fails silently for 30 minutes. What operational blind spots appear first?"
2. "A key counter metric `{metric}` resets frequently and queries don't use `rate()` or `increase()`. What goes wrong in alerting and SLO calculations?"
3. "Histogram buckets for request latency are misconfigured in `{path}`. What invalid conclusions can p95 dashboards produce?"
4. "Alert rules in `{path}` omit runbook links and `for` clauses. What operational failure modes follow?"
5. "Fluent Bit parser config in `{path}` changes and timestamp parsing fails. How does this surface in Kibana and incident timelines?"
6. "Elasticsearch index mapping for `{index}` allows uncontrolled dynamic fields. What scalability and reliability issues emerge?"
7. "ILM policy in `{path}` is misconfigured and hot indices never roll over. What failures do you expect first?"

### How Would You Debug...

8. "The team reports elevated latency, but dashboards look normal. Walk me through your cross-signal debugging order and what each step validates."
9. "A target is missing from Prometheus unexpectedly. Walk through your diagnostic order from service discovery to scrape status to relabeling."
10. "PromQL query `{query}` returns empty series in production but not staging. How do you isolate label mismatch vs data absence?"
11. "Alert fires for high error rate, but dashboard shows low error rate. What query-level consistency checks do you run first?"
12. "Logs from service `{service}` are missing in Kibana. Walk your debug sequence from pod logs to collector pipeline to Elasticsearch index/data view."
13. "Kibana query latency spikes for common searches. How do you isolate index pattern, shard strategy, and query anti-pattern causes?"
14. "Observability bill spikes 2x month-over-month. Walk through your step-by-step cost attribution approach."

### Tradeoff

15. "Scrape every 15s vs every 60s: what fidelity vs cost tradeoff is appropriate here?"
16. "Single Prometheus server vs sharded/federated model: when do you switch and why?"
17. "Threshold-based alerts vs SLO burn-rate alerts: what false-positive and false-negative tradeoffs matter most?"
18. "Fluent Bit vs Fluentd for collection/processing in this environment: what complexity and throughput tradeoffs matter?"
19. "Long retention in Elasticsearch hot tier vs tiered ILM strategy: what cost/performance risk tradeoff are you accepting?"
20. "Self-hosted EFK + Prometheus/Grafana vs managed observability platform: what operational burden vs cost tradeoff is right here?"

### Technology Comparison

21. "[DEFINE] What is observability vs monitoring? How should this project use both terms precisely?"
22. "[DEFINE] Counter vs Gauge vs Histogram: when should each be used in this project?"
23. "[DEFINE] Why Prometheus + Grafana here instead of Datadog or CloudWatch-only monitoring?"
24. "[DEFINE] What is EFK architecture and what role does each component play in this project?"
25. "[DEFINE] Elasticsearch vs Loki for log storage/query: which tradeoffs drove your choice?"
26. "[DEFINE] SLI vs SLO vs SLA and how they should map to this project."
27. "[DEFINE] Burn rate vs raw error rate alerting: why is burn rate often superior operationally?"

### Best Practice Pillar

28. **(Availability)** "Which telemetry components are single points of failure in this architecture, and what would you harden first?"
29. **(Scalability)** "How do you prevent high-cardinality labels from destabilizing Prometheus as this system scales?"
30. **(Cost)** "Name the top three cost drivers in this repo's observability model and rank optimization priority."
31. **(Security)** "Which metrics should never expose PII or secret values, and how do you enforce that?"
32. **(Availability)** "How do you design dashboards to accelerate MTTR during a SEV incident?"
33. **(Cost)** "Which log categories should be sampled, retained, or dropped first to control cost safely?"
34. **(Scalability)** "If metric volume and log volume grow 10x, what component saturates first and why?"
35. **(Security)** "How do you enforce redaction and access boundaries for sensitive log data?"

### AWS-Native Observability (NEW)

36. "Your EKS cluster has Container Insights enabled. What metrics does it provide that Prometheus does not? When would you use Container Insights vs a self-managed Prometheus stack?"
37. "You need to integrate CloudWatch alarms with your existing Alertmanager routing. What are the approaches and tradeoffs?"

### Long-Term Storage (NEW)

38. (generic) "Your Prometheus retention is 15 days but you need 6 months of historical metrics for capacity planning. Compare Thanos, Cortex, and VictoriaMetrics for long-term storage."

### Bonus Pool

39. "[DEFINE] Compare RED and USE methods. Which workloads in this repo should use each and why?"
40. "[DEFINE] Golden Signals vs SLI/SLO framing: where do they overlap and where do they differ operationally?"
41. "How do sync windows and maintenance periods interact with observability alerting?"
42. "[DEFINE] What is Alertmanager routing and inhibition? Why not rely on raw Prometheus alert firing alone?"
43. "[DEFINE] Hot/warm/cold index lifecycle tiers and why they matter for Elasticsearch economics."
44. "Prometheus CPU and memory usage spikes after a deployment. How do you determine if rules, cardinality, or scrape targets caused it?"
