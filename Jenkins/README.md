CATEGORY 1: CI/CD Fundamentals

The conceptual foundation everything else builds on

#	Topic	Difficulty
1.1	What is CI/CD — definitions, goals, and the full pipeline lifecycle	🟢 Beginner
1.2	CI vs CD vs CD (Continuous Delivery vs Deployment) — the distinction most candidates blur	🟢 Beginner
1.3	The Software Delivery Pipeline — Stages, Gates, and Artifacts	🟢 Beginner
1.4	Trunk-Based Development vs Feature Branching vs GitFlow	🟡 Intermediate
1.5	Shift-Left Testing — what it means and why SREs care	🟡 Intermediate
1.6	Pipeline as Code — philosophy and why it matters	🟡 Intermediate
1.7	Deployment Strategies — Blue/Green, Canary, Rolling, Recreate	🔴 Advanced
1.8	Rollback Strategies — automated vs manual, feature flags, database migrations	🔴 Advanced
1.9	DORA Metrics — Deployment Frequency, Lead Time, MTTR, Change Failure Rate	🟡 Intermediate
CATEGORY 2: Jenkins Architecture & Internals

The engine room — heavily tested in senior interviews

#	Topic	Difficulty
2.1	Jenkins Architecture — Master/Controller, Agents, Executors	🟢 Beginner
2.2	Jenkins Installation, System Configuration, and Global Tools	🟢 Beginner
2.3	Jenkins Controller Internals — Job Queue, Build Queue, Thread Model	🟡 Intermediate
2.4	Agent Types — Static, Dynamic, Docker, Kubernetes, SSH, JNLP	🟡 Intermediate
2.5	Jenkins Workspace — how it works, cleanup, shared workspaces	🟡 Intermediate
2.6	Jenkins Home Directory structure — jobs/, builds/, plugins/, nodes/	🟡 Intermediate
2.7	Jenkins HA and Disaster Recovery — no native HA, mitigation patterns	🔴 Advanced
2.8	Jenkins at Scale — controller bottlenecks, agent pools, load distribution	🔴 Advanced
CATEGORY 3: Jenkins Pipelines — Core

The most heavily tested area in interviews

#	Topic	Difficulty
3.1	Freestyle Jobs vs Pipeline Jobs — when to use what	🟢 Beginner
3.2	Declarative Pipeline — full syntax, structure, and directives	🟢 Beginner
3.3	Scripted Pipeline — Groovy-based, node/stage blocks, flexibility	🟡 Intermediate
3.4	Declarative vs Scripted — key differences, when to choose each ⚠️	🟡 Intermediate
3.5	Jenkinsfile — anatomy, best practices, version control	🟢 Beginner
3.6	Stages, Steps, and Post Actions	🟢 Beginner
3.7	Parallel Stages — syntax, use cases, fail-fast behavior ⚠️	🟡 Intermediate
3.8	Pipeline Parameters — input, choice, boolean, string	🟡 Intermediate
3.9	Pipeline Triggers — SCM polling, webhooks, cron, upstream	🟡 Intermediate
3.10	Stash and Unstash — sharing files between stages ⚠️	🟡 Intermediate
3.11	Environment Variables — built-in, custom, secrets injection	🟡 Intermediate
3.12	Credentials Management in Jenkins — types, binding, Vault integration	🔴 Advanced
3.13	Pipeline Shared Libraries — structure, @Library, vars/, src/ ⚠️	🔴 Advanced
3.14	Matrix Builds — multi-axis testing	🔴 Advanced
CATEGORY 4: Jenkins Pipelines — Advanced Patterns

#	Topic	Difficulty
4.1	Multi-Branch Pipelines — how they work, branch detection, PR builds	🟡 Intermediate
4.2	GitHub/GitLab Organization Folders — scanning, auto-discovery	🔴 Advanced
4.3	Pipeline Restart from Stage — checkpoint, limitations ⚠️	🔴 Advanced
4.4	Input Steps and Manual Approval Gates	🟡 Intermediate
4.5	Timeout, Retry, and Resilience Patterns	🟡 Intermediate
4.6	When Conditions and Conditional Execution	🟡 Intermediate
4.7	Pipeline Durability Settings ⚠️	🔴 Advanced
4.8	Build Promotion Patterns	🔴 Advanced
CATEGORY 5: Jenkins Plugins Ecosystem

#	Topic	Difficulty
5.1	Essential Plugins — Git, Pipeline, Credentials, Blue Ocean, Workspace Cleanup	🟢 Beginner
5.2	Build Trigger Plugins — Generic Webhook, GitHub, GitLab	🟡 Intermediate
5.3	Notification Plugins — Slack, Email-ext, PagerDuty	🟡 Intermediate
5.4	Security Plugins — Role-Based Access Control (RBAC), Matrix Auth	🟡 Intermediate
5.5	Kubernetes Plugin — dynamic pod agents, podTemplate ⚠️	🔴 Advanced
5.6	Docker Plugin vs Docker Pipeline Plugin	🟡 Intermediate
5.7	Configuration as Code (JCasC) Plugin ⚠️	🔴 Advanced
5.8	Job DSL Plugin — programmatic job creation	🔴 Advanced
CATEGORY 6: Jenkins Integrations

#	Topic	Difficulty
6.1	Jenkins + Git — webhooks, polling, branch strategies	🟢 Beginner
6.2	Jenkins + Docker — building images, Docker-in-Docker, best practices ⚠️	🟡 Intermediate
6.3	Jenkins + Kubernetes — dynamic agents, Helm deployments from Jenkins	🔴 Advanced
6.4	Jenkins + SonarQube — quality gates, scan integration	🟡 Intermediate
6.5	Jenkins + Nexus/Artifactory — artifact publishing, dependency proxying	🟡 Intermediate
6.6	Jenkins + Vault/Secrets Managers — dynamic secrets, token renewal	🔴 Advanced
6.7	Jenkins + AWS/GCP/Azure — cloud deployments, IAM roles, ECR/GCR	🔴 Advanced
6.8	Jenkins + Terraform — infrastructure pipelines, state management	🔴 Advanced
CATEGORY 7: Security in CI/CD

#	Topic	Difficulty
7.1	Secrets Management — what NOT to do, what to do ⚠️	🟡 Intermediate
7.2	Pipeline Security — script approval, sandbox, Groovy restrictions	🔴 Advanced
7.3	Supply Chain Security — SBOM, image signing, Cosign, Sigstore	🔴 Advanced
7.4	Least Privilege in Pipelines — service accounts, RBAC	🟡 Intermediate
7.5	Dependency Scanning and SAST in Pipelines	🟡 Intermediate
CATEGORY 8: Scaling, Performance & Reliability

#	Topic	Difficulty
8.1	Jenkins Performance Tuning — JVM settings, GC, thread pools	🔴 Advanced
8.2	Build Time Optimization — caching, parallelism, incremental builds	🔴 Advanced
8.3	Ephemeral Agents vs Persistent Agents — tradeoffs ⚠️	🟡 Intermediate
8.4	Jenkins Backup and Recovery strategies	🟡 Intermediate
8.5	Monitoring Jenkins — metrics, Prometheus, Grafana dashboards	🔴 Advanced
CATEGORY 9: CI/CD Design Patterns & SRE Principles

#	Topic	Difficulty
9.1	Immutable Artifacts Pattern	🟡 Intermediate
9.2	Environment Promotion Pattern	🟡 Intermediate
9.3	GitOps vs Push-based CI/CD ⚠️	🔴 Advanced
9.4	Pipeline Observability — tracing, logs, metrics from pipelines	🔴 Advanced
9.5	Testing Strategy in Pipelines — unit, integration, e2e, contract	🟡 Intermediate
9.6	Feature Flags and CI/CD	🟡 Intermediate
9.7	Database Migrations in CI/CD ⚠️	🔴 Advanced
9.8	CI/CD for Microservices — monorepos, polyrepos, fan-out pipelines	🔴 Advanced
CATEGORY 10: Modern CI/CD Ecosystem Awareness

Jenkins comparisons — asked at product companies

#	Topic	Difficulty
10.1	Jenkins vs GitHub Actions vs GitLab CI vs CircleCI vs Tekton	🟡 Intermediate
10.2	When to choose Jenkins — and when NOT to	🟡 Intermediate
10.3	Jenkins X and cloud-native CI/CD	🔴 Advanced
10.4	Argo CD and ArgoWorkflows in the CI/CD landscape	🔴 Advanced
