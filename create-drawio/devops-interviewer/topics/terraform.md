# Terraform & IaC — Interview Module

## Enhancement Instructions

- Evaluate **state management maturity**: remote backend security, locking, encryption, partitioning strategy, drift detection, and recovery from state corruption.
- Probe **module design discipline**: mandatory file structure, variable/output standards, naming conventions, versioning, and documentation.
- Score higher when the candidate connects Terraform decisions to **blast radius** reasoning — why `prevent_destroy`, why bootstrap separation, why state partitioning.
- Evaluate **pre-commit pipeline** knowledge: fmt → validate → lint → scan → docs, in the correct order and with correct tool names.
- Probe **lifecycle rule** understanding: `prevent_destroy`, `create_before_destroy`, `ignore_changes`, `moved` blocks — when each is appropriate and the risks of misuse.

---

## Evaluation Checklist

### Module Structure (Mandatory)

Every Terraform module MUST contain:

- [ ] `main.tf` — Resource definitions
- [ ] `variables.tf` — Input variables (every variable has `description` and `type`)
- [ ] `outputs.tf` — Output values (every output has `description`)
- [ ] `versions.tf` — `required_version` and `required_providers` with version constraints

Should contain:

- [ ] `README.md` — Usage section with examples, inputs table, outputs table
- [ ] `examples/` — Working example configuration

### Variable Standards

- [ ] Every variable has a `description`
- [ ] Every variable has a `type` constraint
- [ ] Variables holding secrets are marked `sensitive = true`
- [ ] Validation rules are used where applicable (e.g., CIDR format, allowed values)
- [ ] Default values are provided only when a sensible default exists

### Output Standards

- [ ] Every output has a `description`
- [ ] Outputs exposing secrets are marked `sensitive = true`
- [ ] Every module exposes at least one output

### Naming Conventions

- [ ] Terraform resource names: `snake_case`
- [ ] AWS resource names: `kebab-case` (pattern: `{project}-{environment}-{service}-{type}`)
- [ ] File names: `snake_case` with `.tf` extension
- [ ] Variable names: `snake_case`, descriptive

### State Management

- [ ] Remote backend with locking (S3 + DynamoDB or S3 native locking)
- [ ] State encryption enabled (`encrypt = true`)
- [ ] S3 versioning enabled for state bucket
- [ ] State files are NEVER committed to Git
- [ ] State is partitioned to minimize blast radius (bootstrap vs main, or per-service)
- [ ] State is never manually edited

### Pre-Commit Pipeline (Mandatory Order)

1. [ ] `terraform fmt -recursive` (or `terraform fmt -check` in CI)
2. [ ] `terraform validate`
3. [ ] `tflint --module`
4. [ ] `checkov -d .` (security scanning)
5. [ ] Module validation (descriptions, required files)
6. [ ] Documentation generation (`terraform-docs`)

### Security

- [ ] Never hardcode credentials in `.tf` files
- [ ] Never use root AWS credentials
- [ ] All S3 buckets have encryption enabled
- [ ] All S3 buckets have public access blocked (unless explicitly required)
- [ ] EBS volumes are encrypted
- [ ] IAM follows least privilege
- [ ] Security groups are restrictive (no 0.0.0.0/0 ingress unless justified)
- [ ] Sensitive variables and outputs are flagged
- [ ] `prevent_destroy` on critical resources (state bucket, ECR, DNS zones, secrets)

### Lifecycle and Safety

- [ ] `prevent_destroy` on resources that must survive destroy/apply cycles
- [ ] `create_before_destroy` on resources where replacement must be zero-downtime
- [ ] `ignore_changes` used sparingly and with documented justification
- [ ] `moved` blocks used for refactoring instead of destroy+recreate

### Provider Management

- [ ] Provider versions pinned with pessimistic constraints (`~> X.Y`)
- [ ] Provider lock file (`.terraform.lock.hcl`) committed to Git
- [ ] Provider aliases used for multi-region resources (e.g., ACM in `us-east-1` for CloudFront)

### Workflow

- [ ] Always run `terraform plan` before `terraform apply`
- [ ] Never apply without reviewing the plan
- [ ] `terraform plan` output is saved as artifact in CI
- [ ] Production applies require approval gate
- [ ] Test in non-prod before applying to prod

### Drift Detection

- [ ] Regular `terraform plan` to detect drift
- [ ] `terraform plan -detailed-exitcode` in CI for automated drift checks
- [ ] Process for handling manual changes (import or revert)
- [ ] Dashboard or alerting for drift status

### Import and Migration

- [ ] `terraform import` used for adopting existing resources
- [ ] `terraform state mv` for reorganizing state
- [ ] `moved` blocks preferred over state surgery for refactors
- [ ] Import scripts documented

### Workspaces (NEW)

- [ ] Understands Terraform workspaces for environment separation
- [ ] Can explain workspace vs directory-based environment strategy tradeoffs
- [ ] Knows `terraform.workspace` interpolation for environment-specific config
- [ ] Can explain when workspaces are appropriate vs dedicated state files

### Testing (NEW)

- [ ] Knows Terratest for integration testing of modules
- [ ] Understands `terraform-compliance` for policy-as-code (BDD-style)
- [ ] Can explain OPA/Conftest for Terraform plan validation
- [ ] Knows `infracost` for cost estimation in PR comments
- [ ] Understands `terraform test` (native testing framework)

### Troubleshooting (Key Error Patterns)

#### State Issues
- [ ] **State lock error**: Verify no concurrent operations, check DynamoDB lock table for S3+DynamoDB backends, use `terraform force-unlock <LOCK_ID>` with caution
- [ ] **State drift**: Review with `terraform plan -detailed-exitcode`, resolve with `terraform apply -refresh-only` or `terraform apply` to revert. **Prevention: policy against manual changes, regular plan-based drift detection.**
- [ ] **State corruption**: Restore from S3 versioned backup, rebuild state with `terraform import` as last resort. **Prevention: pin Terraform version, enable S3 versioning, never manually edit state.**

#### Provider Issues
- [ ] **Provider version conflict**: Specify version constraints, clean cache, sync lock file across platforms
- [ ] **Authentication failure**: Verify with `aws sts get-caller-identity`, check credential order (env vars → shared credentials → IAM role), use profiles or assume-role. **Prevention: use IAM roles in CI/CD, implement OIDC.**

#### Resource Issues
- [ ] **Resource already exists**: Import with `terraform import`, verify config matches with `terraform plan`
- [ ] **Dependency errors**: Add explicit `depends_on`, use `data` sources for existing resources
- [ ] **Timeout errors**: Increase with `timeouts { create = "60m" }`, check resource status manually

#### Module Issues
- [ ] **Source not found**: Verify URL/path, configure Git auth for private repos, clear module cache
- [ ] **Version conflicts**: Run `terraform init -upgrade`, pin module versions

#### Performance Issues
- [ ] **Slow plans/applies**: Split state into smaller units, use `-target`, optimize data sources, increase parallelism

#### Quick Diagnostic Workflow
- [ ] Read full error → check recent changes → verify versions → check state → test auth → review logs → isolate with `-target`

### Cost Optimization

#### Resource Right-Sizing
- [ ] Start with smallest reasonable instance type, scale up based on data
- [ ] Use auto-scaling groups instead of over-provisioning
- [ ] Environment-appropriate sizing

#### Networking Costs
- [ ] VPC endpoints for S3, DynamoDB, ECR to avoid NAT Gateway data transfer charges
- [ ] Co-locate resources in same region/AZ to minimize cross-AZ transfer
- [ ] NAT Gateway strategy documented with tradeoff

#### Storage Lifecycle
- [ ] S3 lifecycle policies: transition to Standard-IA → Glacier → Deep Archive → expire
- [ ] EBS volume optimization: gp3 over gp2, `delete_on_termination = true`
- [ ] ECR lifecycle policies to limit stored images

#### Scheduled Resources
- [ ] Non-production shutdown (Lambda + EventBridge to stop/start instances on schedule)
- [ ] Tag resources for scheduling (`Schedule = "business-hours"`)

#### Cost Tagging and Monitoring
- [ ] Comprehensive tagging strategy: `CostCenter`, `Project`, `Environment`, `Owner`, `ManagedBy`
- [ ] Enforce tags with AWS Config rules
- [ ] AWS Budgets with threshold alerts
- [ ] Cost anomaly detection
- [ ] `infracost` in CI/CD to estimate cost impact before apply

### Diagnostic Commands

```bash
# Formatting and validation
terraform fmt -check -recursive
terraform validate
terraform plan -detailed-exitcode

# State inspection
terraform state list
terraform state show <resource>
terraform show

# Debugging
TF_LOG=DEBUG TF_LOG_PATH=terraform-debug.log terraform plan
terraform providers
terraform version

# Import and migration
terraform import <resource> <id>
terraform state mv <source> <destination>
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

1. "Your S3 state backend at `{path}` loses the lock mechanism. Two engineers run `terraform apply` simultaneously. What happens to your state file, and how do you recover?"
2. "Someone manually deletes the `{resource}` resource in the AWS console but the Terraform state still references it. Walk me through what happens on the next `terraform plan` and how you resolve it."
3. "The `prevent_destroy` lifecycle rule on `{resource}` in `{path}` is accidentally removed and someone runs `terraform destroy`. What is the blast radius and how would you recover?"
4. (generic) "What happens if your Terraform provider version is unpinned and a breaking change ships upstream?"

### How Would You Debug...

5. "You run `terraform plan` and see 15 unexpected changes to resources you didn't touch. The state file is at `{path}`. Walk me through your debugging process."
6. "A new engineer adds a module but `terraform validate` passes while `terraform plan` fails with a cycle error. How do you diagnose and fix this?"
7. (generic) "Your CI pipeline shows `terraform fmt -check` failing. What does this mean and how do you enforce formatting across a team?"
8. (generic) "`terraform plan` is taking over 5 minutes and slowing your CI pipeline. What are your optimization strategies?"

### Tradeoff

9. "In `{path}`, you use a single NAT gateway instead of one per AZ. What are the tradeoffs? When would you change this decision?"
10. "Your bootstrap phase at `bootstrap/` is separate from the main Terraform root. Why partition state this way instead of using one state file for everything?"
11. "You use `data` sources to read secrets from Secrets Manager instead of managing secret values in Terraform. What are the security and operational tradeoffs?"
12. (generic) "Terraform workspaces vs separate directories for environment management. When would you use each approach?"

### Technology Comparison

13. "[DEFINE] What is Terraform and what does it do in this project? Why Terraform instead of Pulumi, CloudFormation, or CDK? What specific advantages does the declarative HCL approach give you for this EKS-based infrastructure?"
14. "[DEFINE] Your state backend uses S3 with native locking. What is Terraform state locking? Why S3 native locking instead of DynamoDB-based locking? When was native S3 locking introduced and what limitation did it remove?"
15. "[DEFINE] You use Terraform modules for VPC, EKS, IAM, ArgoCD, and CDN. What is a Terraform module? Why write custom modules instead of using community modules from the Terraform Registry (e.g., `terraform-aws-modules/vpc/aws`)?"

### Best Practice Pillar

16. **(Security)** "Your Terraform state file contains sensitive data (IRSA role ARNs, OIDC endpoints, VPC IDs). What security measures protect the state backend at `{path}`? What would an attacker learn from your state file?"
17. **(Cost)** "Walk me through the cost implications of your Terraform architecture: single NAT gateway, t3a.medium nodes, CloudFront PriceClass_200, ECR lifecycle policy keeping 10 images. What's the estimated monthly cost and where would you cut first?"
18. **(Availability)** "Your bootstrap stack at `bootstrap/` has `prevent_destroy` on critical resources. From an availability perspective, what happens to the main stack if the bootstrap state file is corrupted?"
19. **(Security)** "Your pre-commit pipeline includes `checkov` for security scanning. What categories of misconfigurations does it catch? What is the blast radius if a security scan is skipped?"

### Bonus Pool

20. "Explain the module structure convention: why must every module have `main.tf`, `variables.tf`, `outputs.tf`, and `versions.tf`?"
21. "What should a pre-commit pipeline for Terraform include, and in what order? (Hint: think fmt, validate, lint, scan.)"
22. "When would you use `terraform import` vs `terraform state mv` vs `moved` blocks? Give a concrete refactoring scenario for each."
23. "Your `{module}` module has variables without descriptions. Why is this a problem?"
24. (generic) "How would you test a Terraform module before promoting it to production? Compare Terratest, `terraform test`, and policy-as-code approaches."
25. (generic) "What are dynamic blocks in Terraform? When are they appropriate and when do they make code harder to understand?"
