# Technology Comparisons

> This file maps every technology choice in the cowsay3d infrastructure to its alternatives.
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

| Chosen | Formal Definition | Alternatives | Why This Choice |
|--------|------------------|-------------|-----------------|
| **Terraform (HCL)** | An open-source Infrastructure as Code tool by HashiCorp that uses declarative configuration files (HCL) to provision and manage cloud resources through a plan/apply lifecycle. | **Pulumi** (imperative, real languages), **CloudFormation** (AWS-native, JSON/YAML), **CDK** (imperative wrapper for CFN), **Ansible** (procedural, config mgmt) | Declarative, cloud-agnostic, strong module ecosystem, explicit plan/apply cycle with state management. The project is AWS-only but Terraform's module system and conventions make it superior for maintainability. |
| **Terraform Modules (custom)** | Reusable, encapsulated Terraform configurations that accept inputs (variables) and produce outputs, used to compose infrastructure from building blocks. | **Terraform Registry community modules** (e.g., `terraform-aws-modules/vpc/aws`), **Terragrunt wrappers** | Custom modules give full control over resource configuration, naming, tagging, and lifecycle rules. Community modules add complexity, version churn, and may not match project conventions. |
| **S3 Backend (native locking)** | Terraform backend that stores state files in S3 with server-side encryption and uses S3's native conditional writes for state locking (introduced in Terraform 1.6+). | **S3 + DynamoDB** (legacy locking), **Terraform Cloud/HCP**, **Consul backend**, **local state** | Native S3 locking removes DynamoDB cost and complexity. S3 provides durability, versioning, and encryption. Local state is a non-starter for teams. |

## Container Orchestration

| Chosen | Formal Definition | Alternatives | Why This Choice |
|--------|------------------|-------------|-----------------|
| **Amazon EKS** | A managed Kubernetes service that runs the Kubernetes control plane across multiple AZs, handling upgrades, patching, and high availability of the API server and etcd. | **Self-managed K8s** (kubeadm), **ECS** (AWS-native containers), **k3s/k0s** (lightweight), **GKE/AKS** (other cloud) | Managed control plane eliminates operational burden. Deep AWS integration (IRSA, VPC-CNI, EBS CSI). The project needs Kubernetes-native tools (Helm, ArgoCD) which rules out ECS. |
| **VPC-CNI** | AWS's Container Network Interface plugin that assigns real VPC IP addresses to Pods, making them first-class VPC citizens with direct security group and routing support. | **Calico** (overlay, network policies), **Cilium** (eBPF-based), **Flannel** (simple overlay) | VPC-CNI gives Pods real VPC IPs — no overlay overhead, native security group support, and compatibility with AWS load balancers. Trade-off: IP address consumption is higher. |
| **EBS CSI Driver** | A Kubernetes Container Storage Interface driver that provisions and attaches Amazon EBS volumes as persistent storage for Pods, replacing the deprecated in-tree provisioner. | **EFS CSI** (shared filesystem), **In-tree provisioner** (deprecated), **Local volumes** | EBS provides block storage with snapshots and encryption. CSI driver is the Kubernetes-standard interface replacing deprecated in-tree plugins. Uses IRSA for secure AWS access. |

## GitOps & Deployment

| Chosen | Formal Definition | Alternatives | Why This Choice |
|--------|------------------|-------------|-----------------|
| **ArgoCD** | A declarative, GitOps-based continuous delivery tool for Kubernetes that continuously reconciles the desired state in Git with the live cluster state using a pull-based model. | **Flux CD** (CNCF, lighter), **Jenkins CD** (push-based), **Spinnaker** (multi-cloud, heavy) | Pull-based reconciliation, powerful UI for debugging sync issues, app-of-apps pattern for multi-app management, strong Helm integration. Flux is lighter but ArgoCD's UI and Application CRD model are more intuitive. |
| **App-of-apps pattern** | An ArgoCD pattern where a parent Application deploys child Application CRDs, enabling hierarchical management of multiple applications with ordered deployment via sync waves. | **ApplicationSets** (template-based generation), **Flat Applications** (each managed independently) | App-of-apps with sync waves gives explicit ordering (cert-manager before ingress-nginx before app). ApplicationSets are better for large-scale, homogeneous deployments across clusters — overkill for this project's 5 apps. |
| **Helm (via ArgoCD)** | A Kubernetes package manager that uses templated YAML charts with configurable values to package, version, and deploy applications as releases. | **Kustomize** (overlay-based), **Raw YAML**, **jsonnet**, **cdk8s** | Helm provides parameterized charts, release versioning, and rollback. ArgoCD natively supports Helm sources. Kustomize is simpler for small customizations but lacks the packaging/versioning model. |

## CI/CD

| Chosen | Formal Definition | Alternatives | Why This Choice |
|--------|------------------|-------------|-----------------|
| **Jenkins** | An open-source automation server that executes CI/CD pipelines defined as code (Jenkinsfile), with plugin-based extensibility for build, test, and deployment stages. | **GitLab CI** (integrated with GitLab), **GitHub Actions** (integrated with GitHub), **Tekton** (Kubernetes-native), **CircleCI/Travis** (SaaS) | Jenkins is self-hosted (full control), integrates with GitLab repos via webhooks, and supports complex multi-stage pipelines with parallel execution. Already deployed on an EC2 instance in this infrastructure. |
| **GitOps deploy (Jenkins → GitOps repo → ArgoCD)** | A deployment pattern where CI writes the new image tag to a GitOps repository, and ArgoCD detects the change and syncs the cluster — separating CI (build/test) from CD (deploy). | **Direct kubectl apply from CI** (push-based), **Helm upgrade from CI**, **ArgoCD image updater** | GitOps separation provides audit trail (Git history), declarative state, and ArgoCD's self-healing. Push-based CI deploys bypass ArgoCD's reconciliation and can cause drift. |

## Networking & DNS

| Chosen | Formal Definition | Alternatives | Why This Choice |
|--------|------------------|-------------|-----------------|
| **CloudFront** | AWS's global content delivery network (CDN) that caches and serves content from edge locations worldwide, reducing latency and offloading origin servers. | **Direct S3 hosting**, **Nginx on EC2/EKS**, **Cloudflare**, **ALB + S3** | CloudFront provides global caching, HTTPS termination, OAC for private S3 access, and HTTP/2+3 support. Direct S3 hosting lacks custom domain SSL and caching control. Cloudflare is an alternative but adds a third-party dependency outside the AWS ecosystem. |
| **Origin Access Control (OAC)** | A CloudFront security feature that allows CloudFront to access a private S3 bucket using signed requests, replacing the legacy Origin Access Identity (OAI). | **Origin Access Identity (OAI)** (legacy), **Public S3 bucket**, **S3 website endpoint** | OAC is the modern replacement for OAI — supports SSE-KMS, S3 bucket policies with conditions, and is the AWS-recommended approach. Public S3 is a security risk. |
| **Route 53** | AWS's scalable, highly available DNS service that supports domain registration, DNS routing, and health checks, with native integration into the AWS ecosystem. | **GoDaddy DNS** (registrar default), **Cloudflare DNS**, **External DNS providers** | Route 53 integrates natively with CloudFront aliases, ACM certificate validation, and ExternalDNS. The domain was migrated from GoDaddy for this integration. |
| **ExternalDNS** | A Kubernetes add-on that automatically creates DNS records in external DNS providers (like Route 53) based on Ingress and Service annotations, syncing Kubernetes state to DNS. | **Manual DNS records**, **Terraform-managed records**, **CoreDNS external plugin** | ExternalDNS automatically creates records when Ingress resources are created — truly GitOps-native DNS. Terraform-managed DNS is more rigid and requires a Terraform apply for every DNS change. |
| **cert-manager (DNS-01)** | A Kubernetes certificate management controller that automates issuance and renewal of TLS certificates from Let's Encrypt (or other CAs) using ACME protocol challenges. | **ACM only** (AWS-managed), **Manual Let's Encrypt (certbot)**, **Self-signed certs** | cert-manager automates certificate lifecycle within Kubernetes. DNS-01 validation works for wildcard certs and doesn't require HTTP ingress during validation. Uses IRSA to create Route 53 TXT records. |
| **NAT Gateway (single)** | An AWS-managed, highly available network address translation service that allows resources in private subnets to access the internet without being directly reachable from the internet. | **NAT Gateway per AZ** (HA), **NAT Instance** (self-managed EC2), **VPC endpoints only** | Single NAT gateway saves ~$32/month per additional gateway. Acceptable for non-production or cost-sensitive deployments. The tradeoff is that the AZ hosting the NAT Gateway is a SPOF for outbound internet access. |

## Secrets Management

| Chosen | Formal Definition | Alternatives | Why This Choice |
|--------|------------------|-------------|-----------------|
| **AWS Secrets Manager** | A managed secrets storage service that encrypts secrets at rest (KMS), provides fine-grained IAM access control, supports automatic rotation, and integrates with AWS services. | **AWS SSM Parameter Store** (simpler, cheaper), **HashiCorp Vault** (self-hosted, powerful), **Kubernetes Secrets** (native but base64 only) | Secrets Manager provides rotation, cross-service access (Jenkins + Kubernetes), and native IRSA integration. Parameter Store is cheaper but lacks rotation. Vault is powerful but adds operational overhead for a small team. |
| **External Secrets Operator (ESO)** | A Kubernetes operator that synchronizes secrets from external secret management systems (like AWS Secrets Manager) into native Kubernetes Secrets, keeping the source of truth outside the cluster. | **Sealed Secrets** (encrypted in Git), **SOPS** (encrypted files), **CSI Secrets Store Driver** (mount-based), **Vault Agent Sidecar** | ESO pulls secrets on a schedule and creates native K8s Secrets — works with any workload without sidecar changes. Sealed Secrets require encryption/decryption tooling. CSI driver requires volume mounts instead of env vars. |
| **IRSA** | IAM Roles for Service Accounts — an EKS feature that allows Kubernetes service accounts to assume AWS IAM roles via OpenID Connect (OIDC) federation, eliminating long-lived credentials. | **Node-level IAM roles** (too broad), **kiam/kube2iam** (intercept-based, deprecated patterns), **Static AWS credentials in Secrets** | IRSA provides pod-level, least-privilege AWS access using short-lived STS tokens. Node roles grant all Pods on a node the same permissions. kiam/kube2iam add latency and complexity. |

## Docker & Container

| Chosen | Formal Definition | Alternatives | Why This Choice |
|--------|------------------|-------------|-----------------|
| **ECR** | Amazon Elastic Container Registry — a fully managed Docker container registry that stores, manages, and deploys container images with integrated vulnerability scanning and IAM-based access control. | **Docker Hub** (public/SaaS), **GitLab Container Registry**, **GitHub Container Registry**, **Harbor** (self-hosted) | ECR integrates natively with EKS node roles (no separate registry credentials needed), provides scan-on-push, lifecycle policies, and lives in the same AWS region as the cluster (fast pulls). |
| **Multi-stage Docker build** | A Dockerfile pattern that uses multiple `FROM` statements to separate build-time dependencies from the runtime image, producing smaller, more secure final images. | **Single-stage build** (larger image), **Distroless base images**, **BuildKit cache mounts** | Multi-stage separates pip install from runtime, reducing image size and attack surface. The final image only contains runtime dependencies and the non-root user. |
