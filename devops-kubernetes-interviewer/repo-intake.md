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
Glob: **/*.json (limit to top-level and .claude/)
```

## Step 2: Terraform Analysis

For each `.tf` file found:

1. **Root modules**: Identify directories containing `backend.tf` or `terraform {}` blocks with backend config. Note the backend type (S3, local, etc.).
2. **Child modules**: Identify `modules/` directories. Read each module's `main.tf` to understand what it provisions.
3. **Bootstrap vs main**: Check for `bootstrap/` directory pattern.
4. **State partitioning**: Note how state is split (one state per module? per environment? bootstrap + main?).
5. **Key resources**: List critical resources (VPC, EKS, IAM roles, security groups, secrets, ECR, Route 53, CDN).
6. **Provider versions**: Read `versions.tf` files to note Terraform and provider version constraints.
7. **Variables and outputs**: Check for descriptions, types, sensitive flags.

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

## Step 4: Helm Analysis

Look for:

1. **Chart.yaml** files — identify charts, their versions, and dependencies.
2. **values.yaml** files — note layering (base values, environment overrides).
3. **Templates** — skim `templates/` directories for key resources.
4. **Helm releases managed by ArgoCD** — check ArgoCD Application manifests for Helm source types.

## Step 5: ArgoCD Analysis

Look for:

1. **Application CRDs**: YAML files with `kind: Application` or `kind: ApplicationSet`.
2. **App-of-apps pattern**: A parent Application that deploys other Applications.
3. **Sync policies**: `automated`, `selfHeal`, `prune` settings.
4. **Sync waves**: `argocd.argoproj.io/sync-wave` annotations.
5. **Repo credentials**: How SSH keys or tokens are managed.
6. **Helm sources**: Applications that use Helm chart sources.
7. **Directory sources**: Applications that point to plain YAML directories.

## Step 6: CI/CD Analysis

Look for:

1. **Pipeline definition**: Jenkinsfile, `.gitlab-ci.yml`, `.github/workflows/*.yml`.
2. **Stages**: Build, test, security scan, deploy — note the order.
3. **Credential management**: How secrets are injected into the pipeline.
4. **Deploy strategy**: Push-based (CI applies) vs pull-based (GitOps/ArgoCD).
5. **Test coverage**: Unit, integration, E2E — which exist?
6. **Artifact management**: Docker image tagging strategy, registry push.

## Step 7: Docker Analysis

Look for:

1. **Dockerfiles**: Multi-stage builds, base images, non-root users, security practices.
2. **docker-compose files**: Local dev and test environments.
3. **Image tagging strategy**: Semantic versioning, commit SHA, latest.

## Step 8: DNS / CDN / Networking

Look for:

1. **Route 53**: Hosted zones, record types, delegation.
2. **CloudFront**: Distributions, origins (S3, custom), cache behaviors, OAC.
3. **ACM certificates**: Validation method (DNS-01).
4. **ExternalDNS config**: Annotations on Ingress/Service resources.
5. **Ingress controllers**: nginx, ALB, etc.

## Step 9: Scripts and Docs

Look for:

1. **Shell scripts**: bootstrap, apply, destroy, utility scripts.
2. **Documentation**: Architecture docs, planning docs, runbooks.
3. **Diagrams**: `.drawio`, `.png`, `.svg` architecture diagrams.

## Step 10: Sibling Repos

If additional working directories are available:

1. Scan for the same artifacts (Dockerfiles, CI, Kubernetes manifests, Helm charts).
2. Note cross-repo relationships (e.g., app repo → infra repo → GitOps repo).
3. Include findings in the repo map.

---

## Output Format

Produce the table specified in `SKILL.md` Phase 1. For each layer, list:

- **Artifacts Found**: Count and type (e.g., "4 modules, 1 bootstrap, 6 secrets")
- **Key Paths**: The most important file paths (max 3-4 per layer)

After the table, categorize each interview round:

- **Available**: Repo has rich artifacts for this domain (questions will reference paths)
- **Limited**: Some artifacts found, mix of repo-grounded and generic questions
- **Generic**: No artifacts for this domain, questions will be rigorous but not repo-specific
- **Skipped**: Domain is irrelevant given the repo (rarely used — most domains apply to most infra repos)
