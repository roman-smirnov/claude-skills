# AWS Architecture — Interview Module

## Enhancement Instructions

- Require **formal definitions** for every AWS service discussed. Score lower when the candidate uses service names without explaining what the service does and why it was chosen.
- Probe **blast radius reasoning**: for every failure-mode question, expect the candidate to map which components are affected, which survive, and how long recovery takes.
- Evaluate **least-privilege depth** for IAM: trust policies, permission boundaries, condition keys, IRSA mechanics, and the difference between node-level and pod-level identity.
- Score higher when the candidate connects AWS decisions to **cost implications**: knows approximate costs, can compare alternatives, and can propose the highest-ROI optimization.
- Evaluate **FinOps maturity**: tagging strategy, budget alerts, cost anomaly detection, monthly review cadence, and the ability to separate quick wins from strategic optimization.

---

## Evaluation Checklist

### Architecture and Service Selection

- [ ] Can describe the platform service map clearly (identity, network, runtime, data, edge, observability)
- [ ] Can explain why each major AWS service was chosen over alternatives
- [ ] Can identify failure domains (AZ, region, account, control-plane dependency)
- [ ] Can explain which failure domains are acceptable vs unacceptable
- [ ] Can connect architecture decisions to team size, delivery speed, and risk tolerance

### IAM and Identity

#### Core IAM Mechanics
- [ ] Understands IAM policy evaluation model (allow/deny precedence)
- [ ] Understands trust policy vs permission policy
- [ ] Understands role assumption flow and temporary credentials
- [ ] Understands identity scope for workloads (IRSA/task role/instance profile)

#### Least Privilege and Blast Radius
- [ ] Uses least-privilege resource scoping (no broad wildcards unless justified)
- [ ] Limits high-risk actions with explicit conditions where possible
- [ ] Can identify over-privileged identities and right-size them
- [ ] Can describe blast radius if key automation identity is compromised

#### Operational Identity Safety
- [ ] Avoids long-lived static credentials for workloads
- [ ] Can debug `AccessDenied` systematically
- [ ] Can explain break-glass controls and auditability

### Networking and Traffic

#### Core Networking
- [ ] Understands VPC/subnet segmentation and public/private tiering
- [ ] Understands route tables and egress/ingress pathing
- [ ] Understands security groups vs NACL purpose and precedence
- [ ] Understands NAT strategy and endpoint alternatives

#### Reliability and Scale
- [ ] Can identify network SPOFs (single NAT, shared gateways)
- [ ] Can explain AZ spread and resilience implications
- [ ] Can reason about IP/subnet/ENI constraints before scaling events
- [ ] Can explain DNS behavior, TTL impact, and propagation/caching pitfalls

#### Troubleshooting
- [ ] Can debug end-to-end request path with ordered checks
- [ ] Can isolate DNS vs routing vs ACL/SG vs upstream failures
- [ ] Can triage edge-layer issues (CloudFront/ALB/API Gateway) with concrete evidence

### Compute and Runtime Operations

- [ ] Can justify runtime choice (EKS/ECS/Lambda/EC2) for workload profile
- [ ] Can explain what would trigger a different runtime decision
- [ ] Understands autoscaling signals and policy tuning
- [ ] Can explain rollout and rollback mechanism
- [ ] Can explain health-check and replacement behavior

### EKS Operations (NEW)

- [ ] Understands EKS control plane vs data plane responsibilities
- [ ] Can explain managed node groups vs self-managed vs Fargate profiles
- [ ] Knows EKS add-on management (vpc-cni, kube-proxy, coredns, ebs-csi-driver)
- [ ] Can explain EKS upgrade strategy (control plane first, then data plane)
- [ ] Understands EKS Pod Identity vs IRSA (newer mechanism)

### ECR Management (NEW)

- [ ] Understands ECR image scanning (scan-on-push, enhanced scanning)
- [ ] Can explain lifecycle policies (keep N images, expire untagged, age-based)
- [ ] Knows ECR replication for cross-region/cross-account access
- [ ] Can explain ECR authentication patterns (docker-credential-ecr-login, IAM)
- [ ] Understands image immutability tag settings

### Data, Storage, and DR

#### Data-Layer Fundamentals
- [ ] Understands storage service fit (RDS/Aurora/DynamoDB/S3/EBS/EFS)
- [ ] Understands encryption-at-rest and in-transit controls
- [ ] Understands backup/retention settings and lifecycle controls

#### Recovery Discipline
- [ ] Can define realistic RTO and RPO targets
- [ ] Can explain restore workflow for major data components
- [ ] Can explain restore testing cadence and evidence
- [ ] Can identify data-loss scenarios still possible and why

### EBS CSI Driver (NEW)

- [ ] Understands EBS CSI driver as the standard for dynamic EBS provisioning in EKS
- [ ] Can explain gp3 vs gp2 vs io2 volume type selection
- [ ] Knows IRSA requirement for CSI driver pod-level AWS access
- [ ] Can troubleshoot PVC stuck in Pending (CSI driver, StorageClass, IRSA)

### ALB/NLB Integration (NEW)

- [ ] Understands AWS Load Balancer Controller for Ingress and Service resources
- [ ] Can explain ALB Ingress Controller vs NLB for different traffic patterns
- [ ] Knows target group binding and health check configuration
- [ ] Can explain when to use ALB (L7) vs NLB (L4)

### Edge, DNS, and Certificates

- [ ] Understands CloudFront origin and cache policy behavior
- [ ] Understands OAC/OAI/public-origin security tradeoffs
- [ ] Understands Route 53 routing strategy and record management
- [ ] Understands ACM certificate lifecycle and validation dependencies
- [ ] Can troubleshoot edge-origin mismatch and stale-cache issues

### Security and Compliance Controls

- [ ] Can explain KMS key ownership and access controls
- [ ] Can explain secrets lifecycle (storage, access, rotation, revocation)
- [ ] Can explain audit trail design (CloudTrail/infra logs/deployment history)
- [ ] Can identify missing guardrails (policy checks, detection controls)
- [ ] Can propose containment actions for suspected credential compromise

### Cost and Governance

- [ ] Can identify top 3 cost drivers in the current architecture
- [ ] Can propose practical optimization with risk assessment
- [ ] Understands egress, NAT, storage retention, and overprovisioning costs
- [ ] Understands tagging/accountability model for cost allocation
- [ ] Can explain when to use spot/reserved/savings plans and associated risk controls

#### FinOps Operating Model
- [ ] Can explain a monthly review cadence (discover → analyze → prioritize → monitor)
- [ ] Can separate quick wins vs medium-effort vs strategic optimization initiatives
- [ ] Can define ownership model for optimization actions and tracking
- [ ] Can explain budget/anomaly escalation paths and expected response time

#### Cost Optimization Heuristics
- [ ] Quick-win discovery: unattached EBS volumes, old snapshots, unused Elastic IPs, idle gateways/LBs, idle compute
- [ ] Commitment strategy: identifies stable workloads for RI/Savings Plans, avoids commitments for variable workloads
- [ ] Strategic optimization: old generation migration, Graviton suitability, Spot candidate identification

### Observability and Incident Readiness

- [ ] Uses clear signal model (metrics, logs, traces, events)
- [ ] Distinguishes symptom alerts from component-noise alerts
- [ ] Knows alarm-quality basics: severity labels, `for` windows, runbook links
- [ ] Can define SLI/SLO/error budget and use them during incidents
- [ ] Can run structured incident phases: detect → triage → investigate → mitigate → verify → post-incident

### Delivery Integration (Terraform, GitOps, CI/CD)

- [ ] Understands Terraform state safety (remote backend, locking, encryption, blast radius partitioning)
- [ ] Understands drift detection and reconciliation strategy
- [ ] Understands CI-to-CD handoff and rollback path
- [ ] Identifies anti-patterns (manual production mutation, missing plan review, missing promotion gates)
- [ ] Can explain reproducible release evidence (immutable artifacts, tag/digest discipline)

### AWS Systems Manager (NEW)

- [ ] Understands SSM Session Manager for secure node access (no SSH keys needed)
- [ ] Knows SSM Parameter Store vs Secrets Manager tradeoffs
- [ ] Can explain SSM Patch Manager for node patching strategy

### Diagnostic Commands

```bash
# Identity and IAM
aws sts get-caller-identity
aws iam get-role --role-name <name>
aws iam simulate-principal-policy --policy-source-arn <arn> --action-names <action>

# Networking
aws ec2 describe-vpcs
aws ec2 describe-route-tables
aws ec2 describe-security-groups
aws ec2 describe-nat-gateways

# Compute / EKS
aws eks describe-cluster --name <cluster>
aws eks list-nodegroups --cluster-name <cluster>
aws eks describe-addon --cluster-name <cluster> --addon-name <addon>

# ECR
aws ecr describe-images --repository-name <repo>
aws ecr get-lifecycle-policy --repository-name <repo>
aws ecr describe-image-scan-findings --repository-name <repo> --image-id imageTag=<tag>

# Cost and Monitoring
aws cloudwatch describe-alarms
aws ce get-cost-and-usage --time-period Start=<date>,End=<date> --granularity DAILY --metrics BlendedCost
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=AssumeRole

# Kubernetes cross-check
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
argocd app get <app>
```

### Scoring Application

| Coverage | Score |
|----------|-------|
| 0-20% of relevant items | 1–25 (Awareness) |
| 20-50% | 26–50 (Competent) |
| 50-80% with sound tradeoff and diagnostic reasoning | 51–75 (Proficient) |
| 80%+ with proactive depth and clear operational judgment | 76–100 (Expert) |

---

## Question Templates

### What Breaks If...

1. "Your architecture at `{path}` assumes a single region and that region has a control-plane outage. What breaks first, what degrades, and what survives?"
2. "Your single NAT gateway in `{path}` goes down. What happens to workloads in private subnets? How long until AWS restores it?"
3. "The IRSA trust policy for external-secrets in `{path}` is misconfigured with the wrong OIDC issuer URL. What symptom does the user see?"
4. "Your CloudFront distribution's S3 OAC policy is too permissive. What is the blast radius?"
5. (generic) "A broad policy (`Action: *`, `Resource: *`) is attached temporarily to a service role and never removed. What is the real blast radius?"

### How Would You Debug...

6. "A workload gets `AccessDenied` for an action it previously performed successfully. Walk me through your diagnostic order."
7. "ECR image pulls from EKS nodes are failing with `unauthorized`. Given the IAM role at `{path}`, what do you check?"
8. "Route 53 DNS resolution for `{config_detail}` is returning stale records after a change. Walk me through your debugging."
9. (generic) "An IAM policy you wrote is being denied despite having the right Action and Resource. What else could cause the denial?"
10. "Traffic reaches the load balancer but not targets in `{path}`. What do you inspect first?"

### Tradeoff

11. "Single NAT gateway vs per-AZ NAT gateways: cost vs availability tradeoff in this architecture."
12. "You use CloudFront PriceClass_200 in `{path}`. What does this exclude and when would you upgrade to PriceClass_All?"
13. "IRSA/task role/service identity vs static credentials in secrets: what tradeoffs matter most here?"
14. "VPC endpoints vs NAT egress for AWS APIs: where is the cost/security break-even in this platform?"
15. "EKS managed node groups vs self-managed node groups vs Fargate: which fits this workload and why?"

### Technology Comparison

16. "[DEFINE] What is IRSA (IAM Roles for Service Accounts)? Why IRSA instead of node-level IAM roles, kiam, or kube2iam? How does the OIDC federation work under the hood?"
17. "[DEFINE] What is CloudFront? What is an Origin Access Control (OAC)? Why CloudFront + S3 OAC instead of serving the frontend directly from S3 website hosting or from nginx behind a load balancer?"
18. "[DEFINE] What is ExternalDNS? Why ExternalDNS + Route 53 instead of managing DNS records manually or via Terraform? What is cert-manager and how does DNS-01 validation work with Route 53?"
19. "[DEFINE] What is a NAT Gateway? Why a managed NAT Gateway instead of a NAT instance, or a VPC endpoint-based approach? What is the cost difference?"
20. "[DEFINE] What is a security group vs a network ACL? Why use both, and where should primary control live?"
21. "[DEFINE] What is a Well-Architected workload in AWS terms? How do the pillars translate to this project?"

### Best Practice Pillar

22. **(Cost)** "Break down the monthly AWS cost of this infrastructure: EKS control plane, 3x t3a.medium nodes, single NAT gateway, CloudFront, Route 53, S3, ECR, Secrets Manager. Where are the biggest costs? What would you optimize first?"
23. **(Scalability)** "Your VPC uses /24 subnets (256 IPs each). Is this enough for future growth? What happens when you need more than 3 nodes or more pods? How would you expand the IP space?"
24. **(Availability)** "Map your major failure domains and explain which are acceptable vs unacceptable for this project."
25. **(Security)** "Which identity path in this repo is currently most over-privileged and how would you right-size it?"
26. **(Cost)** "Reserved Instances vs Savings Plans vs On-Demand: how do you decide commitment strategy for this repo's workload profile?"

### EKS Deep-Dive (NEW)

27. "Explain the EKS control plane vs data plane separation. What happens to your workloads if the control plane is briefly unavailable? What operations fail?"
28. "You need to upgrade EKS from version 1.28 to 1.30. Walk me through the upgrade strategy: control plane first, then add-ons, then node groups. What can go wrong at each step?"
29. "When would you choose Fargate profiles over managed node groups for EKS workloads? What are the limitations of Fargate for this project?"

### ECR Questions (NEW)

30. "Your ECR lifecycle policy keeps the last 10 tagged images and expires untagged images after 1 day. A developer accidentally pushes without a tag. What happens? How would you improve this policy?"
31. "ECR enhanced scanning reports HIGH CVEs in your base image. Walk me through your triage: which CVEs matter, how do you assess exploitability, and what is your remediation timeline?"

### EBS CSI and Storage (NEW)

32. "The EBS CSI driver in your cluster needs IRSA permissions to provision volumes. Walk me through what happens if the IRSA role is deleted: existing volumes vs new PVC requests."

### ALB/NLB (NEW)

33. "Your services use the AWS Load Balancer Controller. When would you choose an ALB Ingress (L7) vs an NLB Service (L4)? What are the cost and feature tradeoffs?"

### Bonus Pool

34. "What is your safe break-glass model during SEV-1 incidents? How do you grant temporary elevated access without violating least-privilege?"
35. "How do you audit and remediate IAM drift continuously?"
36. (generic) "What evidence would convince you to split the platform into multiple AWS accounts now, not later?"
37. "What is AWS Systems Manager Session Manager? How does it eliminate the need for SSH bastion hosts?"
