1. GitOps Fundamentals (New concept — must know)

What is GitOps and how it differs from traditional CI/CD
Push-based (Jenkins/GHA) vs Pull-based (ArgoCD) deployment model
Why GitOps over traditional pipelines
Git as single source of truth

2. ArgoCD Architecture (Frequently asked)

Core components: API Server, Repo Server, Application Controller, Dex, Redis
How ArgoCD watches and reconciles Git state
How it differs from Jenkins architecture

3. Core Concepts (Must know)

Application, AppProject, Source, Destination
Sync Status vs Health Status
Desired state vs Live state
Drift detection

4. Sync Policies & Strategies (Very commonly asked)

Manual vs Automatic sync
Self-heal and Prune — what they do and risks
Sync waves and hooks (PreSync, Sync, PostSync)
How this compares to a Jenkins pipeline stage

5. Config Management Tools (Practical questions)

Helm with ArgoCD
Kustomize with ArgoCD
Plain manifests
When to use which

6. App of Apps & ApplicationSets (Senior-level must)

App of Apps pattern — what and why
ApplicationSet generators (Git, Cluster, List, Matrix)
Managing multi-env and multi-cluster deployments

7. RBAC & SSO (Security round questions)

ArgoCD RBAC model vs Jenkins role strategy
AppProjects for team isolation
SSO with OIDC/Dex (GitHub, Google)

8. Multi-Cluster Management (Scenario-based)

Registering and deploying to multiple clusters
Hub-spoke pattern
How you'd manage dev/staging/prod clusters

9. Secrets Management (Always asked)

Why you can't put raw secrets in Git
Sealed Secrets, External Secrets Operator
Vault integration with ArgoCD

10. Notifications & Alerting (Practical)

Setting up Slack/email notifications
Triggers and templates
Comparison to Jenkins post-build notification

11. ArgoCD + CI Pipeline Integration (Your strong zone)

Where Jenkins/GHA ends and ArgoCD begins
Triggering ArgoCD sync from GitHub Actions
Image tag update strategies (manual commit vs Image Updater)
ArgoCD Image Updater


12. Progressive Delivery (Advanced interviews)

Argo Rollouts — Blue-Green, Canary
How it integrates with ArgoCD
Comparison to manual Jenkins deployment strategies

13. Monitoring & Observability (Ops-focused rounds)

Prometheus metrics from ArgoCD
Key metrics: sync status, health, reconciliation
Grafana dashboards

14. Troubleshooting (Scenario-based rounds)

App stuck in Progressing/Degraded state
Sync failed — how to debug
Out-of-sync but no changes in Git — why?
Comparing to Jenkins pipeline failure debugging

15. Best Practices & Patterns (Architectural rounds)

Monorepo vs Polyrepo for GitOps
Branch vs directory strategy for environments
Avoiding config drift
Separation of app code repo vs config repo
