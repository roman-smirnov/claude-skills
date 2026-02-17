# GitOps / ArgoCD — Interview Module

## Enhancement Instructions

- Validate **OpenGitOps fundamentals**: declarative, versioned, pulled automatically, continuously reconciled, auditable. Score lower if the candidate treats GitOps as "Git + CI deploy."
- Probe **reconciliation and drift** depth: `OutOfSync`, `selfHeal`, `prune`, sync windows, rollback via Git revert. Score higher when the candidate can explain what ArgoCD does vs what it should NOT do.
- Evaluate **architecture tradeoff** reasoning: app-of-apps vs ApplicationSet, monorepo vs polyrepo, manual vs automated sync, and promotion strategies.
- Probe **secrets strategy** within GitOps: SOPS + age vs Sealed Secrets vs External Secrets Operator, with clear justification for the project's choice.
- Score higher when the candidate demonstrates awareness of **ArgoCD governance**: Projects/RBAC boundaries, blast radius reasoning, audit trail, and anti-pattern avoidance (manual kubectl, mutable tags, broad permissions).

---

## Evaluation Checklist

### OpenGitOps Principles (Mandatory)

The candidate should be able to explain and apply all five:

- [ ] **Declarative**: Desired state is declared in manifests
- [ ] **Versioned and immutable**: Desired state is stored in Git
- [ ] **Pulled automatically**: Agents reconcile desired vs actual state
- [ ] **Continuously reconciled**: Drift is detected and corrected
- [ ] **Auditable**: Deployment history is traceable in Git and controller history

### ArgoCD Core Mechanics

- [ ] Understands `Application` CRD purpose and lifecycle
- [ ] Understands `ApplicationSet` purpose and generator patterns (cluster, list, matrix)
- [ ] Can explain reconciliation loop and sync intervals
- [ ] Can explain `syncPolicy.automated`
- [ ] Can explain `selfHeal` behavior
- [ ] Can explain `prune` behavior and associated risk
- [ ] Understands sync waves (`argocd.argoproj.io/sync-wave`) and dependency ordering
- [ ] Understands hooks vs normal sync ordering
- [ ] Understands app-of-apps pattern and when it fits

### Repo Structure and Promotion

- [ ] Can explain repo model choice (monorepo/polyrepo/hybrid)
- [ ] Can explain environment promotion model (dev → staging → prod)
- [ ] Understands PR-based promotion and approval gates
- [ ] Explains why direct production commits are risky
- [ ] Can describe rollback via Git revert/cherry-pick and reconciliation
- [ ] Can explain why push-based cluster mutation from CI is anti-GitOps

### Drift and OutOfSync Troubleshooting

Candidate should demonstrate a structured flow:

1. [ ] Confirm app health/sync status and scope of impact
2. [ ] Inspect diff and resource-level status
3. [ ] Identify cause category: Git source issue, render issue, dependency ordering, credentials/RBAC, cluster-side validation
4. [ ] Validate with controller/application logs and events
5. [ ] Propose remediation that preserves Git as source of truth
6. [ ] Verify resolution and watch for recurrence

#### Common Issue Patterns
- [ ] Application `OutOfSync` with meaningful diff
- [ ] `OutOfSync` with non-obvious or noisy diff (server-side defaulting, annotation noise)
- [ ] Sync failure due to invalid resource/CRD mismatch
- [ ] Tracking behavior issues (annotation/label expectations)
- [ ] App appears healthy but resources remain degraded
- [ ] Repo credential/auth failures
- [ ] Sync wave deadlock due to dependency order

### Multi-Cluster and Scale

- [ ] Understands when to use ApplicationSet over app-of-apps
- [ ] Can explain cluster generator and matrix generator tradeoffs
- [ ] Understands ArgoCD scaling concerns (controller throughput, cache/API pressure)
- [ ] Can propose scale strategy (app partitioning, project boundaries, controller tuning)
- [ ] Understands blast radius implications of centralized control plane

### Secrets and Identity in GitOps

- [ ] Never commits plaintext secrets to Git
- [ ] Can compare **SOPS + age** vs **Sealed Secrets** vs **External Secrets Operator**
- [ ] Understands workload identity patterns (IRSA/OIDC) over static credentials
- [ ] Can explain credential rotation implications for ArgoCD repo access
- [ ] Understands least-privilege for controller and repo access
- [ ] Can identify blast radius if ArgoCD/controller credentials are compromised

### Security and Governance

- [ ] Understands ArgoCD Projects/RBAC boundaries
- [ ] Can explain separation of duties (who can change Git vs who can run infra)
- [ ] Can explain auditability of GitOps (Git history + controller history)
- [ ] Can identify dangerous anti-patterns: manual kubectl hotfixes, mutable `latest` tags, broad `cluster-admin`

### Reliability and Disaster Recovery

- [ ] Can explain what happens when ArgoCD is down vs when cluster workloads are down
- [ ] Can describe ArgoCD recovery/bootstrap strategy
- [ ] Can explain safe fallback/degraded operation during control-plane outage
- [ ] Can discuss sync windows/freeze policies and incident handling
- [ ] Can define recovery objectives for deployment control plane

### Delivery Safety and Progressive Rollout

- [ ] Understands progressive delivery concepts (canary/blue-green)
- [ ] Can explain when Argo Rollouts adds value vs standard rolling update
- [ ] Can define rollback trigger criteria (error rate/latency/SLO burn)
- [ ] Understands promotion guardrails and policy checks

### Argo Rollouts (NEW)

- [ ] Understands Rollout CRD and how it replaces Deployment for progressive delivery
- [ ] Can explain canary strategy with traffic splitting and analysis
- [ ] Knows metric-based rollback (AnalysisTemplate, AnalysisRun)
- [ ] Can compare canary vs blue-green rollout strategies

### Argo Notifications (NEW)

- [ ] Understands notification triggers and templates
- [ ] Can explain integration with Slack/Teams/webhook for sync status
- [ ] Knows how to configure notifications for sync failures and health degradation

### ArgoCD Image Updater (NEW)

- [ ] Understands how Image Updater watches registries for new tags
- [ ] Can explain write-back methods (Git commit vs annotation)
- [ ] Knows limitations and when manual image tag updates are preferred

### ArgoCD Projects Depth (NEW)

- [ ] Understands Project-level source/destination restrictions
- [ ] Can explain RBAC scoping with Projects (who can sync what)
- [ ] Knows how Projects limit blast radius in multi-team environments
- [ ] Can explain sync windows at the Project level

### ArgoCD vs Flux Comparison

| Criteria | ArgoCD | Flux |
|----------|--------|------|
| Architecture | Monolithic (UI + API + controller) | Modular microservices |
| UI | Built-in web UI | No built-in UI |
| Multi-tenancy | Built-in Projects with RBAC | Manual via K8s RBAC |
| Multi-cluster | ApplicationSets | Manual or hub-spoke |
| OCI Artifacts | Limited | GA since v2.6 |
| Secrets | ESO endorsed | Native SOPS+age |
| Image Automation | Via Image Updater (separate) | GA since v2.7 |
| Progressive Delivery | Argo Rollouts | Flagger |
| Resource Footprint | Higher (~500MB-1GB) | Lower (~200-400MB) |

### Secrets Management Comparison

| Solution | Type | Complexity | Best For |
|----------|------|------------|----------|
| **Sealed Secrets** | Encrypted in Git, controller decrypts | Low | Simple GitOps, static secrets, no external deps |
| **ESO** | Pulls from external store (AWS SM, Vault) | Medium | Cloud-native, dynamic/rotating, compliance |
| **SOPS + age** | Encrypted files in Git | Medium | Git-centric, multi-cloud, flexible, Flux-native |

### Diagnostic Commands

```bash
# ArgoCD Application Status
argocd app list
argocd app get <app>
argocd app diff <app>
argocd app get <app> --refresh

# Controller and Events
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
kubectl get applications,applicationsets -A
kubectl get events -A --sort-by='.lastTimestamp'

# Manifest Validation
helm template <chart> -f values.yaml
kustomize build <path>
```

### Scoring Application

| Coverage | Score |
|----------|-------|
| 0-20% of relevant items | 1–25 (Awareness) |
| 20-50% | 26–50 (Competent) |
| 50-80% with correct reconcile/drift reasoning | 51–75 (Proficient) |
| 80%+ with proactive scale/safety/governance depth | 76–100 (Expert) |

---

## Question Templates

### What Breaks If...

1. "Your root Application at `{path}` has `selfHeal: true` and `prune: true`. A manual `kubectl apply` hotfix is made in production. What happens and why?"
2. "The repo credential secret referenced by `{path}` is rotated, but ArgoCD is still using the old key. What fails first and how does this appear in app status?"
3. "A parent app-of-apps Application in `{path}` points to a directory where one child app manifest was deleted accidentally. What does prune do on next reconciliation?"
4. (generic) "The Git repo backing ArgoCD becomes unavailable for 2 hours. What is the impact on running workloads? What is the impact on new deployments?"
5. "[SKIP if no ApplicationSet] Your ApplicationSet at `{path}` uses a cluster generator. One cluster label changes unexpectedly. What deployment side effect should you expect?"

### How Would You Debug...

6. "An Application is `OutOfSync`, but the diff output looks trivial. Walk me through your debugging sequence and what each step validates."
7. "Sync wave ordering in `{path}` stalls at wave 2. How do you identify the blocking resource and confirm if it is a health, hook, or dependency problem?"
8. "A new app directory was merged to Git, but ArgoCD never creates the corresponding Application. Where do you investigate first, second, third?"
9. "ArgoCD UI shows healthy controller pods, but only one application repeatedly fails sync. How do you isolate app-level vs controller-level root cause?"

### Tradeoff

10. "You use app-of-apps in `{path}`. When would ApplicationSet be the better design, and what complexity does it introduce?"
11. "Monorepo vs polyrepo for GitOps configuration: which model fits this repo and why?"
12. "Auto-sync with prune/self-heal vs manual sync in production: what risk/speed tradeoff are you making?"
13. "PR-based promotion vs fully automated promotion to production: where should human approval live and why?"

### Technology Comparison

14. "[DEFINE] What is GitOps? What is ArgoCD? Why ArgoCD instead of push-based CD from Jenkins or GitHub Actions? What specific advantage does pull-based reconciliation give you?"
15. "[DEFINE] What is an ArgoCD Application vs ApplicationSet? Why use one over the other in this project?"
16. "[DEFINE] What is reconciliation? How does it differ from one-time deployment commands?"
17. "[DEFINE] What are ArgoCD sync waves and resource hooks? How do they compare to Helm hooks? Why use sync waves to order cert-manager before ingress-nginx?"
18. "[DEFINE] Why app-of-apps instead of Flux Kustomization/HelmRelease grouping for this setup?"

### Best Practice Pillar

19. **(Security)** "ArgoCD has cluster credentials and Git repo access. What is the blast radius if ArgoCD is compromised, and how do you reduce it?"
20. **(Availability)** "If ArgoCD control plane is down for an hour, what keeps running, what stops, and what is your recovery plan?"
21. **(Scalability)** "As app count grows 10x, what ArgoCD bottlenecks appear first (controller throughput, cache, API load), and how would you scale?"
22. **(Cost)** "What are the operational cost drivers of your GitOps model (controller resources, observability, promotion overhead), and where would you optimize first?"
23. **(Security)** "How do ArgoCD Projects limit blast radius? How would you configure RBAC so that team A can only sync apps in namespace A?"

### Argo Rollouts Questions (NEW)

24. "You want to add canary deployments for `{service}`. How does Argo Rollouts differ from a standard Kubernetes Deployment rolling update? What does the Rollout CRD add?"
25. "Your canary analysis shows a 3% error rate increase after shifting 10% of traffic. The AnalysisRun fails. What happens next automatically, and what would you investigate manually?"

### Bonus Pool

26. "What is annotation-based tracking in ArgoCD 3.x, and why does it matter during upgrades?"
27. "How do sync windows reduce deployment risk during incidents or freeze periods?"
28. "What metrics would you monitor to detect GitOps delivery health degradation early?"
29. "Explain a safe emergency fix process that stays GitOps-compliant."
30. "How would you set up ArgoCD Notifications to alert on sync failures in Slack?"
31. "A promotion PR merged but production behavior is unchanged. Which evidence do you check in Git, ArgoCD, and cluster state?"
32. "A rollback commit was merged to the GitOps repo, but the cluster still serves the bad version. How do you triage reconciliation lag vs sync failure?"
