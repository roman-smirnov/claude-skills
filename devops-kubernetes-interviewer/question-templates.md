# Question Templates

Each round draws from five mandatory categories plus a pool of bonus questions. Templates use `{placeholder}` for repo-specific values filled in during the interview.

**Mandatory categories per round:**
1. "What breaks if..." — failure-mode
2. "How would you debug..." — troubleshooting
3. "Tradeoff" — design-choice
4. "Technology Comparison" — why X instead of Y (candidate must define both technologies formally)
5. "Best Practice Pillar" — security, scalability, availability, or cost optimization

---

## Template Syntax

- `{path}` — actual file path from repo intake
- `{resource}` — Terraform resource name found in code
- `{module}` — module directory name
- `{service}` — Kubernetes service or app name
- `{config_detail}` — specific config value found in code (e.g., instance type, CIDR, node count)
- `[SKIP if no artifact]` — skip this question if the repo lacks the referenced artifact; fall back to the generic variant
- `(generic)` — version of the question that does not require repo artifacts
- `[DEFINE]` — candidate is expected to formally define the technology/term before answering

---

## Round 1: Terraform & IaC

### What Breaks If...

1. "Your S3 state backend at `{path}` loses the lock mechanism. Two engineers run `terraform apply` simultaneously. What happens to your state file, and how do you recover?"
2. "Someone manually deletes the `{resource}` resource in the AWS console but the Terraform state still references it. Walk me through what happens on the next `terraform plan` and how you resolve it."
3. "The `prevent_destroy` lifecycle rule on `{resource}` in `{path}` is accidentally removed and someone runs `terraform destroy`. What is the blast radius and how would you recover?"
4. (generic) "What happens if your Terraform provider version is unpinned and a breaking change ships upstream?"

### How Would You Debug...

5. "You run `terraform plan` and see 15 unexpected changes to resources you didn't touch. The state file is at `{path}`. Walk me through your debugging process."
6. "A new engineer adds a module but `terraform validate` passes while `terraform plan` fails with a cycle error. How do you diagnose and fix this?"
7. (generic) "Your CI pipeline shows `terraform fmt -check` failing. What does this mean and how do you enforce formatting across a team?"

### Tradeoff

8. "In `{path}`, you use a single NAT gateway instead of one per AZ. What are the tradeoffs? When would you change this decision?"
9. "Your bootstrap phase at `bootstrap/` is separate from the main Terraform root. Why partition state this way instead of using one state file for everything?"
10. "You use `data` sources to read secrets from Secrets Manager instead of managing secret values in Terraform. What are the security and operational tradeoffs?"

### Technology Comparison

11. "[DEFINE] What is Terraform and what does it do in this project? Why Terraform instead of Pulumi, CloudFormation, or CDK? What specific advantages does the declarative HCL approach give you for this EKS-based infrastructure?"
12. "[DEFINE] Your state backend uses S3 with native locking. What is Terraform state locking? Why S3 native locking instead of DynamoDB-based locking? When was native S3 locking introduced and what limitation did it remove?"
13. "[DEFINE] You use Terraform modules for VPC, EKS, IAM, ArgoCD, and CDN. What is a Terraform module? Why write custom modules instead of using community modules from the Terraform Registry (e.g., `terraform-aws-modules/vpc/aws`)?"

### Best Practice Pillar

14. **(Security)** "Your Terraform state file contains sensitive data (IRSA role ARNs, OIDC endpoints, VPC IDs). What security measures protect the state backend at `{path}`? What would an attacker learn from your state file?"
15. **(Cost)** "Walk me through the cost implications of your Terraform architecture: single NAT gateway, t3a.medium nodes, CloudFront PriceClass_200, ECR lifecycle policy keeping 10 images. What's the estimated monthly cost and where would you cut first?"
16. **(Availability)** "Your bootstrap stack at `bootstrap/` has `prevent_destroy` on critical resources. From an availability perspective, what happens to the main stack if the bootstrap state file is corrupted?"

### Bonus Pool

17. "Explain the module structure convention: why must every module have `main.tf`, `variables.tf`, `outputs.tf`, and `versions.tf`?"
18. "What should a pre-commit pipeline for Terraform include, and in what order? (Hint: think fmt, validate, lint, scan.)"
19. "When would you use `terraform import` vs `terraform state mv` vs `moved` blocks?"
20. "Your `{module}` module has variables without descriptions. Why is this a problem?"

---

## Round 2: Kubernetes

*Grounded in `k8s-troubleshooter@devops-skills` standards. Questions probe systematic troubleshooting methodology, common issue diagnosis, incident response, and performance analysis.*

### What Breaks If...

1. "Your EKS node group in `{path}` is set to `{config_detail}` nodes. One node becomes NotReady. Walk me through the `k8s-troubleshooter` diagnostic workflow: what do you check at the node level (kubelet, disk pressure, memory pressure, container runtime), what happens to Pods on that node, and how does Kubernetes respond?"
2. "The `vpc-cni` addon configured in `{path}` runs out of IP addresses in the subnet. According to the common issues reference, how does this manifest — what symptoms do you see at the Pod level and the node level? What are your remediation options?"
3. (generic) "A Deployment has no resource requests or limits set. What are the consequences during a node resource crunch? How does the Kubernetes QoS class system determine which Pods get evicted first?"
4. "The `aws-ebs-csi-driver` addon in `{path}` loses its IRSA permissions. What happens to existing PVCs? What happens when a new Pod tries to claim a PVC? Walk me through the troubleshooting steps from the storage issues playbook."
5. (generic) "A container is repeatedly OOMKilled (exit code 137). Walk me through the systematic diagnosis: how do you confirm OOMKill, profile memory usage, determine whether it's a leak or an undersized limit, and what remediation steps do you take?"

### How Would You Debug...

6. "A Pod is stuck in `CrashLoopBackOff`. Following the k8s-troubleshooter workflow, walk me through your debugging steps: what kubectl commands do you run, in what order, and what are you looking for at each step?"
7. "Services in namespace A cannot reach services in namespace B. Using the networking issues diagnostic approach, what do you check first, second, third? How do you distinguish between a network policy issue, a CNI issue, and a service misconfiguration?"
8. "Your EBS-backed PVC is stuck in `Pending`. Given that you have the `aws-ebs-csi-driver` addon in `{path}`, walk me through the storage troubleshooting workflow: what diagnostic commands do you run, what are the likely causes, and how do you verify each?"
9. (generic) "You suspect CPU throttling is causing latency spikes in your application. How do you confirm throttling is happening, distinguish it from other latency sources, and what are your remediation options — including the controversial choice of removing CPU limits entirely?"
10. "A Pod shows `ImagePullBackOff`. Your images are stored in ECR configured at `{path}`. Walk me through the full diagnostic checklist: image name/tag verification, registry authentication, network issues, rate limiting. How does the troubleshooting differ for ECR vs public registries?"

### Tradeoff

11. "You use `t3a.medium` instances for your EKS nodes. What are the tradeoffs vs `t3.medium`, `m6i.large`, or Graviton-based instances? How do burst credits affect Kubernetes workloads specifically?"
12. "Public + private endpoint access is enabled for your EKS cluster in `{path}`. Why not private-only? What are the security implications of each mode?"
13. (generic) "When would you use a DaemonSet vs a Deployment with node affinity? Give concrete examples from a production Kubernetes environment."
14. (generic) "Resource requests vs limits: some teams set requests = limits (Guaranteed QoS), others set only requests with no limits (Burstable QoS). What are the tradeoffs and when would you use each strategy?"

### Technology Comparison

15. "[DEFINE] What is EKS? Why EKS instead of self-managed Kubernetes (kubeadm), ECS, or k3s? What does the managed control plane give you that self-managed doesn't? What operational burden does it remove?"
16. "[DEFINE] What is VPC-CNI? You use the AWS VPC-CNI plugin in `{path}`. Why VPC-CNI instead of Calico, Flannel, or Cilium? How does VPC-CNI assign Pod IP addresses differently, and what are the IP exhaustion implications?"
17. "[DEFINE] What is the EBS CSI Driver? Why use the CSI driver addon instead of the in-tree EBS provisioner? What is IRSA and why does the CSI driver need it?"
18. "[DEFINE] What is CoreDNS? How does DNS resolution work for `service.namespace.svc.cluster.local`? Why CoreDNS instead of kube-dns? What performance issues can arise with DNS in large clusters?"

### Best Practice Pillar

19. **(Scalability)** "Your EKS node group is fixed at 3 nodes in `{path}`. What is a Cluster Autoscaler vs Karpenter? Why don't you have auto-scaling enabled, and what are the risks? How would the k8s-troubleshooter diagnose a 'Pending Pods due to insufficient resources' scenario?"
20. **(Availability)** "You have 2 AZs with 3 nodes. What happens if one AZ goes down? How many nodes survive? Can your workloads still run? What Kubernetes primitives (PDBs, topology spread constraints, pod anti-affinity) would improve availability?"
21. **(Security)** "What are Kubernetes RBAC, Pod Security Standards, and Network Policies? Which of these are implemented in your cluster? What attack vectors remain unaddressed? How would you use `kubectl auth can-i` to audit permissions?"
22. **(Cost)** "Your 3x t3a.medium nodes cost ~$92/month. How do you determine if they're right-sized? What kubectl and metrics-server commands reveal actual utilization? What cost savings could Karpenter with spot instances achieve?"

### Bonus Pool (k8s-troubleshooter depth)

23. "Walk me through the incident response framework for a SEV-1 Kubernetes outage: detection, triage, investigation, resolution, and post-incident phases. What specific kubectl commands do you run at each phase?"
24. "Explain the difference between liveness, readiness, and startup probes. When would you use each? What happens when each type fails, and how does misconfiguring them lead to cascading failures?"
25. "What is a PodDisruptionBudget and when is it critical? How does it interact with node drains, cluster autoscaler, and rolling updates?"
26. "Describe the Kubernetes eviction hierarchy: what determines which Pods get evicted first during node pressure? How do QoS classes, priority classes, and resource requests interact?"
27. "Your cluster's control plane seems slow — API server requests are taking 5+ seconds. Walk me through the performance troubleshooting approach: what metrics do you check on the API server, etcd, scheduler, and controller manager?"

---

## Round 3: Helm

### What Breaks If...

1. "A Helm release upgrade fails mid-way. What state is the release in, and what are your recovery options? How does `--atomic` change the behavior?"
2. "Someone runs `helm upgrade` with `--set` flags that conflict with the values file used by ArgoCD. What happens on the next ArgoCD sync?"
3. (generic) "A chart subchart dependency has a breaking change in a new version. How does this propagate?"

### How Would You Debug...

4. "A Helm chart renders differently in CI vs locally. How do you isolate the difference? Which commands do you use?"
5. "`helm template` produces valid YAML but `helm install` fails. What are the common causes?"
6. [SKIP if no Helm charts] "The Helm chart at `{path}` uses values layering. How do you verify which values actually get applied?"

### Tradeoff

7. "ArgoCD manages your Helm releases. Why use ArgoCD's Helm support instead of raw `helm install/upgrade` in CI?"
8. "When would you use a Helm library chart vs copy-pasting templates across charts?"
9. (generic) "Helm vs Kustomize — when would you pick one over the other?"

### Technology Comparison

10. "[DEFINE] What is Helm? Why use Helm for packaging Kubernetes applications instead of raw YAML manifests with `kubectl apply`? What problems does Helm solve that plain manifests don't?"
11. "[DEFINE] What is a Helm chart vs a Helm release vs a Helm repository? You use ArgoCD to manage Helm releases. Why not Helmfile or Flux's HelmRelease CRD?"
12. (generic) "[DEFINE] What are Helm hooks? How do they compare to ArgoCD sync hooks and resource hooks? Which takes precedence when ArgoCD deploys a Helm chart?"

### Best Practice Pillar

13. **(Availability)** "A Helm release upgrade fails and leaves the release in a `FAILED` state. What is Helm's rollback mechanism? How many release versions does Helm keep by default, and how does this affect your ability to recover?"
14. **(Security)** "Helm values files can contain secrets. How should secrets be handled in Helm charts? How does your project avoid putting secrets directly in values files?"

---

## Round 4: ArgoCD / GitOps

### What Breaks If...

1. "The SSH key for your GitOps repo (stored in Secrets Manager as `gitlab-cowsay3d-gitops-key`, read in `{path}`) is rotated but ArgoCD's repo secret is not updated. What happens?"
2. "ArgoCD's sync policy in `{path}` has `selfHeal: true` and `prune: true`. Someone manually applies a hotfix with `kubectl apply`. What happens within the next sync interval?"
3. (generic) "The Git repo backing ArgoCD becomes unavailable for 2 hours. What is the impact on running workloads?"

### How Would You Debug...

4. "An Application shows `OutOfSync` but `argocd app diff` shows no meaningful differences. What are the common causes?"
5. "Sync waves in your app-of-apps (`{path}`) are failing at wave 2. How do you investigate which resource is blocking?"
6. (generic) "ArgoCD is healthy but a new Application you added never appears. Where do you look?"

### Tradeoff

7. "You use app-of-apps pattern in `{path}`. When would ApplicationSets be a better choice?"
8. "Auto-sync with prune vs manual sync — what are the risk/speed tradeoffs for production?"
9. "Your sync waves order is cert-manager -> external-secrets -> ingress-nginx -> external-dns -> app. Why this order? What would break if you changed it?"

### Technology Comparison

10. "[DEFINE] What is ArgoCD? What is GitOps? Why ArgoCD instead of Flux CD, Jenkins CD (push-based), or Spinnaker? What specific advantage does the pull-based reconciliation model give you?"
11. "[DEFINE] What is an ArgoCD Application CRD? You use app-of-apps in `{path}`. What are ApplicationSets and how do they differ? Why did you choose app-of-apps over ApplicationSets for this project?"
12. "[DEFINE] What are ArgoCD sync waves and resource hooks? How do they compare to Helm hooks? Why use sync waves to order cert-manager before ingress-nginx?"

### Best Practice Pillar

13. **(Availability)** "ArgoCD itself is a single point of failure for your deployment pipeline. If ArgoCD goes down, can you still deploy? What is your disaster recovery plan for ArgoCD?"
14. **(Security)** "ArgoCD has access to your Git repo via SSH key and full cluster admin. What is the blast radius if ArgoCD is compromised? How would you reduce its privilege scope?"
15. **(Scalability)** "As you add more applications and environments, how does ArgoCD scale? What is Application Controller sharding and when would you need it?"

---

## Round 5: AWS Architecture

### What Breaks If...

1. "Your single NAT gateway in `{path}` goes down. What happens to workloads in private subnets? How long until AWS restores it?"
2. "The IRSA trust policy for external-secrets in `{path}` is misconfigured with the wrong OIDC issuer URL. What symptom does the user see?"
3. "Your CloudFront distribution's S3 OAC policy is too permissive. What is the blast radius?"

### How Would You Debug...

4. "ECR image pulls from EKS nodes are failing with `unauthorized`. Given the IAM role at `{path}`, what do you check?"
5. "Route 53 DNS resolution for `{config_detail}` is returning stale records after a change. Walk me through your debugging."
6. (generic) "An IAM policy you wrote is being denied despite having the right Action and Resource. What else could cause the denial?"

### Tradeoff

7. "You use CloudFront PriceClass_200 in `{path}`. What does this exclude and when would you upgrade to PriceClass_All?"
8. "S3 native state locking vs DynamoDB locking — you chose native. What are the tradeoffs?"
9. "You store secret values manually in Secrets Manager rather than in Terraform. Why? What are the operational costs?"

### Technology Comparison

10. "[DEFINE] What is IRSA (IAM Roles for Service Accounts)? Why IRSA instead of node-level IAM roles, kiam, or kube2iam? How does the OIDC federation work under the hood?"
11. "[DEFINE] What is CloudFront? What is an Origin Access Control (OAC)? Why CloudFront + S3 OAC instead of serving the frontend directly from S3 website hosting or from nginx behind a load balancer?"
12. "[DEFINE] What is ExternalDNS? Why ExternalDNS + Route 53 instead of managing DNS records manually or via Terraform? What is cert-manager and how does DNS-01 validation work with Route 53?"
13. "[DEFINE] What is a NAT Gateway? Why a managed NAT Gateway instead of a NAT instance, or a VPC endpoint-based approach? What is the cost difference?"

### Best Practice Pillar

14. **(Cost)** "Break down the monthly AWS cost of this infrastructure: EKS control plane, 3x t3a.medium nodes, single NAT gateway, CloudFront, Route 53, S3, ECR, Secrets Manager. Where are the biggest costs? What would you optimize first?"
15. **(Scalability)** "Your VPC uses /24 subnets (256 IPs each). Is this enough for future growth? What happens when you need more than 3 nodes or more pods? How would you expand the IP space?"
16. **(Availability)** "Your architecture has several SPOFs: single NAT gateway, single CloudFront origin for the backend. Map out the failure domains and their blast radius."

---

## Round 6: Reliability / Observability / Incident Response

### What Breaks If...

1. "Your application exposes `/health` and `/metrics` endpoints (routed through CloudFront with no-cache in `{path}`). CloudFront goes down. Can your health checks still work? What about Kubernetes probes?"
2. (generic) "Your alerting system fires 50 alerts in 5 minutes during an incident. How do you triage?"
3. (generic) "Your SLO error budget is exhausted mid-sprint. What actions should the team take?"

### How Would You Debug...

4. "Latency spikes on `/api/conversation` but CPU and memory look normal. What do you check?"
5. (generic) "A canary deployment shows 2% error rate vs 0.1% baseline. Is this actionable? How do you decide?"
6. "Your Jenkins pipeline at `{path}` passes all tests but the deployed application is broken. What gap in your testing strategy allowed this?"

### Tradeoff

7. "Prometheus pull-based vs push-based metrics collection — which is better for your EKS setup and why?"
8. (generic) "Detailed logging vs cost — how do you decide what log level to run in production?"
9. (generic) "Single global SLO vs per-endpoint SLOs — when does each make sense?"

### Technology Comparison

10. "[DEFINE] What is an SLI, SLO, and SLA? How do they relate to each other? Your app exposes `/health` and `/metrics` — what SLIs would you derive from these endpoints?"
11. "[DEFINE] What is Prometheus? Why Prometheus for metrics instead of CloudWatch, Datadog, or OpenTelemetry Collector with a backend? What are the cost and operational tradeoffs?"
12. (generic) "[DEFINE] What is distributed tracing? Why would you need it for a microservices architecture? Compare Jaeger, Zipkin, and AWS X-Ray. Which would fit this project best?"

### Best Practice Pillar

13. **(Availability)** "Define RTO and RPO. What are the RTO and RPO for this project's components: EKS cluster, MongoDB data, frontend static assets, DNS records? Which component has the longest recovery time?"
14. **(Security)** "Your Jenkins pipeline has access to ECR, S3, GitLab, and the EKS cluster. If Jenkins is compromised, what is the blast radius? How would you reduce it?"
15. **(Cost)** "Observability infrastructure (Prometheus, Grafana, log aggregation) can itself become expensive. How do you budget for observability in a small team? What's the minimum viable monitoring stack?"

---

## Round 7: Security

### What Breaks If...

1. "The `external-secrets-sa` service account's IRSA role in `{path}` is granted `secretsmanager:*` instead of the scoped policy. What is the blast radius?"
2. "Your Docker images run as `appuser` (UID 1000). What if a vulnerability allows container escape? What additional layers protect the node?"
3. "Someone pushes a `.env` file past your `.gitignore` and it reaches the remote. What is your incident response?"

### How Would You Debug...

4. "A Pod running with IRSA cannot access Secrets Manager. `aws sts get-caller-identity` inside the pod shows the node role instead of the IRSA role. What went wrong?"
5. "ECR scan-on-push flags a HIGH CVE in your base image. Walk me through your triage and remediation."
6. (generic) "You suspect a compromised container in your cluster. What do you do in the first 5 minutes? How does the k8s-troubleshooter security incident playbook apply?"

### Tradeoff

7. "External Secrets Operator vs mounting secrets as environment variables from Kubernetes Secrets — what are the security tradeoffs?"
8. "You use Jenkins credentials (Secrets Manager) with `jenkins:credentials:type` tags. Why not use Kubernetes-native secrets directly?"
9. (generic) "Image signing and admission policies — is the operational overhead worth it for a small team?"

### Technology Comparison

10. "[DEFINE] What is the External Secrets Operator (ESO)? Why ESO instead of Sealed Secrets, SOPS, Vault, or the AWS Secrets and Configuration Provider (ASCP) for the CSI driver? What are the security and operational tradeoffs?"
11. "[DEFINE] What is a Container Security Context? Your Dockerfile uses a non-root user (`appuser`, UID 1000). Why is this important? How does it compare to using `runAsNonRoot` in Kubernetes vs Pod Security Standards (PSS/PSA)?"
12. (generic) "[DEFINE] What is OPA/Gatekeeper vs Kyverno for Kubernetes policy enforcement? Would either help prevent misconfigurations in this cluster? What policies would you write first?"

### Best Practice Pillar

13. **(Security)** "Map the trust boundaries in this architecture: Jenkins -> ECR -> EKS -> Secrets Manager -> Route 53. At each boundary, what authentication and authorization mechanism is in place? Where is the weakest link?"
14. **(Scalability)** "As you add more secrets to Secrets Manager, how does the External Secrets Operator scale? What is the sync interval and what happens if Secrets Manager API rate limits are hit?"
15. **(Availability)** "Your SSH key for the GitOps repo is a single secret. What is your recovery plan if this key is rotated, revoked, or compromised? How quickly can you restore the GitOps pipeline?"
