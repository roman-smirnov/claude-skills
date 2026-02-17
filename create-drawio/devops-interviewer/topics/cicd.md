# CI/CD — Interview Module

## Enhancement Instructions

- Evaluate **pipeline design maturity**: stage ordering (checkout → build → test → scan → push → deploy/trigger), failure handling, parallelization, and credential management.
- Probe **credential management** depth: Jenkins credentials store vs Kubernetes secrets vs Secrets Manager, `withCredentials` binding, blast radius if Jenkins is compromised.
- Probe **artifact lifecycle** discipline: build → tag → push → scan → deploy, image immutability, digest pinning, and ECR lifecycle policies.
- Score higher when the candidate connects CI to CD boundary: push-based CI produces the artifact, pull-based GitOps deploys it. The candidate should explain why CI should NOT run `kubectl apply` directly.
- Evaluate **Docker build quality**: multi-stage builds, layer caching, non-root user, security scanning, `.dockerignore`, and image tagging strategy.

---

## Evaluation Checklist

### Jenkins Pipeline Design

- [ ] Understands declarative vs scripted pipeline syntax
- [ ] Can explain stage ordering: checkout → build → test → security scan → push → deploy/trigger
- [ ] Understands shared libraries and pipeline reuse
- [ ] Can explain agent architecture (controller vs agents, ephemeral agents)
- [ ] Knows credential management patterns (Jenkins credentials store, `withCredentials` binding)
- [ ] Can explain when to use pipeline vs freestyle vs multibranch
- [ ] Understands Jenkinsfile triggers (SCM polling, webhooks, cron)
- [ ] Can explain parallel stages and when to use them
- [ ] Understands post-build actions (always, success, failure, cleanup)

### Jenkins Governance and Reliability

- [ ] Uses Declarative Pipeline by default unless scripted behavior is justified
- [ ] Includes `options` such as `timestamps`, `disableConcurrentBuilds`, and `buildDiscarder` where needed
- [ ] Uses stage or global timeouts for long-running operations
- [ ] Understands Jenkins Configuration as Code (JCasC) for reproducible controller configuration
- [ ] Can explain plugin version governance and upgrade safety checks
- [ ] Can explain branch and promotion strategy (PR, main, release, prod)
- [ ] Can identify architecture bottlenecks (queue pressure, single controller risk, agent starvation)
- [ ] Understands shared library role (`vars/`, `src/`) and safe reuse boundaries
- [ ] Can explain test layering (lint/unit/integration/e2e) and gating policy
- [ ] Can identify risks of rebuild-at-deploy versus promote-built-artifact
- [ ] Can identify where flaky tests undermine delivery confidence

### Docker Build Quality

- [ ] Understands multi-stage builds and why they reduce image size
- [ ] Can explain Docker layer caching and how to optimize for CI
- [ ] Knows non-root user pattern (USER directive, numeric UID)
- [ ] Can explain `.dockerignore` purpose and common entries
- [ ] Understands image tagging strategy (commit SHA, semantic versioning, never `latest` in prod)
- [ ] Knows Docker Compose purpose (local dev, integration testing)
- [ ] Can explain healthcheck directive in Dockerfile vs Kubernetes probes
- [ ] Understands build args vs environment variables and when to use each
- [ ] Can explain `COPY --from=builder` pattern and artifact extraction

### Pipeline Security

- [ ] Secrets never hardcoded in Jenkinsfile or Dockerfile
- [ ] Understands image scanning integration (Trivy, Snyk, ECR scan-on-push)
- [ ] Can explain SBOM generation and software supply chain security
- [ ] Knows least-privilege for CI service accounts (Jenkins IAM role scoped to ECR push + S3)
- [ ] Can explain credential rotation impact on CI pipelines
- [ ] Understands pipeline audit trail (build logs, artifact provenance)
- [ ] Knows how to prevent secret leakage in build logs (masking, `set +x`)

### Artifact Management

- [ ] Understands ECR lifecycle policies (keep N images, expire untagged)
- [ ] Can explain image immutability (once pushed, never overwrite same tag)
- [ ] Knows registry authentication patterns (ECR login, docker-credential-ecr-login)
- [ ] Can explain artifact promotion (dev → staging → prod with same image, different config)
- [ ] Understands image digest vs tag for reproducibility

### CI/CD Integration with GitOps

- [ ] Understands CI → CD boundary (CI pushes image, ArgoCD detects and deploys)
- [ ] Can explain why CI should NOT run `kubectl apply` directly
- [ ] Knows image tag update patterns (CI updates values.yaml/kustomization → Git commit → ArgoCD sync)
- [ ] Can explain rollback in CI+GitOps model (Git revert, ArgoCD rollback, or both)
- [ ] Understands the handoff: CI is push-based, CD is pull-based

### Docker Compose for Development

- [ ] Understands service definitions, `depends_on` ordering, healthchecks
- [ ] Can explain volumes and bind mounts for local development
- [ ] Knows network modes and service discovery in Compose
- [ ] Can compare Docker Compose vs Kind/Minikube/Tilt for local K8s development

### Jenkins Troubleshooting Workflow (Mandatory Sequence)

1. [ ] Confirm scope and impact (single stage/job vs platform-wide)
2. [ ] Check queue/executor/agent availability and recent platform changes
3. [ ] Inspect failing stage logs and related environment differences
4. [ ] Validate credentials/identity path for external calls
5. [ ] Validate artifact integrity and downstream deployment signals
6. [ ] Apply mitigation and verify with rerun or targeted rollback
7. [ ] Record preventive actions to reduce recurrence

### Common Jenkins Failure Patterns

- [ ] Agent offline / label mismatch → queued jobs never start
- [ ] Workspace contention with concurrent builds → nondeterministic failures
- [ ] Missing timeout → hung build consumes executors indefinitely
- [ ] Credential ID mismatch → auth failures in deploy/publish stages
- [ ] Plugin incompatibility → sudden pipeline regression after updates
- [ ] Missing artifact handoff → deploy stage cannot find expected build output

### CI/CD Troubleshooting

- [ ] Can diagnose Jenkins build failures (agent connectivity, plugin issues, workspace cleanup)
- [ ] Can debug Docker build failures (cache invalidation, missing dependencies, network issues)
- [ ] Can troubleshoot ECR push/pull failures (authentication, permissions, network)
- [ ] Can isolate pipeline timing issues (slow builds, resource contention, artifact upload)

### Diagnostic Commands

```bash
# Jenkins
curl -s http://jenkins:8080/api/json               # Jenkins status
curl -s http://jenkins:8080/job/<job>/lastBuild/api/json  # Last build info

# Docker
docker build --target <stage> .                     # Build specific stage
docker history <image>                              # Layer inspection
docker inspect <container>                          # Container details
docker-compose config                               # Validate compose file
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# ECR
aws ecr describe-images --repository-name <repo>
aws ecr get-lifecycle-policy --repository-name <repo>
aws ecr describe-image-scan-findings --repository-name <repo> --image-id imageTag=<tag>

# Build debugging
docker build --no-cache --progress=plain .          # Verbose build output
docker run --rm -it <image> sh                      # Inspect image contents
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

1. "Your Jenkinsfile at `{path}` uses `withCredentials` to bind ECR credentials. The IAM role attached to the Jenkins EC2 instance loses ECR push permissions. What fails, what still works, and how do you diagnose it?"
2. "Your Docker multi-stage build at `{path}` uses `COPY --from=builder`. The builder stage fails silently (exits 0 but produces no artifact). What does the final image contain and how do you catch this in CI?"
3. "Someone pushes a Docker image tagged `latest` to ECR and ArgoCD's Helm values reference `latest`. What happens on the next ArgoCD sync? Why is this dangerous?"
4. (generic) "Your CI pipeline succeeds but the deployed application is broken. What gap in your testing strategy allowed this?"
5. "CI publishes an image tag but fails to update GitOps manifests in the second pipeline stage. What state does the system end up in, and how do you detect this?"
6. "Digest pinning is removed and `latest` is reintroduced for a critical service. What reproducibility problems appear across environments?"
7. "The Jenkins controller becomes unavailable for 30 minutes. What stops immediately, what continues, and what recovers automatically?"
8. "Credentials in `{path}` are injected as plain environment variables without binding/masking safeguards. What is the blast radius?"

### How Would You Debug...

9. "Jenkins builds are failing with 'no space left on device'. The Jenkins agent is an EC2 instance. Walk me through your diagnosis."
10. "A Docker image that builds locally fails in Jenkins CI with a dependency resolution error. What environment differences do you check?"
11. "ECR image pulls from EKS are intermittently failing with `unauthorized`. What do you check?"
12. (generic) "Your CI pipeline takes 25 minutes. Walk me through how you would optimize it."
13. "Pipeline succeeded, but ArgoCD never deploys the new version. Walk through your end-to-end debugging path from Jenkins to Git to ArgoCD."
14. "A rollback commit was merged to the GitOps repo, but the cluster still serves the bad version. How do you triage reconciliation lag vs sync failure?"

### Tradeoff

15. "Jenkins vs GitHub Actions vs GitLab CI vs Tekton — you chose Jenkins. What are the tradeoffs? When would you switch?"
16. "Multi-stage Docker builds add complexity. When is a single-stage build acceptable?"
17. "Your CI pipeline pushes images to ECR but does NOT deploy to the cluster. ArgoCD handles deployment. Why this separation? What would change if you used Jenkins to deploy directly?"
18. "Push-based deploy from CI vs pull-based deploy from ArgoCD: compare auditability, speed, and blast radius."
19. "Manual approval in production vs fully automated promotion: where do you place controls?"
20. "Declarative Pipeline vs scripted Groovy in this repo: why this choice, and what governance/risk tradeoff comes with it?"
21. "Static agents vs ephemeral agents: what operational tradeoffs matter most for this project?"

### Technology Comparison

22. "[DEFINE] What is Jenkins? Why Jenkins instead of GitHub Actions, GitLab CI, or CircleCI? What specific advantage does Jenkins give you for this project?"
23. "[DEFINE] What is a Docker multi-stage build? Why multi-stage instead of a single Dockerfile with all dependencies? What problem does it solve for image size and security?"
24. "[DEFINE] What is ECR? Why ECR instead of Docker Hub, GitHub Container Registry, or Harbor? What AWS-native integrations does ECR provide?"
25. "[DEFINE] What is continuous reconciliation in the context of CI/CD? Why is Git-based promotion safer than direct runtime patching?"
26. "[DEFINE] Git SHA tags vs semantic tags vs digest pinning: which is strongest for reproducibility and why?"
27. "[DEFINE] What is a Multibranch Pipeline? Why use it over single-pipeline jobs with manual branch handling?"
28. "[DEFINE] Compare SAST, SCA, and container scanning in CI. Why are all three not interchangeable?"

### Best Practice Pillar

29. **(Security)** "Map the credential flow in your CI pipeline: Jenkins → ECR (push), Jenkins → GitLab (clone), Jenkins → S3 (artifacts). For each, what authentication mechanism is used and what is the blast radius if Jenkins is compromised?"
30. **(Cost)** "Your Jenkins agent runs 24/7 on a t3a.medium EC2 instance. Builds happen ~10 times per day. What is the cost? What alternatives exist (spot, ephemeral agents, managed CI)?"
31. **(Availability)** "Jenkins is a single point of failure in your pipeline. If Jenkins goes down, can you still deploy? What is your Jenkins disaster recovery plan?"
32. **(Security)** "Which credentials in this delivery chain have the highest blast radius and how would you reduce them?"
33. **(Scalability)** "As teams and environments grow, what process bottleneck appears first in the current CI/CD workflow?"
34. **(Security)** "Short-lived OIDC role assumption vs static access keys stored in Jenkins credentials: what are the operational tradeoffs?"

### Bonus Pool

35. "What is the difference between `docker build --cache-from` and BuildKit cache mounts? When would you use each in CI?"
36. "Your Jenkinsfile has 8 stages. Which stages can run in parallel? What is the syntax?"
37. "How would you implement a canary deployment purely from the CI/CD perspective, before ArgoCD detects the change?"
38. "What pre-merge checks are mandatory before infrastructure changes can reach production?"
39. "How do you enforce reproducible releases end-to-end in this delivery model?"
40. (generic) "What is an SBOM (Software Bill of Materials)? Why is it important for supply chain security? How would you generate one in your CI pipeline?"
41. "A suspected credential leak appears in old build logs. What do you do in the first 15 minutes?"
42. "Build duration regresses 40% after a dependency upgrade. How do you isolate cache misses vs compute saturation vs test expansion?"
43. "Bigger always-on agents vs smaller elastic agents: where is your break-even point?"
