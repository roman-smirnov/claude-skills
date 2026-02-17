# Technology Comparisons

> This file maps every technology choice in the project infrastructure to its alternatives.
> Use this when asking "Technology Comparison" questions and when scoring technology justification.
> The candidate should be able to define each technology formally and explain why it was chosen.

---

## How to Use This File

When a technology comes up in a question:
1. Look it up in this table
2. Ask the candidate to **define it formally** (1-2 sentences, as in an interview)
3. Ask **why this instead of [alternative]**
4. Evaluate whether they tie the choice to project constraints (team size, budget, AWS-native, GitOps model)

---

## Infrastructure as Code

| Category | Chosen | Formal Definition | Alternatives | Why This Choice |
|----------|--------|-------------------|-------------|-----------------|
| IaC Tool | **Terraform (HCL)** | An open-source Infrastructure as Code tool by HashiCorp that uses declarative configuration files (HCL) to provision and manage cloud resources through a plan/apply lifecycle. | Pulumi (imperative, real languages), CloudFormation (AWS-native, JSON/YAML), CDK (imperative wrapper for CFN), Ansible (procedural, config mgmt) | Declarative, cloud-agnostic, strong module ecosystem, explicit plan/apply cycle with state management. HCL is easier to review in PRs than imperative code. Terraform has established conventions (mandatory descriptions, version constraints, pre-commit pipeline). |
| Modules | **Terraform Modules (custom)** | Reusable, encapsulated Terraform configurations that accept inputs (variables) and produce outputs, used to compose infrastructure from building blocks. | Terraform Registry community modules, Terragrunt wrappers | Custom modules give full control over resource configuration, naming, tagging, and lifecycle rules. Community modules add complexity and version churn. |
| State Backend | **S3 Backend (native locking)** | Terraform backend that stores state files in S3 with server-side encryption and uses S3's native conditional writes for state locking (introduced in Terraform 1.6+). | S3 + DynamoDB (legacy locking), Terraform Cloud/HCP, Consul backend, local state | Native S3 locking removes DynamoDB cost and complexity. S3 provides durability, versioning, and encryption. |
| State Strategy | **State partitioning** | Splitting Terraform state by boundary (environment/platform/component) to reduce blast radius. | Single monolithic state | Limits failure impact, improves team parallelism, and simplifies controlled recovery. |

## Container Orchestration

| Category | Chosen | Formal Definition | Alternatives | Why This Choice |
|----------|--------|-------------------|-------------|-----------------|
| Orchestrator | **Amazon EKS** | A managed Kubernetes service that runs the control plane across multiple AZs, handling upgrades, patching, and HA of the API server and etcd. | Self-managed K8s (kubeadm), ECS (AWS-native), k3s/k0s (lightweight), GKE/AKS (other cloud) | Managed control plane eliminates operational burden. Deep AWS integration (IRSA, VPC-CNI, EBS CSI). Kubernetes-native tools (Helm, ArgoCD) rule out ECS. |
| CNI | **VPC-CNI** | AWS's Container Network Interface plugin that assigns real VPC IP addresses to Pods, making them first-class VPC citizens with direct security group and routing support. | Calico (overlay, network policies), Cilium (eBPF-based), Flannel (simple overlay) | VPC-CNI gives Pods real VPC IPs — no overlay overhead, native security group support, AWS load balancer compatibility. Tradeoff: higher IP consumption. |
| Storage | **EBS CSI Driver** | A Kubernetes Container Storage Interface driver that provisions and attaches Amazon EBS volumes as persistent storage for Pods, replacing the deprecated in-tree provisioner. | EFS CSI (shared filesystem), In-tree provisioner (deprecated), Local volumes | EBS provides block storage with snapshots and encryption. CSI is the Kubernetes-standard interface. Uses IRSA for secure AWS access. |

## GitOps and Deployment

| Category | Chosen | Formal Definition | Alternatives | Why This Choice |
|----------|--------|-------------------|-------------|-----------------|
| GitOps CD | **ArgoCD** | A declarative, GitOps-based continuous delivery tool for Kubernetes that continuously reconciles the desired state in Git with the live cluster state using a pull-based model. | Flux CD (CNCF, lighter), Jenkins CD (push-based), Spinnaker (multi-cloud, heavy) | Pull-based reconciliation, powerful UI for debugging sync issues, app-of-apps pattern, strong Helm integration. Flux is lighter but ArgoCD's UI and Application CRD model are more intuitive. |
| App Management | **App-of-apps pattern** | An ArgoCD pattern where a parent Application deploys child Application CRDs, enabling hierarchical management with ordered deployment via sync waves. | ApplicationSets (template-based generation), Flat Applications (each independent) | Explicit ordering with sync waves (cert-manager before ingress-nginx before app). ApplicationSets are better for large-scale, homogeneous deployments — overkill for this project's ~5 apps. |
| Packaging | **Helm (via ArgoCD)** | A Kubernetes package manager that uses templated YAML charts with configurable values to package, version, and deploy applications as releases. | Kustomize (overlay-based), Raw YAML, jsonnet, cdk8s | Helm provides parameterized charts, release versioning, and rollback. ArgoCD natively supports Helm sources. Kustomize is simpler but lacks packaging/versioning model. |

## CI/CD

| Category | Chosen | Formal Definition | Alternatives | Why This Choice |
|----------|--------|-------------------|-------------|-----------------|
| CI Server | **Jenkins** | An open-source automation server that executes CI/CD pipelines defined as code (Jenkinsfile), with plugin-based extensibility for build, test, and deployment stages. | GitLab CI (integrated with GitLab), GitHub Actions (integrated with GitHub), Tekton (Kubernetes-native), CircleCI/Travis (SaaS) | Jenkins is self-hosted (full control), integrates with GitLab repos via webhooks, supports complex multi-stage pipelines with parallel execution. |
| Deploy Pattern | **GitOps deploy (CI → GitOps repo → ArgoCD)** | A deployment pattern where CI writes the new image tag to a GitOps repository, and ArgoCD detects the change and syncs the cluster — separating CI (build/test) from CD (deploy). | Direct kubectl apply from CI (push-based), Helm upgrade from CI, ArgoCD image updater | GitOps separation provides audit trail (Git history), declarative state, and ArgoCD's self-healing. Push-based CI deploys bypass ArgoCD's reconciliation and cause drift. |

## Networking and DNS

| Category | Chosen | Formal Definition | Alternatives | Why This Choice |
|----------|--------|-------------------|-------------|-----------------|
| CDN | **CloudFront** | AWS's global content delivery network that caches and serves content from edge locations worldwide, reducing latency and offloading origin servers. | Direct S3 hosting, Nginx on EC2/EKS, Cloudflare, ALB + S3 | Global caching, HTTPS termination, OAC for private S3 access, HTTP/2+3 support. Cloudflare adds third-party dependency outside AWS ecosystem. |
| Origin Security | **Origin Access Control (OAC)** | A CloudFront security feature that allows CloudFront to access a private S3 bucket using signed requests, replacing the legacy OAI. | Origin Access Identity (OAI, legacy), Public S3 bucket, S3 website endpoint | OAC is the modern replacement — supports SSE-KMS, S3 bucket policies with conditions, and is AWS-recommended. Public S3 is a security risk. |
| DNS | **Route 53** | AWS's scalable, highly available DNS service that supports domain registration, DNS routing, and health checks, with native AWS integration. | GoDaddy DNS, Cloudflare DNS, External DNS providers | Integrates natively with CloudFront aliases, ACM certificate validation, and ExternalDNS. |
| DNS Automation | **ExternalDNS** | A Kubernetes add-on that automatically creates DNS records in external DNS providers (like Route 53) based on Ingress and Service annotations. | Manual DNS records, Terraform-managed records, CoreDNS external plugin | Automatically creates records when Ingress resources are created — GitOps-native DNS. Terraform-managed DNS requires a separate apply for every change. |
| Certificates | **cert-manager (DNS-01)** | A Kubernetes certificate management controller that automates issuance and renewal of TLS certificates from Let's Encrypt using ACME protocol challenges. | ACM only (AWS-managed), Manual Let's Encrypt (certbot), Self-signed certs | Automates certificate lifecycle within Kubernetes. DNS-01 works for wildcard certs without HTTP ingress. Uses IRSA to create Route 53 TXT records. |
| NAT | **NAT Gateway (single)** | An AWS-managed network address translation service that allows private subnet resources to access the internet without being directly reachable. | NAT Gateway per AZ (HA), NAT Instance (self-managed), VPC endpoints only | Single NAT saves cost. Acceptable for cost-sensitive or non-production deployments. Tradeoff: SPOF for outbound internet access. |

## Secrets Management

| Category | Chosen | Formal Definition | Alternatives | Why This Choice |
|----------|--------|-------------------|-------------|-----------------|
| Secret Store | **AWS Secrets Manager** | A managed secrets storage service that encrypts at rest (KMS), provides fine-grained IAM access control, supports automatic rotation, and integrates with AWS services. | SSM Parameter Store (simpler, cheaper), HashiCorp Vault (self-hosted), Kubernetes Secrets (native but base64 only) | Rotation support, cross-service access (Jenkins + Kubernetes), native IRSA integration. Parameter Store is cheaper but lacks rotation. Vault adds operational overhead. |
| K8s Secret Sync | **External Secrets Operator (ESO)** | A Kubernetes operator that synchronizes secrets from external secret management systems into native Kubernetes Secrets, keeping the source of truth outside the cluster. | Sealed Secrets (encrypted in Git), SOPS (encrypted files), CSI Secrets Store Driver (mount-based), Vault Agent Sidecar | ESO pulls secrets on a schedule and creates native K8s Secrets — works with any workload without sidecar changes. Sealed Secrets require encryption tooling. |
| Workload Identity | **IRSA** | IAM Roles for Service Accounts — an EKS feature that allows Kubernetes service accounts to assume AWS IAM roles via OpenID Connect (OIDC) federation, eliminating long-lived credentials. | Node-level IAM roles (too broad), kiam/kube2iam (deprecated), Static AWS credentials in Secrets | Pod-level, least-privilege AWS access using short-lived STS tokens. Node roles grant all Pods the same permissions. |
| Encryption | **KMS-managed encryption** | Centralized key management service for encrypt/decrypt operations and policy-based key access control. | Service-managed default keys only, app-level custom crypto | Strong access boundaries, auditable key usage, centralized governance model. |

## Docker and Container

| Category | Chosen | Formal Definition | Alternatives | Why This Choice |
|----------|--------|-------------------|-------------|-----------------|
| Registry | **ECR** | Amazon Elastic Container Registry — a fully managed Docker container registry with integrated vulnerability scanning and IAM-based access control. | Docker Hub (public/SaaS), GitLab Container Registry, GitHub Container Registry, Harbor (self-hosted) | Integrates natively with EKS (no separate credentials), scan-on-push, lifecycle policies, same-region fast pulls. |
| Build Strategy | **Multi-stage Docker build** | A Dockerfile pattern that uses multiple `FROM` statements to separate build-time dependencies from the runtime image, producing smaller, more secure final images. | Single-stage build (larger), Distroless base images, BuildKit cache mounts | Multi-stage separates build from runtime, reducing image size and attack surface. Final image contains only runtime dependencies and non-root user. |
| Local Dev | **Docker Compose** | A tool for defining and running multi-container Docker applications using a YAML file, used for local development and integration testing. | Kind/Minikube (local K8s), Tilt (K8s dev), Direct host execution | Docker Compose provides fast local iteration without Kubernetes overhead. Suitable for running app + database + dependencies during development. |

## Monitoring and Observability

| Category | Chosen | Formal Definition | Alternatives | Why This Choice |
|----------|--------|-------------------|-------------|-----------------|
| Metrics | **Prometheus** | An open-source time-series monitoring system that scrapes metrics, stores them in a TSDB, and evaluates alerting/recording rules using PromQL. | Datadog, CloudWatch, New Relic, VictoriaMetrics | Kubernetes-native ecosystem, transparent query model, low per-metric cost for self-managed operations. |
| Visualization | **Grafana** | A visualization and alerting platform that queries multiple data sources and renders operational dashboards and alerts. | Kibana-only dashboards, Datadog dashboards, CloudWatch dashboards | Multi-datasource flexibility and strong Prometheus integration for platform-wide observability. |
| Alert Routing | **Alertmanager routing** | Prometheus alert routing component that groups, deduplicates, inhibits, and routes alerts to receivers by policy. | Direct paging from each source | Centralized routing and deduplication improves noise control and on-call sustainability. |
| Alert Model | **Burn-rate alerting** | SLO-based alerting that detects rapid error-budget consumption over short and long windows. | Static threshold alerts only | Better alignment to reliability objectives, lower noise during transient anomalies. |
| Log Collection | **Fluent Bit** | Lightweight log processor/forwarder optimized for efficient Kubernetes node-level collection. | Fluentd (heavier, more plugins), Logstash, Vector | Lower resource footprint and strong Kubernetes adoption for node-level collection. |
| Log Storage | **Elasticsearch + Kibana** | Elasticsearch: distributed search and analytics engine for log storage and full-text search. Kibana: exploration and visualization interface for Elasticsearch data. | Loki + Grafana (label-based, cheaper), CloudWatch Logs, Splunk | Powerful full-text and structured search. Mature ecosystem for operational teams. |
| ILM | **Elasticsearch ILM** | Index Lifecycle Management policy that automates phase transitions (hot/warm/cold/delete) and retention behavior. | Manual retention scripts, unlimited hot retention | Controls storage cost while preserving operationally valuable data tiers. |

## AWS Platform

| Category | Chosen | Formal Definition | Alternatives | Why This Choice |
|----------|--------|-------------------|-------------|-----------------|
| Network Model | **VPC private subnet model** | Network segmentation where workloads run without direct public ingress and egress is controlled. | Fully public subnet model | Better attack-surface reduction and predictable governance. |
| AWS Telemetry | **CloudWatch-native telemetry** | AWS-managed metrics, logs, dashboards, and alarms stack for service observability. | Prometheus/Grafana (self-managed), Datadog, New Relic | Fast AWS integration, lower setup friction for AWS-centric environments. |
| Tracing | **X-Ray tracing** | AWS distributed tracing service for request path and latency diagnostics across integrated services. | OpenTelemetry + Tempo/Jaeger, Datadog APM | Native AWS integration and lower setup overhead for AWS-heavy environments. |
| SLO Alerting | **SLO-driven alerting** | Alert model aligned to service-level objectives and error-budget burn, not raw threshold noise. | Static threshold-only alerting | Better signal quality and incident prioritization linked to user impact. |

---

## Scoring Signals

When evaluating technology comparison answers:
- [ ] Candidate provides formal definitions before tradeoff discussion
- [ ] Candidate compares at least one concrete alternative with non-trivial tradeoffs
- [ ] Candidate ties choice to repo context and one or more pillars
- [ ] Candidate avoids tool tribalism and acknowledges operational constraints
- [ ] Candidate explains what the technology does in this specific project
