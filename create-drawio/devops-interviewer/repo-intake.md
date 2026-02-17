# Repo Intake Procedure

This document defines exactly what to scan and how to produce the repo map. All operations are **read-only**.

---

## Step 1: Directory Structure

Use `Glob` and `Read` to discover top-level layout:

```
Glob: **/*.tf
Glob: **/*.yaml
Glob: **/*.yml
Glob: **/Dockerfile*
Glob: **/Jenkinsfile*
Glob: **/.gitlab-ci*
Glob: **/.github/workflows/*
Glob: **/Makefile
Glob: **/*.sh
Glob: **/Chart.yaml
Glob: **/values*.yaml
Glob: **/kustomization.yaml
Glob: **/argocd/**/*
Glob: **/helmfile.yaml
Glob: **/.sops.yaml
Glob: **/*prometheus*
Glob: **/*alertmanager*
Glob: **/*grafana*
Glob: **/*dashboard*.json
Glob: **/*fluent-bit*
Glob: **/*fluentd*
Glob: **/*elasticsearch*
Glob: **/*kibana*
Glob: **/*slo*
Glob: **/*runbook*
Glob: **/*iam*
Glob: **/*vpc*
Glob: **/*cloudfront*
Glob: **/*cloudwatch*
Glob: **/*budget*
Glob: **/*cost*
Glob: **/*.md
Glob: **/*.json (limit to top-level and .claude/)
```

Capture top-level directories tied to infrastructure concerns (`infra/`, `terraform/`, `platform/`, `monitoring/`, `logging/`, `observability/`, `environments/`, `modules/`, `ops/`, `alerts/`, `dashboards/`).

---

## Step 2: Terraform Analysis

For each `.tf` file found:

1. **Root modules**: Identify directories containing `backend.tf` or `terraform {}` blocks with backend config. Note the backend type (S3, local, etc.).
2. **Child modules**: Identify `modules/` directories. Read each module's `main.tf` to understand what it provisions.
3. **Bootstrap vs main**: Check for `bootstrap/` directory pattern.
4. **State partitioning**: Note how state is split (one state per module? per environment? bootstrap + main?).
5. **Key resources**: List critical resources (VPC, EKS, IAM roles, security groups, secrets, ECR, Route 53, CDN).
6. **Provider versions**: Read `versions.tf` files to note Terraform and provider version constraints.
7. **Variables and outputs**: Check for descriptions, types, sensitive flags.

---

## Step 3: Kubernetes Analysis

Look for:

1. **EKS configuration**: Node groups, instance types, Kubernetes version, addons (vpc-cni, coredns, kube-proxy, ebs-csi-driver).
2. **IRSA roles**: Service accounts with IAM role annotations.
3. **Namespaces**: Infer from IRSA configs and Helm values.
4. **Kubernetes manifests**: Any `.yaml` files that are Kubernetes resources (Deployments, Services, ConfigMaps, etc.).
5. **Kustomize**: `kustomization.yaml` files.
6. **Resource requests/limits**: Check if pod specs define CPU/memory requests and limits.
7. **Probes**: Look for liveness, readiness, and startup probe configurations.
8. **Network policies**: Any NetworkPolicy resources defined.
9. **RBAC**: Roles, ClusterRoles, RoleBindings, ClusterRoleBindings.
10. **Storage**: PVCs, StorageClasses, CSI driver configurations.
11. **Scheduling controls**: Node selectors, affinity, taints/tolerations, topology spread constraints.
12. **Reliability controls**: PDBs, rollout strategy, disruption handling.

---

## Step 4: Helm Analysis

Look for:

1. **Chart.yaml** files — identify charts, their versions, and dependencies.
2. **values.yaml** files — note layering (base values, environment overrides).
3. **Templates** — skim `templates/` directories for key resources.
4. **Helm releases managed by ArgoCD** — check ArgoCD Application manifests for Helm source types.
5. **Kustomize overlays** — patch strategy and where Helm vs Kustomize is used.

---

## Step 5: ArgoCD / GitOps Analysis

Look for:

1. **Application CRDs**: YAML files with `kind: Application` or `kind: ApplicationSet`.
2. **App-of-apps pattern**: A parent Application that deploys other Applications.
3. **Sync policies**: `automated`, `selfHeal`, `prune` settings.
4. **Sync waves**: `argocd.argoproj.io/sync-wave` annotations.
5. **Sync options**: `CreateNamespace`, `ServerSideApply`, etc.
6. **Repo credentials**: How SSH keys or tokens are managed.
7. **Helm sources**: Applications that use Helm chart sources.
8. **Directory sources**: Applications that point to plain YAML directories.
9. **Argo projects / RBAC boundaries**: `AppProject`, destination/source restrictions.
10. **Drift and diff settings**: ignore differences, compare options.
11. **GitOps topology patterns**: `apps/`, `clusters/`, `infrastructure/`, `environments/`, base/overlay layouts, promotion conventions.

---

## Step 6: CI/CD Analysis

Look for:

1. **Pipeline definition**: Jenkinsfile, `.gitlab-ci.yml`, `.github/workflows/*.yml`.
2. **Stages**: Build, test, security scan, deploy — note the order.
3. **Credential management**: How secrets are injected into the pipeline.
4. **Deploy strategy**: Push-based (CI applies) vs pull-based (GitOps/ArgoCD).
5. **Test coverage**: Unit, integration, E2E — which exist?
6. **Artifact management**: Docker image tagging strategy, registry push.
7. **GitOps handoff model**: CI updates Git manifest/tag, ArgoCD reconciles from Git.
8. **Promotion strategy**: dev → staging → prod via PR, automation gates, manual approvals.
9. **Anti-patterns to note**: Direct `kubectl apply` in CI, no promotion gate before production.

---

## Step 7: Docker Analysis

Look for:

1. **Dockerfiles**: Multi-stage builds, base images, non-root users, security practices.
2. **docker-compose files**: Local dev and test environments.
3. **Image tagging strategy**: Semantic versioning, commit SHA, latest.

---

## Step 8: AWS Infrastructure Analysis

Look for:

1. **IAM model**: Roles, policies, trust relationships, instance profiles, IRSA/OIDC, permission boundaries, cross-account assumptions.
2. **Networking**: VPC topology (CIDRs, public/private subnets, AZ distribution), route tables, NAT gateways, security groups/NACLs, VPC endpoints.
3. **Compute**: Runtime choices (EKS/ECS/Lambda/EC2), node/task sizing, scaling configuration, availability controls.
4. **Data and storage**: S3 buckets (policies, encryption, versioning, lifecycle), RDS/Aurora, DynamoDB, EBS/EFS, backup and DR strategy.
5. **Edge and DNS**: CloudFront distributions (origins, OAC, cache policies), ALB/NLB, Route 53 hosted zones, ACM certificates, WAF.
6. **Security controls**: KMS keys, encryption posture, CloudTrail, audit controls, policy-as-code (SCP, Config rules, OPA/Kyverno).
7. **Cost and governance**: Tagging strategy, lifecycle controls, budget/anomaly alerts, compute purchase strategy hints (spot/savings/reserved).

---

## Step 9: Observability Analysis

Look for:

1. **Prometheus**: Deployment/config manifests, ServiceMonitor/PodMonitor, scrape jobs, retention/storage, recording rules, federation/remote-write, cardinality controls.
2. **Alerting**: PrometheusRule resources, alert quality (severity, `for` clause, runbook links), burn-rate patterns, Alertmanager routing.
3. **Grafana**: Dashboard JSON/provisioning, data sources, folder strategy, dashboard quality.
4. **Logging collectors**: Fluent Bit/Fluentd/Logstash configs (inputs, parsers, filters, outputs), DaemonSet scope, backpressure/retry settings.
5. **Elasticsearch**: Cluster/index configuration, index templates, ILM policies (hot/warm/cold/delete), shard/replica decisions.
6. **Kibana**: Index patterns/data views, dashboards, saved searches.
7. **SLO artifacts**: SLI/SLO/SLA definitions, error budget policies, burn-rate rules, incident runbooks, RTO/RPO.
8. **CloudWatch**: Metrics, dashboards, alarms, log retention.

---

## Step 10: Secrets Management

Look for:

1. **AWS Secrets Manager**: Secret resources and access patterns.
2. **External Secrets Operator**: `ExternalSecret`, `SecretStore`, `ClusterSecretStore` resources.
3. **SOPS**: `.sops.yaml`, encrypted manifests.
4. **Sealed Secrets**: SealedSecret resources.
5. **Plain secret risks**: `kind: Secret` with plaintext data in Git.
6. **Workload identity**: IRSA/OIDC annotations on service accounts.

---

## Step 11: DNS / CDN / Networking

Look for:

1. **Route 53**: Hosted zones, record types, delegation.
2. **CloudFront**: Distributions, origins (S3, custom), cache behaviors, OAC.
3. **ACM certificates**: Validation method (DNS-01).
4. **ExternalDNS config**: Annotations on Ingress/Service resources.
5. **Ingress controllers**: nginx, ALB, etc.

---

## Step 12: Scripts and Docs

Look for:

1. **Shell scripts**: Bootstrap, apply, destroy, utility scripts.
2. **Documentation**: Architecture docs, planning docs, runbooks.
3. **Diagrams**: `.drawio`, `.png`, `.svg` architecture diagrams.

---

## Step 13: Sibling Repos

If additional working directories are available:

1. Scan for the same artifacts (Dockerfiles, CI, Kubernetes manifests, Helm charts).
2. Note cross-repo relationships (e.g., app repo → infra repo → GitOps repo).
3. Include findings in the repo map.

---

## Output Format

Produce the table specified in `SKILL.md` Phase 1. For each layer, list:

- **Artifacts Found**: Count and type (e.g., "4 modules, 1 bootstrap, 6 secrets")
- **Key Paths**: The most important file paths (max 3-4 per layer)

After the table, classify each topic for the interview:

- **Available**: Repo has rich artifacts for this domain (questions will reference paths)
- **Limited**: Some artifacts found, mix of repo-grounded and generic questions
- **Generic**: No artifacts for this domain, questions will be rigorous but not repo-specific
- **Skipped**: Domain is irrelevant given the repo (rarely used)

When artifacts are absent, explicitly state that absence. Never fabricate file paths. Only reference paths confirmed during intake. All operations are read-only.
