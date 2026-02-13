# Best Practice Pillars Checklist

> Use this checklist when asking "Best Practice Pillar" questions and when scoring answers.
> Every interview session must cover all four pillars. Track which pillars have been addressed
> and prioritize uncovered ones in later rounds.

---

## Pillar 1: Security

### Authentication & Authorization
- [ ] IRSA for all Kubernetes-to-AWS access (no node-level roles for workloads)
- [ ] IRSA trust policies scoped to specific namespace + service account
- [ ] IAM policies follow least privilege (specific resource ARNs, not `*`)
- [ ] EKS API server authentication via AWS IAM (aws-auth ConfigMap or EKS access entries)
- [ ] ArgoCD RBAC configured (projects, roles, not default admin for everything)
- [ ] Jenkins credentials stored in Secrets Manager, not in Jenkins config files

### Secrets Management
- [ ] Secrets stored in AWS Secrets Manager (not in Git, not in Kubernetes Secrets directly)
- [ ] External Secrets Operator syncs secrets into Kubernetes
- [ ] `.gitignore` covers `.env`, `creds/`, `secrets/`, `*.pem`, `*.key`
- [ ] Terraform `sensitive = true` on all secret-containing variables/outputs
- [ ] Secret values set manually (not in Terraform state)
- [ ] Secret rotation plan documented

### Network Security
- [ ] Private subnets for workloads (EKS nodes in private subnets)
- [ ] Public subnets only for load balancers and NAT gateway
- [ ] Security groups are restrictive (no 0.0.0.0/0 ingress unless justified)
- [ ] Kubernetes network policies (if implemented)
- [ ] CloudFront OAC for private S3 access
- [ ] HTTPS everywhere (ACM certificates, TLS termination)

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
- [ ] CI pipeline includes security scanning stage (or should)

### Audit & Compliance
- [ ] Git history as audit trail for all infrastructure changes
- [ ] ArgoCD sync history for deployment audit
- [ ] AWS CloudTrail enabled (or should be)
- [ ] Terraform state versioning in S3

---

## Pillar 2: Scalability

### Compute Scaling
- [ ] EKS node group scaling: current min/max/desired (3/3/3 — fixed)
- [ ] Cluster Autoscaler or Karpenter (not implemented — candidate should identify this gap)
- [ ] Horizontal Pod Autoscaler (HPA) for application workloads
- [ ] Instance type selection justified (t3a.medium — burstable, AMD, cost-efficient)

### Network Scaling
- [ ] VPC CIDR sizing: /16 (65,536 IPs total)
- [ ] Subnet sizing: /24 per subnet (256 IPs each)
- [ ] Pod IP capacity: VPC-CNI ENI limits per instance type
- [ ] Prefix delegation for higher pod density (not enabled — candidate should know this)
- [ ] NAT gateway throughput limits (45 Gbps, 55,000 simultaneous connections)

### Storage Scaling
- [ ] EBS CSI driver for dynamic volume provisioning
- [ ] S3 for static assets (virtually unlimited)
- [ ] ECR lifecycle policy (keep last 10 images)
- [ ] CloudFront caching reduces origin load

### Application Scaling
- [ ] Stateless application design (backend is stateless, MongoDB is the state)
- [ ] MongoDB scaling strategy (not managed here — external dependency)
- [ ] CDN offloads static asset serving
- [ ] API caching strategy (CloudFront no-cache for API — candidate should discuss when to cache)

### Pipeline Scaling
- [ ] Jenkins single instance (SPOF — candidate should identify)
- [ ] Parallel test stages in Jenkinsfile
- [ ] Docker layer caching for faster builds
- [ ] ArgoCD handles deployment at cluster scale

---

## Pillar 3: Availability

### Control Plane Availability
- [ ] EKS managed control plane: multi-AZ by default
- [ ] etcd: managed by AWS, replicated across AZs
- [ ] API server: behind AWS-managed NLB

### Data Plane Availability
- [ ] Nodes spread across 2 AZs (candidate should note: 3 nodes / 2 AZs means uneven distribution)
- [ ] Pod Disruption Budgets (if implemented)
- [ ] Liveness and readiness probes (configured in Helm charts in GitOps repo)
- [ ] Graceful shutdown and preStop hooks

### Networking Availability
- [ ] Single NAT gateway: SPOF for outbound traffic from private subnets
- [ ] Route 53: 100% SLA, globally distributed
- [ ] CloudFront: globally distributed, multi-AZ origins
- [ ] cert-manager: certificate renewal before expiry

### Application Availability
- [ ] Multiple replicas (if configured — depends on GitOps repo values)
- [ ] Health checks: `/health` endpoint
- [ ] Rolling update strategy (default for Deployments)
- [ ] ArgoCD self-heal: auto-recovers drifted resources

### Disaster Recovery
- [ ] Terraform state versioned in S3 (can restore previous state)
- [ ] ECR images retained (lifecycle policy keeps 10)
- [ ] Bootstrap stack separates critical resources from destroyable infra
- [ ] `prevent_destroy` on ECR, Secrets Manager, Route 53 zone
- [ ] GitOps repo is the single source of truth for cluster state
- [ ] DNS TTL considerations for failover

### Single Points of Failure (SPOFs)
Candidate should be able to identify these:
1. **Single NAT gateway** — outbound internet for private subnets
2. **Jenkins EC2 instance** — CI/CD pipeline
3. **GitLab repo** — source code and GitOps state
4. **MongoDB** — application data (external dependency)
5. **ArgoCD** — deployment pipeline (if down, no deploys but running apps unaffected)

---

## Pillar 4: Cost Optimization

### Compute Costs
- [ ] t3a.medium: AMD-based, cheaper than t3.medium (~10% savings)
- [ ] Fixed 3 nodes (no auto-scaling — saves Cluster Autoscaler overhead but may over-provision)
- [ ] EKS control plane: $0.10/hour ($73/month)
- [ ] Candidate should know: Graviton (t4g) would save additional ~20%
- [ ] Spot instances not used (candidate should discuss when spot is appropriate for EKS)

### Network Costs
- [ ] Single NAT gateway: ~$32/month + data processing ($0.045/GB)
- [ ] CloudFront PriceClass_200: excludes South America, Australia (cheapest non-minimal option)
- [ ] Data transfer: CloudFront → internet is cheaper than direct EC2 → internet
- [ ] VPC endpoints could eliminate NAT costs for AWS API calls (not implemented)

### Storage Costs
- [ ] S3: negligible for static assets and Terraform state
- [ ] ECR: lifecycle policy limits image storage
- [ ] EBS: 20 GiB per node (60 GiB total, ~$6/month in ap-south-1)

### DNS & Certificate Costs
- [ ] Route 53: $0.50/month per hosted zone + $0.40 per million queries
- [ ] ACM: free certificates
- [ ] Let's Encrypt via cert-manager: free certificates

### Operational Costs
- [ ] Secrets Manager: $0.40/secret/month × 6 = $2.40/month
- [ ] Jenkins: EC2 instance cost (separate from this stack)
- [ ] Monitoring/observability: not yet implemented (cost to be added)

### Cost Optimization Opportunities
Candidate should be able to identify:
1. **Graviton instances** — ~20% cheaper for same workload
2. **Spot instances** — up to 90% savings for fault-tolerant workloads
3. **VPC endpoints** — eliminate NAT costs for AWS API calls (S3, ECR, STS)
4. **Reserved instances / Savings Plans** — 30-60% savings for committed usage
5. **Right-sizing** — monitor actual CPU/memory usage to determine if t3a.medium is appropriate
6. **NAT gateway** — consider NAT instance for dev environments (~$4/month vs $32/month)
7. **CloudFront** — PriceClass_100 if South Asia is the only target audience (ap-south-1 is in PriceClass_200)

### Monthly Cost Estimate (Candidate Should Approximate)
| Resource | Estimated Cost |
|----------|---------------|
| EKS control plane | $73 |
| 3x t3a.medium nodes | ~$92 |
| NAT gateway | ~$35 |
| CloudFront | ~$1-10 (traffic dependent) |
| Route 53 | ~$1 |
| S3 (state + frontend) | ~$1 |
| ECR | ~$1 |
| Secrets Manager | ~$3 |
| EBS (60 GiB) | ~$6 |
| **Total** | **~$210-220/month** |
