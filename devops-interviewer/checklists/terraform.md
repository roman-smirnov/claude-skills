`# Terraform Checklist

> **Terraform standards derived from `iac-terraform@devops-skills`.**
>
> This checklist is the source of truth for evaluating Terraform answers during the interview.
> When scoring Terraform questions, compare the candidate's answer against these standards.

---

## Module Structure (Mandatory)

Every Terraform module MUST contain:

- [ ] `main.tf` — Resource definitions
- [ ] `variables.tf` — Input variables (every variable has `description` and `type`)
- [ ] `outputs.tf` — Output values (every output has `description`)
- [ ] `versions.tf` — `required_version` and `required_providers` with version constraints

Should contain:

- [ ] `README.md` — Usage section with examples, inputs table, outputs table
- [ ] `examples/` — Working example configuration

## Variable Standards

- [ ] Every variable has a `description`
- [ ] Every variable has a `type` constraint
- [ ] Variables holding secrets (password, key, token, secret, uri with credentials) are marked `sensitive = true`
- [ ] Validation rules are used where applicable (e.g., CIDR format, allowed values)
- [ ] Default values are provided only when a sensible default exists

## Output Standards

- [ ] Every output has a `description`
- [ ] Outputs exposing secrets are marked `sensitive = true`
- [ ] Every module exposes at least one output

## Naming Conventions

- [ ] Terraform resource names: `snake_case`
- [ ] AWS resource names: `kebab-case` (pattern: `{project}-{environment}-{service}-{type}`)
- [ ] File names: `snake_case` with `.tf` extension
- [ ] Variable names: `snake_case`, descriptive

## State Management

- [ ] Remote backend with locking (S3 + DynamoDB or S3 native locking)
- [ ] State encryption enabled (`encrypt = true`)
- [ ] S3 versioning enabled for state bucket
- [ ] State files are NEVER committed to Git
- [ ] State is partitioned to minimize blast radius (bootstrap vs main, or per-service)
- [ ] State is never manually edited

## Pre-Commit Pipeline (Mandatory Order)

1. [ ] `terraform fmt -recursive` (or `terraform fmt -check` in CI)
2. [ ] `terraform validate`
3. [ ] `tflint --module`
4. [ ] `checkov -d .` (security scanning)
5. [ ] Module validation (descriptions, required files)
6. [ ] Documentation generation (`terraform-docs`)

## Security

- [ ] Never hardcode credentials in `.tf` files
- [ ] Never use root AWS credentials
- [ ] All S3 buckets have encryption enabled
- [ ] All S3 buckets have public access blocked (unless explicitly required)
- [ ] EBS volumes are encrypted
- [ ] IAM follows least privilege
- [ ] Security groups are restrictive (no 0.0.0.0/0 ingress unless justified)
- [ ] Sensitive variables and outputs are flagged
- [ ] `prevent_destroy` on critical resources (state bucket, ECR, DNS zones, secrets)

## Lifecycle and Safety

- [ ] `prevent_destroy` on resources that must survive destroy/apply cycles
- [ ] `create_before_destroy` on resources where replacement must be zero-downtime
- [ ] `ignore_changes` used sparingly and with documented justification
- [ ] `moved` blocks used for refactoring instead of destroy+recreate

## Provider Management

- [ ] Provider versions pinned with pessimistic constraints (`~> X.Y`)
- [ ] Provider lock file (`.terraform.lock.hcl`) committed to Git
- [ ] Provider aliases used for multi-region resources (e.g., ACM in `us-east-1` for CloudFront)

## Workflow

- [ ] Always run `terraform plan` before `terraform apply`
- [ ] Never apply without reviewing the plan
- [ ] `terraform plan` output is saved as artifact in CI
- [ ] Production applies require approval gate
- [ ] Test in non-prod before applying to prod

## Cost Awareness

- [ ] NAT gateway strategy documented (single vs per-AZ)
- [ ] Instance types justified (cost vs performance)
- [ ] ECR lifecycle policies to limit stored images
- [ ] S3 lifecycle policies where appropriate
- [ ] Resources tagged for cost allocation (`Project`, `ManagedBy`, `Environment`)

## Drift Detection

- [ ] Regular `terraform plan` to detect drift
- [ ] `terraform plan -detailed-exitcode` in CI for automated drift checks
- [ ] Process for handling manual changes (import or revert)
- [ ] Dashboard or alerting for drift status

## Import and Migration

- [ ] `terraform import` used for adopting existing resources
- [ ] `terraform state mv` for reorganizing state
- [ ] `moved` blocks preferred over state surgery for refactors
- [ ] Import scripts documented (e.g., `bootstrap.sh --import`)

---

## Scoring Application

When evaluating a Terraform answer:

1. Count how many of the relevant checklist items the candidate addressed.
2. Check whether the candidate proactively mentioned items not prompted (expert signal).
3. Flag dangerous omissions (e.g., no state locking, no encryption, hardcoded secrets).
4. Apply the rubric from `rubrics.md` — Terraform section.

| Coverage | Score |
|----------|-------|
| 0-20% of relevant items | 1 (Awareness) |
| 20-50% | 2 (Competent) |
| 50-80% | 3 (Proficient) |
| 80%+ with proactive depth | 4 (Expert) |
