# Best Practice Pillars Checklist

> Use this checklist when asking "Best Practice Pillar" questions and when scoring answers.
> Every interview session must cover all four pillars. Track which pillars have been addressed
> and prioritize uncovered ones in later rounds.

---

## Pillar 1: Security

### Identity and Access

- [ ] Least privilege for workload and automation identities
- [ ] Role assumption with temporary credentials (avoid long-lived static keys)
- [ ] IRSA for all Kubernetes-to-AWS access (no node-level roles for workloads)
- [ ] IRSA trust policies scoped to specific namespace + service account
- [ ] IAM policies follow least privilege (specific resource ARNs, not `*`)
- [ ] Trust policies scoped to intended principals and conditions
- [ ] Separation of duties across deployment, operations, and admin paths
- [ ] Break-glass access is controlled, auditable, and time-bounded
- [ ] EKS API server authentication via AWS IAM (aws-auth ConfigMap or EKS access entries)
- [ ] ArgoCD RBAC configured (projects, roles, not default admin for everything)
- [ ] Jenkins credentials stored in Secrets Manager, not in Jenkins config files

### Secrets and Key Management

- [ ] Secrets stored in AWS Secrets Manager (not in Git, not in Kubernetes Secrets directly)
- [ ] External Secrets Operator syncs secrets into Kubernetes
- [ ] `.gitignore` covers `.env`, `creds/`, `secrets/`, `*.pem`, `*.key`
- [ ] Terraform `sensitive = true` on all secret-containing variables/outputs
- [ ] Secret values set manually (not in Terraform state)
- [ ] Secret rotation plan documented and testable
- [ ] KMS key ownership and access boundaries are defined
- [ ] Encryption at rest and in transit is consistently applied
- [ ] Secret access is scoped by identity and environment

### Network and Edge Security

- [ ] Private subnets for workloads (EKS nodes in private subnets)
- [ ] Public subnets only for load balancers and NAT gateway
- [ ] Security groups are restrictive (no 0.0.0.0/0 ingress unless justified)
- [ ] Kubernetes network policies (if implemented)
- [ ] CloudFront OAC for private S3 access
- [ ] HTTPS everywhere (ACM certificates, TLS termination)
- [ ] Edge protections (WAF, origin hardening) where needed
- [ ] East-west segmentation strategy exists for critical workloads
- [ ] Data egress paths are controlled and observable

### Container Security

- [ ] Non-root user in Dockerfiles (`appuser`, UID 1000)
- [ ] Multi-stage builds to minimize attack surface
- [ ] ECR scan-on-push for vulnerability detection
- [ ] Base image pinned to specific version (not `latest`)
- [ ] Pod Security Standards (if implemented)
- [ ] Image pull policy set appropriately

### Supply Chain

- [ ] Container images from trusted registries only (ECR)
- [ ] Terraform provider versions pinned
- [ ] Helm chart versions pinned
- [ ] ECR lifecycle policy limits stored images
- [ ] CI pipeline includes security scanning stage
- [ ] SBOM generation and software supply chain security awareness

### Audit and Compliance

- [ ] Git history as audit trail for all infrastructure changes
- [ ] ArgoCD sync history for deployment audit
- [ ] AWS CloudTrail enabled
- [ ] Terraform state versioning in S3
- [ ] Identity and configuration changes are auditable
- [ ] Retention and access policies exist for audit logs
- [ ] Deployment events are traceable to source control and principals

### Telemetry Data Protection

- [ ] Sensitive fields (passwords, tokens, keys, PII) are redacted before storage
- [ ] Secret values are never logged intentionally
- [ ] Sensitive logs are access-scoped by role/team
- [ ] Least-privilege access for Grafana/Kibana/Elasticsearch/Prometheus
- [ ] Separation of admin vs read-only roles for observability tools

---

## Pillar 2: Scalability

### Compute and Runtime

- [ ] EKS node group scaling: current min/max/desired configuration
- [ ] Cluster Autoscaler or Karpenter (candidate should identify if missing)
- [ ] Horizontal Pod Autoscaler (HPA) for application workloads
- [ ] Instance type selection justified (burstable vs general-purpose tradeoff)
- [ ] Autoscaling strategy maps to actual demand signals
- [ ] Capacity limits and quota risks are known and monitored
- [ ] Scaling path is tested for expected growth

### Network and Throughput

- [ ] VPC CIDR sizing supports projected growth
- [ ] Subnet sizing (IPs per subnet vs expected Pod count)
- [ ] Pod IP capacity: VPC-CNI ENI limits per instance type
- [ ] Prefix delegation for higher pod density
- [ ] NAT gateway throughput limits understood
- [ ] DNS and edge strategy can handle scale events
- [ ] Bottlenecks are identified before they become outages

### Storage and Data

- [ ] EBS CSI driver for dynamic volume provisioning
- [ ] S3 for static assets (virtually unlimited)
- [ ] ECR lifecycle policy (keep last N images)
- [ ] CloudFront caching reduces origin load
- [ ] Data service capacity model is explicit (throughput/latency/partitioning)
- [ ] Storage tiering and retention model scales sustainably

### Application and Pipeline

- [ ] Stateless application design where possible
- [ ] CDN offloads static asset serving
- [ ] Jenkins scaling (single instance = SPOF — candidate should identify)
- [ ] Parallel test stages in CI pipeline
- [ ] Docker layer caching for faster builds
- [ ] ArgoCD handles deployment at cluster scale

### Observability Stack Scaling

- [ ] Cardinality control strategy exists and is enforced
- [ ] Expensive repeated queries are moved to recording rules
- [ ] Collector architecture can handle burst traffic
- [ ] Buffer/retry/backpressure behavior is understood
- [ ] Index mapping growth is controlled (Elasticsearch)
- [ ] Shard sizing and rollover strategy are documented
- [ ] Dashboard query patterns are performance-aware

---

## Pillar 3: Availability

### Control Plane

- [ ] EKS managed control plane: multi-AZ by default
- [ ] etcd: managed by AWS, replicated across AZs
- [ ] API server: behind AWS-managed NLB

### Data Plane and Runtime

- [ ] Nodes spread across multiple AZs (uneven distribution = risk)
- [ ] Pod Disruption Budgets (if implemented)
- [ ] Liveness and readiness probes configured
- [ ] Graceful shutdown and preStop hooks
- [ ] Multi-AZ (or equivalent) strategy exists for critical paths
- [ ] Load balancing and health-check strategy is explicit
- [ ] Dependency failures are isolated to reduce blast radius

### Networking

- [ ] Single NAT gateway: SPOF for outbound traffic from private subnets
- [ ] Route 53: 100% SLA, globally distributed
- [ ] CloudFront: globally distributed, multi-AZ origins
- [ ] cert-manager: certificate renewal before expiry

### Application

- [ ] Multiple replicas (if configured)
- [ ] Health checks: `/health` endpoint
- [ ] Rolling update strategy (default for Deployments)
- [ ] ArgoCD self-heal: auto-recovers drifted resources

### Recovery and Resilience

- [ ] Terraform state versioned in S3 (can restore previous state)
- [ ] ECR images retained (lifecycle policy)
- [ ] Bootstrap stack separates critical resources from destroyable infra
- [ ] `prevent_destroy` on critical resources (ECR, Secrets Manager, Route 53 zone)
- [ ] GitOps repo is the single source of truth for cluster state
- [ ] Rollback path is fast, safe, and tested
- [ ] Backup strategy includes restore validation
- [ ] RTO/RPO targets are defined for critical systems
- [ ] Incident response process is structured and repeatable

### Delivery Safety

- [ ] Changes are reviewed and validated before production
- [ ] Drift detection and reconciliation controls are active
- [ ] Emergency changes preserve traceability and safe recovery

### Detectability and Signal Reliability

- [ ] Core user-impact alerts exist and are actionable
- [ ] Alert noise is managed (grouping/inhibition/for clauses)
- [ ] Signal quality is sufficient for incident triage
- [ ] Runbooks exist and are mapped to critical alerts
- [ ] SLO/error-budget alerts guide prioritization

### Single Points of Failure (SPOFs)

Candidate should be able to identify these:
1. **Single NAT gateway** — outbound internet for private subnets
2. **Jenkins EC2 instance** — CI/CD pipeline
3. **GitLab/GitHub repo** — source code and GitOps state
4. **MongoDB** — application data (external dependency)
5. **ArgoCD** — deployment pipeline (if down, no deploys but running apps unaffected)
6. **Single Prometheus instance** — metrics collection gap during failure
7. **Single Elasticsearch node** — log search unavailable during failure

---

## Pillar 4: Cost Optimization

### Spend Visibility

- [ ] Cost allocation tags cover owner/environment/service dimensions
- [ ] Cost trends are reviewed regularly with ownership accountability
- [ ] Major service cost drivers are known and tracked
- [ ] Budget alerts and anomaly alerts route to clear owners
- [ ] Monthly cost report cadence is defined and followed

### Compute Costs

- [ ] Instance type justified (AMD/Graviton tradeoff, burstable vs general-purpose)
- [ ] Auto-scaling vs fixed-size node analysis
- [ ] Candidate knows: Graviton instances save ~20%
- [ ] Spot instances evaluated for fault-tolerant workloads
- [ ] Right-sizing based on actual utilization data

### Network Costs

- [ ] NAT gateway cost understood (base + per-GB processing)
- [ ] CloudFront PriceClass selection justified
- [ ] Data transfer: CDN → internet is cheaper than direct origin
- [ ] VPC endpoints could eliminate NAT costs for AWS API calls
- [ ] Egress paths optimized (CDN/endpoints/caching)

### Storage Costs

- [ ] S3: lifecycle and tiering strategy
- [ ] ECR: lifecycle policy limits stored images
- [ ] EBS: volume type optimization (gp2 → gp3)
- [ ] Observability: ILM tiering for log retention (hot/warm/cold/delete)
- [ ] Idle/unused resources identified and removed (EBS, snapshots, EIPs)

### Operational Costs

- [ ] Secrets Manager per-secret cost known
- [ ] Jenkins: compute cost separate from K8s cluster
- [ ] Observability stack: self-hosted vs managed tradeoff understood
- [ ] Dashboard sprawl and alert sprawl controlled

### Purchase Strategy

- [ ] Spot/reserved/savings options evaluated where appropriate
- [ ] Commitment purchases (RI/SP) based on stable baseline evidence
- [ ] Cost optimization does not violate reliability or security requirements
- [ ] Optimization actions prioritized by risk-adjusted ROI
- [ ] Strategic initiatives (Spot, Graviton, chargeback) have staged rollout plans

### Cost Estimation Methodology

Candidate should be able to approximate monthly costs by service category. Do not memorize exact dollar amounts — demonstrate the ability to:
1. Identify the major cost drivers (EKS control plane, nodes, NAT, storage, DNS)
2. Use AWS pricing calculator or CLI to estimate
3. Compare alternatives (e.g., NAT gateway vs NAT instance, t3a vs t4g vs spot)
4. Propose optimizations ordered by effort vs savings ratio

---

## Scoring Application

When evaluating best-practice answers:

1. Check whether candidate maps controls to pillar outcomes.
2. Require practical implementation detail, not slogans.
3. Penalize unsafe shortcuts (e.g., disabling alerts, broad IAM, skipping scanning).
4. Use `rubrics.md` to map depth and correctness to score.

| Coverage of relevant items | Suggested Range |
|----------------------------|-----------------|
| 0-20% | 1-25 (Awareness) |
| 20-50% | 26-50 (Competent) |
| 50-80% | 51-75 (Proficient) |
| 80%+ with proactive depth | 76-100 (Expert) |
