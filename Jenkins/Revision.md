# DevOps / SRE / Cloud Interview Questions — Reordered Learning Sequence

---

## PHASE 1: CI/CD Fundamentals — Core Concepts

---

**Q1 — CI/CD Fundamentals**

You're onboarding a new engineer who has only worked with manual deployments. They ask:
"What specific problem does CI/CD solve, and why was it so painful before it existed?"
Give me a concrete, production-aware answer.

> 📄 **Document:** `01_CI_CD_Fundamentals.md`
> **Topic:** 1.1 — What is CI/CD
> **Section:** Why It Exists — The Problem It Solves

---

**Q2 — CI vs Continuous Delivery vs Continuous Deployment**

A senior engineer tells you: "We do CI/CD."
What follow-up questions would you ask to understand what they actually mean — and why does that matter?

> 📄 **Document:** `01_CI_CD_Fundamentals.md`
> **Topic:** 1.2 — CI vs Continuous Delivery vs Continuous Deployment ⚠️
> **Section:** All subsections

---

**Q9 — Software Delivery Pipeline**

Walk me through the stages of a production-grade software delivery pipeline. Why does stage ordering matter, and what is artifact immutability?

> 📄 **Document:** `01_CI_CD_Fundamentals.md`
> **Topic:** 1.3 — The Software Delivery Pipeline
> **Section:** All subsections

---

**Q6 — Branching Strategy**

Your team currently uses GitFlow with 2-week feature branches. Your Deployment Frequency DORA metric is 'Low' — once per month. A colleague says switching to Trunk-Based Development will fix this. Are they right? What's the full picture?

> 📄 **Document:** `01_CI_CD_Fundamentals.md`
> **Topic:** 1.4 — Trunk-Based Development vs Feature Branching vs GitFlow
> **Section:** All subsections

---

**Q8 — Shift-Left Testing**

As an SRE, why do you care about the testing strategy developers use? What's the direct connection between shift-left testing and production reliability?

> 📄 **Document:** `01_CI_CD_Fundamentals.md`
> **Topic:** 1.5 — Shift-Left Testing
> **Section:** All subsections

---

**Q7 — Pipeline as Code**

A Jenkins admin configures all pipelines through the Jenkins GUI. What risks are they taking that they may not be aware of?

> 📄 **Document:** `01_CI_CD_Fundamentals.md`
> **Topic:** 1.6 — Pipeline as Code
> **Section:** All subsections

---

**Q4 — Deployment Strategies**

Your team is debating Blue/Green vs Canary for your high-traffic e-commerce checkout service. Walk me through the tradeoffs and tell me which you'd recommend and why.

> 📄 **Document:** `01_CI_CD_Fundamentals.md`
> **Topic:** 1.7 — Deployment Strategies ⚠️
> **Section:** All subsections

---

**Q3 — Safe Continuous Deployment (Scenario)**

Your team practices Continuous Deployment to production. A new developer joins and on their second day does a git push at 4pm Friday. The build passes all tests. The code goes straight to production. 30 minutes later, error rates spike.
Walk me through: what architectural decisions should have been in place to make this safe, and what's your recovery path?

> 📄 **Document:** `01_CI_CD_Fundamentals.md`
> **Topic:** 1.2 — CI vs Continuous Delivery vs Continuous Deployment
> **Topic:** 1.7 — Deployment Strategies ⚠️
> **Topic:** 1.8 — Rollback Strategies ⚠️
> **Section:** All subsections

---

**Q5 — DORA Metrics**

Your engineering VP asks you: our deployments are failing 35% of the time and MTTR is 4 hours. Which do you fix first and how does CI/CD directly help?

> 📄 **Document:** `01_CI_CD_Fundamentals.md`
> **Topic:** 1.9 — DORA Metrics
> **Section:** All subsections

---

## PHASE 2: Jenkins Architecture & Internals

---

**Q10 — Jenkins Architecture**

Why should the Jenkins Controller always have zero executors in production? What are the security AND stability reasons?

> 📄 **Document:** `02_Jenkins_Architecture_Internals.md`
> **Topic:** 2.1 — Jenkins Architecture: Master/Controller, Agents, Executors
> **Section:** Why This Architecture Exists; How It Works Internally

---

**Q62 — Jenkins Installation and JVM Tuning**

You're deploying Jenkins on Kubernetes for the first time. A colleague copies a basic deployment YAML and sets -Xmx512m. Three days later Jenkins is crashing under load. What are the critical JVM settings they missed, and what's the Kubernetes-native way to size the heap correctly?

> 📄 **Document:** `02_Jenkins_Architecture_Internals.md`
> **Topic:** 2.2 — Jenkins Installation, System Configuration and Global Tools
> **Section:** JVM Tuning — Critical for Production; Kubernetes Deployment

---

**Q27 — Jenkins Controller Internals**

A developer says 'my build has been queued for 45 minutes but I can see agents are available.' Walk me through exactly how you diagnose this — what are the three queue states, and what does each one tell you?

> 📄 **Document:** `02_Jenkins_Architecture_Internals.md`
> **Topic:** 2.3 — Jenkins Controller Internals: Job Queue, Build Queue, Thread Model
> **Section:** The Build Queue States in Detail; Build Queue Stuck Analysis

---

**Q12 — Jenkins Agent Types**

Your team is moving to Kubernetes. Should you use DinD, DooD, or Kaniko for building Docker images inside Jenkins pod agents? Walk me through the tradeoffs.

> 📄 **Document:** `02_Jenkins_Architecture_Internals.md`
> **Topic:** 2.4 — Agent Types: Static, Dynamic, Docker, Kubernetes, SSH, JNLP
> **Section:** Type 5 — Kubernetes Agents; Docker-in-Docker vs Docker-out-of-Docker vs Kaniko

---

**Q52 — Jenkins Workspace Management**

A developer reports that their build is failing because it's picking up compiled artifacts from a previous build that no longer exist in the codebase. The build passes on a fresh agent but fails on the regular one. What's happening and what are the three ways to fix it?

> 📄 **Document:** `02_Jenkins_Architecture_Internals.md`
> **Topic:** 2.5 — Jenkins Workspace: How It Works, Cleanup, Shared Workspaces
> **Section:** The Stale Files Problem; Workspace Cleanup Strategies

---

**Q57 — Jenkins Home Directory**

Your Jenkins Controller disk is at 95% capacity and builds are starting to fail. Walk me through exactly what's consuming the space, what's safe to delete, and what you must never delete — and why one specific directory is more critical than all others combined.

> 📄 **Document:** `02_Jenkins_Architecture_Internals.md`
> **Topic:** 2.6 — Jenkins Home Directory Structure
> **Section:** Critical Files for Operations; Backup Priority; Build Log Storage

---

**Q11 — Jenkins HA & DR**

Jenkins has no native HA. Your CTO asks: what's your plan if the Jenkins Controller goes down at 2pm on a Tuesday? Walk me through your architecture.

> 📄 **Document:** `02_Jenkins_Architecture_Internals.md`
> **Topic:** 2.7 — Jenkins HA and Disaster Recovery ⚠️
> **Section:** HA and DR Strategies; Recovery Procedures; Recovery Time Objectives

---

**Q28 — Jenkins at Scale**

Your Jenkins Controller starts becoming unresponsive during peak hours — 9am Monday mornings when everyone pushes after the weekend. You have 200 concurrent builds. Walk me through your diagnosis and resolution.

> 📄 **Document:** `02_Jenkins_Architecture_Internals.md`
> **Topic:** 2.8 — Jenkins at Scale: Controller Bottlenecks, Agent Pools, Load Distribution
> **Section:** Controller Bottlenecks — Identified and Solved; Agent Pool Architecture Patterns

---

## PHASE 3: Jenkins Pipelines — Core

---

**Q92 — Freestyle vs Pipeline Jobs**

A legacy Jenkins setup has 400 Freestyle jobs. A new engineer asks: why should we migrate to Pipeline jobs? What are you actually gaining, and is there any legitimate reason to keep a Freestyle job in 2024?

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.1 — Freestyle Jobs vs Pipeline Jobs
> **Section:** Why Pipeline Jobs Exist — The Problems Freestyle Couldn't Solve

---

**Q87 — Jenkins Declarative Pipeline Syntax**

Write me the skeleton of a production-grade Declarative Pipeline for a Java service that: runs tests, builds a Docker image, deploys to staging automatically, requires manual approval for production, sends Slack notifications on failure, and cleans up the workspace. What are the critical options you'd always include and why?

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.2 — Declarative Pipeline — Full Syntax and Directives
> **Section:** The Complete Declarative Pipeline Anatomy; Directive Reference

---

**Q13 — Declarative vs Scripted**

A developer asks: should I write my Jenkins pipeline in Declarative or Scripted syntax? What's your answer and what are the edge cases where you'd switch?

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.4 — Declarative vs Scripted ⚠️
> **Section:** Side-By-Side Comparison; When to Choose Each

---

**Q88 — Jenkins Environment Variables**

A developer writes this in their Jenkinsfile: `sh "docker build -t myapp:${env.BUILD_NUMBER} ."` and separately `sh 'docker push myapp:$BUILD_NUMBER'`. One of these is subtly wrong in a way that causes hard-to-debug issues. Which one, why, and what's the rule you always follow?

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.11 — Environment Variables
> **Section:** GString Interpolation vs Shell Dollar Signs

---

**Q64 — Jenkins Credentials Management**

A developer needs to call an API using a Bearer token in their pipeline. They write: `sh "curl -H 'Authorization: Bearer ${TOKEN}' https://api.example.com"`. Walk me through exactly why this is dangerous, how Jenkins masking fails here, and write the correct implementation.

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.12 — Credentials Management in Jenkins
> **Section:** Injecting Credentials — All Patterns; Log Masking — How Jenkins Masks Credentials

---

**Q63 — Jenkins Pipeline Parameters**

A team adds a `parameters {}` block to their Jenkinsfile for the first time, pushes the commit, and the webhook fires a build. They expect to see parameter prompts but the build runs without them. Why, and what's the gotcha with choice parameter defaults that causes silent production incidents?

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.8 — Pipeline Parameters
> **Section:** All Parameter Types; Tricky Edge Cases and Gotchas

---

**Q65 — Jenkins Pipeline Triggers**

A developer adds `triggers { githubPush() }` to their Jenkinsfile in a Multibranch Pipeline. They push a commit but no build triggers. Meanwhile their colleague's single Pipeline job with the same trigger works fine. Why?

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.9 — Pipeline Triggers
> **Section:** Webhook Configuration — The Right Way; Tricky Edge Cases and Gotchas

---

**Q14 — Parallel Stages**

You have a pipeline running lint, unit tests, SAST scan, and dependency check sequentially — total 14 minutes. How do you restructure it, and what's the one Groovy bug that kills most parallel stage implementations?

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.7 — Parallel Stages ⚠️
> **Section:** Dynamic Parallel; Tricky Edge Cases and Gotchas

---

**Q90 — Jenkins Matrix Builds**

You need to test a Java library against JDK 11, 17, and 21 on both Ubuntu and Alpine. A junior engineer writes 6 separate parallel stages. What's the better approach, how does it work, and what happens to your agent pool when the matrix runs?

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.14 — Matrix Builds
> **Section:** Complete Matrix Build Syntax; Matrix Resource Planning

---

**Q15 — Jenkins Shared Libraries**

You manage 50 microservice pipelines. Every team copy-pastes the same 60-line Docker build block. What's the correct solution, how does it work internally, and what's the version pinning rule?

> 📄 **Document:** `03_Jenkins_Pipelines_Core.md`
> **Topic:** 3.13 — Pipeline Shared Libraries ⚠️
> **Section:** Shared Library Structure; vars/ — Global Variables; @Library Annotation

---

## PHASE 4: Jenkins Pipelines — Advanced Patterns

---

**Q29 — Multi-Branch Pipelines**

A developer creates a new feature branch and pushes a commit. Nothing happens in Jenkins — no pipeline created, no build triggered. Walk me through every possible reason and how you diagnose each one.

> 📄 **Document:** `04_Jenkins_Pipelines_Advanced_Patterns.md`
> **Topic:** 4.1 — Multi-Branch Pipelines
> **Section:** How Multi-Branch Pipelines Work Internally; Tricky Edge Cases and Gotchas

---

**Q91 — Jenkins Organisation Folders**

Your company has 200 GitHub repositories. Every time a new repo is created, a Jenkins admin has to manually set up a pipeline. This is taking hours per week. What Jenkins feature eliminates this entirely, how does it work, and what's the security risk you must configure correctly from day one?

> 📄 **Document:** `04_Jenkins_Pipelines_Advanced_Patterns.md`
> **Topic:** 4.2 — GitHub/GitLab Organization Folders
> **Section:** Organization Folder Setup; GitHub App Authentication; Tricky Edge Cases and Gotchas

---

**Q66 — Jenkins Restart from Stage**

A 45-minute pipeline fails at the very last stage — production deploy. The developer wants to restart from that stage to avoid waiting 44 minutes again. They do it, but the deployment fails because env.IMAGE_TAG is empty. Explain exactly why this happens and how you redesign the pipeline to make restart-from-stage reliable.

> 📄 **Document:** `04_Jenkins_Pipelines_Advanced_Patterns.md`
> **Topic:** 4.3 — Pipeline Restart from Stage ⚠️
> **Section:** What Restart from Stage PRESERVES and What It LOSES; Practical Patterns for Safe Stage Restart

---

**Q30 — Input Steps and Approval Gates**

Your pipeline has a production approval gate using the input step. A colleague notices that during the 8-hour approval window, one Jenkins agent executor is sitting completely idle. Why is this happening and how do you fix it?

> 📄 **Document:** `04_Jenkins_Pipelines_Advanced_Patterns.md`
> **Topic:** 4.4 — Input Steps and Manual Approval Gates
> **Section:** Agent Efficiency — The Critical Input Step Trick

---

**Q93 — Timeout and Retry Patterns**

A developer writes `timeout(time: 10, unit: 'MINUTES') { retry(3) { sh './deploy.sh' } }`. A senior engineer says the timeout and retry are in the wrong order for their use case — each attempt should get its own 10-minute budget. Who is right and why does the order matter?

> 📄 **Document:** `04_Jenkins_Pipelines_Advanced_Patterns.md`
> **Topic:** 4.5 — Timeout, Retry, and Resilience Patterns
> **Section:** retry() — Complete Reference; Tricky Edge Cases and Gotchas

---

**Q67 — Jenkins When Conditions**

You have a pipeline stage that deploys to a GPU node pool — these are expensive and scarce. The stage has a `when { branch 'main' }` condition. A colleague notices that even on feature branch builds, the GPU agent gets provisioned and then immediately released. What's happening and what's the one-word fix?

> 📄 **Document:** `04_Jenkins_Pipelines_Advanced_Patterns.md`
> **Topic:** 4.6 — When Conditions and Conditional Execution
> **Section:** beforeAgent and beforeInput — Critical Performance Optimization

---

**Q16 — Pipeline Durability**

A developer complains their pipeline loses state after a Jenkins Controller restart. You check and see PERFORMANCE_OPTIMIZED durability is set globally. Explain exactly what's happening and how you fix it without sacrificing build speed.

> 📄 **Document:** `04_Jenkins_Pipelines_Advanced_Patterns.md`
> **Topic:** 4.7 — Pipeline Durability Settings ⚠️
> **Section:** The Three Durability Levels; Durability and Pipeline Restart from Stage Interaction

---

**Q68 — Jenkins Build Promotion**

Your team rebuilds the Docker image at each environment stage — once for staging, once for production. A senior engineer says this violates a fundamental CI/CD principle. What principle, why does it matter for compliance, and what's the correct pattern?

> 📄 **Document:** `04_Jenkins_Pipelines_Advanced_Patterns.md`
> **Topic:** 4.8 — Build Promotion Patterns
> **Section:** Pattern 1 — Linear Promotion Pipeline; The Immutable Artifact Lifecycle

---

## PHASE 5: Jenkins Plugins Ecosystem

---

**Q94 — Essential Plugins Stack**

You're setting up a brand new Jenkins instance from scratch. What is the minimum viable plugin stack you install, and what's the single most dangerous operational mistake teams make with plugin management that causes silent production breakage?

> 📄 **Document:** `05_Jenkins_Plugins_Ecosystem.md`
> **Topic:** 5.1 — Essential Plugins
> **Section:** The Essential Plugin Stack — Detailed Reference; Plugin Management — Operations

---

**Q58 — Jenkins Trigger Plugins**

Your team has 500 Jenkins jobs all using `pollSCM('H/5 * * * *')`. Your platform team says this is causing problems at scale. Explain exactly what's wrong, what the correct alternative is, and what the 'H' syntax actually means.

> 📄 **Document:** `05_Jenkins_Plugins_Ecosystem.md`
> **Topic:** 5.2 — Build Trigger Plugins
> **Section:** Generic Webhook Trigger Plugin; Comparison — Which Trigger Plugin to Use

---

**Q95 — Notification Plugins Strategy**

Your team configured Slack notifications on every build — success and failure. Three weeks later nobody reads the Slack channel anymore. What went wrong, what's the correct notification strategy, and what are the specific post conditions you'd use and why?

> 📄 **Document:** `05_Jenkins_Plugins_Ecosystem.md`
> **Topic:** 5.3 — Notification Plugins
> **Section:** Notification Strategy — Production Patterns

---

**Q53 — Jenkins RBAC and Security Plugins**

Your company has 5 teams. Each team should only see and build their own pipelines, and cannot access other teams' production credentials even if they know the credential ID. Walk me through the exact Jenkins configuration to achieve this.

> 📄 **Document:** `05_Jenkins_Plugins_Ecosystem.md`
> **Topic:** 5.4 — Security Plugins — RBAC and Matrix Auth
> **Section:** Role-Based Authorization Strategy (RBAC); Folder-Based Access Control; Credentials Plugin Scoping

---

**Q17 — Kubernetes Plugin**

You're setting up Jenkins on Kubernetes. A junior engineer asks why the pod template always needs a container named `jnlp` and why you can't override its command. What's your explanation?

> 📄 **Document:** `05_Jenkins_Plugins_Ecosystem.md`
> **Topic:** 5.5 — Kubernetes Plugin ⚠️
> **Section:** How the Kubernetes Plugin Works Internally; Pod Template Configuration; Tricky Edge Cases and Gotchas

---

**Q69 — Docker Plugin vs Docker Pipeline Plugin**

A developer asks: 'I need to build a Docker image in my pipeline — should I use the Docker Plugin or the Docker Pipeline Plugin?' Most candidates confuse these. What's your answer?

> 📄 **Document:** `05_Jenkins_Plugins_Ecosystem.md`
> **Topic:** 5.6 — Docker Plugin vs Docker Pipeline Plugin
> **Section:** The Two Docker Use Cases; Docker Plugin vs Docker Pipeline Plugin — Decision Matrix

---

**Q18 — JCasC Plugin**

Your Jenkins Controller gets corrupted on a Friday evening. Your manager asks: how long until CI is back? Walk me through your recovery — and why teams without JCasC are in serious trouble here.

> 📄 **Document:** `05_Jenkins_Plugins_Ecosystem.md`
> **Topic:** 5.7 — Configuration as Code (JCasC) Plugin ⚠️
> **Section:** JCasC File Structure; JCasC Secrets Handling; JCasC Reload Without Restart

---

**Q96 — Job DSL Plugin**

A colleague says 'we use Job DSL to manage all our Jenkins jobs as code.' A senior engineer says 'Job DSL is largely obsolete in modern Jenkins.' Who is right, when is Job DSL still valuable, and what has replaced it for most use cases?

> 📄 **Document:** `05_Jenkins_Plugins_Ecosystem.md`
> **Topic:** 5.8 — Job DSL Plugin
> **Section:** Job DSL in the Modern Context; Job DSL vs JCasC vs Org Folders

---

## PHASE 6: Jenkins Integrations

---

**Q70 — Jenkins Git Integration**

Your monorepo is 5GB with 5 years of history. Every Jenkins build clones the full repo — taking 90 seconds just for checkout. Walk me through two Git-level optimizations that bring this under 10 seconds.

> 📄 **Document:** `06_Jenkins_Integrations.md`
> **Topic:** 6.1 — Jenkins + Git
> **Section:** Shallow Clone and Sparse Checkout Optimization

---

**Q71 — Jenkins Docker Integration**

You're running Jenkins on Kubernetes and need to build Docker images. A junior engineer proposes mounting `/var/run/docker.sock` from the host into the build pod. What's the security risk they don't understand, and what's the production-correct alternative?

> 📄 **Document:** `06_Jenkins_Integrations.md`
> **Topic:** 6.2 — Jenkins + Docker ⚠️
> **Section:** Approach 3 — Kaniko; Docker Layer Cache Optimization

---

**Q48 — Jenkins + Kubernetes Deployment**

Your Jenkins pipeline needs to deploy to a production Kubernetes cluster. A junior engineer suggests storing a cluster-admin kubeconfig as a Jenkins credential. What's wrong with this approach and what's the production-correct alternative?

> 📄 **Document:** `06_Jenkins_Integrations.md`
> **Topic:** 6.3 — Jenkins + Kubernetes
> **Section:** Authentication — Jenkins to Kubernetes Cluster; Helm Deployment Pattern

---

**Q55 — Jenkins SonarQube Integration**

A pipeline runs SonarQube analysis successfully but the quality gate check immediately passes even though there are critical issues. The developer says 'SonarQube is broken.' What's actually happening and what's the fix?

> 📄 **Document:** `06_Jenkins_Integrations.md`
> **Topic:** 6.4 — Jenkins + SonarQube
> **Section:** Complete SonarQube Pipeline Integration; Tricky Edge Cases and Gotchas

---

**Q85 — Jenkins Nexus Artifact Upload Fails**

Your pipeline fails with '401 Unauthorized' when uploading artifacts to Nexus. The credentials haven't changed. A colleague says 'just hardcode the password in the pom.xml.' Why is this wrong, and what's the correct fix?

> 📄 **Document:** `06_Jenkins_Integrations.md`
> **Topic:** 6.5 — Jenkins + Nexus/Artifactory
> **Section:** Publishing Artifacts to Nexus — Maven; Tricky Edge Cases and Gotchas

---

**Q19 — Jenkins + Vault Integration**

Your security team says: no more static credentials in Jenkins. All secrets must come from Vault. Walk me through two different integration patterns and which you'd recommend for a Kubernetes-based Jenkins setup.

> 📄 **Document:** `06_Jenkins_Integrations.md`
> **Topic:** 6.6 — Jenkins + Vault/Secrets Managers
> **Section:** Integration Method 1 — HashiCorp Vault Plugin; Integration Method 2 — Vault Kubernetes Auth

---

**Q97 — AWS/GCP/Azure Cloud Authentication**

Your Jenkins pipeline needs to push images to ECR and deploy to EKS. A junior engineer creates an IAM user, generates an access key, and stores it as a Jenkins credential. Your security team flags this immediately. What's wrong, what's the correct AWS-native alternative, and what's the equivalent pattern on GCP and Azure?

> 📄 **Document:** `06_Jenkins_Integrations.md`
> **Topic:** 6.7 — Jenkins + AWS/GCP/Azure
> **Section:** AWS Integration; Key Concepts — Cloud Auth Patterns

---

**Q56 — Terraform in CI/CD**

Your team runs `terraform plan` in Jenkins, a senior engineer reviews the output and approves, then Jenkins runs `terraform apply`. A colleague says this is safe. What's the subtle but critical flaw in this workflow that could cause the apply to execute a completely different plan than what was reviewed?

> 📄 **Document:** `06_Jenkins_Integrations.md`
> **Topic:** 6.8 — Jenkins + Terraform
> **Section:** Complete Terraform Pipeline; Preventing Destructive Operations; Tricky Edge Cases and Gotchas

---

## PHASE 7: Security in CI/CD

---

**Q20 — Secrets Management Security**

A developer writes this in their Jenkinsfile: `sh "curl -H 'Authorization: Bearer ${TOKEN}' https://api.example.com"`. What's wrong with it, why does Jenkins masking NOT protect them, and what's the fix?

> 📄 **Document:** `07_Security_CICD.md`
> **Topic:** 7.1 — Secrets Management ⚠️
> **Section:** Log Masking — How Jenkins Masks Credentials; What NOT to Do — Anti-Patterns; What TO Do — Best Practices

---

**Q41 — Pipeline Security: Script Approval and Sandbox**

A developer comes to you and says 'my pipeline keeps failing with Scripts not permitted to use new java.io.File — can you just approve it in Script Approval?' What do you say, what's the security risk they don't understand, and what's the correct fix?

> 📄 **Document:** `07_Security_CICD.md`
> **Topic:** 7.2 — Pipeline Security — Script Approval, Sandbox, Groovy Restrictions
> **Section:** The Groovy Sandbox — How It Works; In-Process Script Approval — Risks; Practical Security Checklist

---

**Q25 — Supply Chain Security**

After the SolarWinds attack, your CISO asks: how do we prove that what's running in production is exactly what our CI pipeline built — and hasn't been tampered with? What's your answer?

> 📄 **Document:** `07_Security_CICD.md`
> **Topic:** 7.3 — Supply Chain Security — SBOM, Image Signing, Cosign, Sigstore
> **Section:** SBOM; Image Signing with Cosign; SLSA Framework; Image Verification at Deployment

---

**Q44 — Jenkins Security: Least Privilege**

Your security audit finds that your Jenkins build agents use a Kubernetes ServiceAccount with cluster-admin. The engineer who set it up says 'it works for everything.' What's your response, what's the actual blast radius risk, and how do you fix it?

> 📄 **Document:** `07_Security_CICD.md`
> **Topic:** 7.4 — Least Privilege in Pipelines — Service Accounts, RBAC
> **Section:** Jenkins Service Account Design; AWS IAM Least Privilege for CI/CD; Build Agent Network Isolation

---

## PHASE 8: Scaling, Performance & Reliability

---

**Q31 — Jenkins Performance Tuning**

Your Jenkins Controller is experiencing 30-60 second UI freezes during peak build hours. You suspect GC issues. Walk me through your diagnosis approach, what JVM settings you'd check, and what you'd change.

> 📄 **Document:** `08_Scaling_Performance_Reliability.md`
> **Topic:** 8.1 — Jenkins Performance Tuning
> **Section:** Jenkins Performance Failure Modes; JVM Heap Settings; Garbage Collector Selection and Tuning; GC Logging for Diagnosis

---

**Q32 — Build Time Optimization**

A Java microservice pipeline takes 45 minutes. Your VP says get it under 10 minutes. Walk me through your optimization strategy in priority order — what do you attack first and why?

> 📄 **Document:** `08_Scaling_Performance_Reliability.md`
> **Topic:** 8.2 — Build Time Optimization
> **Section:** Where Build Time Goes; Caching Strategies; Parallel Stage Execution; Docker Layer Cache Optimization

---

**Q33 — Ephemeral vs Persistent Agents**

Your team is debating whether to use ephemeral Kubernetes pod agents or persistent VM agents for Jenkins builds. A colleague says 'persistent agents are faster so we should use those.' Is he right? What's the complete picture?

> 📄 **Document:** `08_Scaling_Performance_Reliability.md`
> **Topic:** 8.3 — Ephemeral vs Persistent Agents ⚠️
> **Section:** Persistent Agents — Characteristics; Ephemeral Agents — Characteristics; Hybrid Strategy; Mitigating Ephemeral Agent Cold Start Latency

---

**Q72 — Jenkins Backup and Recovery**

Your Jenkins Controller's disk gets corrupted on a Sunday night. You have a backup from Saturday. Your manager asks: what's the minimum you need to restore, and what happens if you restore credentials.xml but the secrets/ directory is from a different Jenkins instance?

> 📄 **Document:** `08_Scaling_Performance_Reliability.md`
> **Topic:** 8.4 — Jenkins Backup and Recovery
> **Section:** What Needs to Be Backed Up; Recovery Procedures; Recovery Time Objectives

---

**Q34 — Monitoring Jenkins**

Your manager asks you to build a Jenkins health dashboard. What are the 8 most important metrics you'd put on it, what thresholds would you alert on, and which single metric is the most important leading indicator of capacity problems?

> 📄 **Document:** `08_Scaling_Performance_Reliability.md`
> **Topic:** 8.5 — Monitoring Jenkins
> **Section:** Key Jenkins Metrics — Complete Reference; Alerting Rules; Grafana Dashboard

---

## PHASE 9: CI/CD Design Patterns & SRE Principles

---

**Q45 — Immutable Artifacts Pattern**

Your team rebuilds the Docker image separately for dev, staging, and production using the same Dockerfile and git commit. A colleague says 'it's the same code so it's fine.' What's wrong with this, and what's the correct pattern?

> 📄 **Document:** `09_CI_CD_Design_Patterns_SRE_Principle.md`
> **Topic:** 9.1 — Immutable Artifacts Pattern
> **Section:** The Immutable Artifact Lifecycle; Versioning Strategy; Image Signing for Supply Chain Security

---

**Q68 — (see Phase 4 — Build Promotion)**

> *(Already covered under Jenkins Advanced Patterns — Topic 4.8)*

---

**Q21 — GitOps vs Push-Based CD**

Your team currently has Jenkins running `kubectl apply` directly against the production cluster. A colleague proposes switching to ArgoCD + GitOps. What are you actually changing architecturally, and what problems does it solve that your current setup can't?

> 📄 **Document:** `09_CI_CD_Design_Patterns_SRE_Principle.md`
> **Topic:** 9.3 — GitOps vs Push-Based CI/CD ⚠️
> **Section:** Push-Based vs Pull-Based Architecture; GitOps Security Model; When to Use GitOps vs Push-Based

---

**Q49 — Feature Flags and CI/CD**

Your team wants to practice Continuous Deployment but the product team is nervous about incomplete features reaching users. A developer says 'we just won't merge until the feature is complete.' Why is this the wrong answer and what's the correct solution?

> 📄 **Document:** `09_CI_CD_Design_Patterns_SRE_Principle.md`
> **Topic:** 9.6 — Feature Flags and CI/CD
> **Section:** Feature Flag Lifecycle; CI/CD Integration Pattern; Feature Flag Types

---

**Q24 — Database Migrations in CI/CD**

Your team does Blue/Green deployments. A developer wants to rename a database column in the same release as the code that uses the new name. Why is this a production outage waiting to happen, and what's the correct pattern?

> 📄 **Document:** `09_CI_CD_Design_Patterns_SRE_Principle.md`
> **Topic:** 9.7 — Database Migrations in CI/CD ⚠️
> **Section:** Expand/Contract Pattern; The Three-Deployment Process; Blue/Green + Database Problem in Detail

---

**Q59 — Pipeline Observability**

Your CTO asks: how do we know if our CI/CD pipelines are getting slower over time? Right now nobody has visibility. What do you build, what metrics do you track, and how do you connect pipeline events to production incidents?

> 📄 **Document:** `09_CI_CD_Design_Patterns_SRE_Principle.md`
> **Topic:** 9.4 — Pipeline Observability
> **Section:** Pillar 1 — Metrics; Pillar 2 — Logs; Pillar 3 — Tracing; DORA Metrics Dashboard

---

**Q73 — Testing Strategy in Pipelines**

A team has 500 E2E tests and 50 unit tests. Their pipeline takes 2 hours. A senior engineer says their testing pyramid is inverted and that's the root cause. Explain what this means, why it's a problem, and what the correct distribution looks like.

> 📄 **Document:** `09_CI_CD_Design_Patterns_SRE_Principle.md`
> **Topic:** 9.5 — Testing Strategy in Pipelines
> **Section:** The Four Test Types; The Complete Pipeline Testing Orchestration

---

**Q26 — CI/CD for Microservices**

You have 50 microservices in a monorepo. Every commit to any file triggers a full rebuild of all 50 services — total build time is 3 hours. This is clearly broken. How do you fix it architecturally?

> 📄 **Document:** `09_CI_CD_Design_Patterns_SRE_Principle.md`
> **Topic:** 9.8 — CI/CD for Microservices
> **Section:** Monorepo vs Polyrepo; Affected Build Detection; Fan-Out Pipeline Pattern

---

## PHASE 10: ArgoCD — Fundamentals to Advanced

---

**Q22 — ArgoCD Architecture**

An interviewer asks you to explain ArgoCD's internal components. They specifically ask: why does the Application Controller run as a StatefulSet and not a Deployment? Most candidates get this wrong.

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 3 — ArgoCD Architecture — Deep Dive
> **Subsection:** Component Deep Dive; Application Controller; How Components Interact — The Full Flow

---

**Q89 — ArgoCD Refresh vs Sync**

A developer asks: 'I clicked Refresh in ArgoCD and it shows OutOfSync — so I need to click Sync now right?' They treat these as two steps of the same action. Correct their mental model — what does each operation actually do, and can Refresh ever change the cluster?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 7 — Drift Detection and Reconciliation Loops
> **Subsection:** Refresh — Commonly Asked; Reconciliation Loop Detail

---

**Q42 — ArgoCD Drift Detection and Self-Heal**

An on-call engineer manually runs `kubectl scale deployment my-app --replicas=10` during an incident to handle a traffic spike. 3 minutes later the replicas drop back to 3. They're furious. What's happening, why is this actually correct behaviour, and what should they have done instead?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 7 — Drift Detection and Reconciliation Loops
> **Subsection:** Self-Heal Behavior; Changing Reconciliation Interval

---

**Q47 — ArgoCD Sync Policies and Prune**

A team deletes a Deployment from their Git repo expecting it to be removed from the cluster. Three days later it's still running. They're confused — ArgoCD shows the app as Synced. What's happening and what configuration is missing?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 5 — Sync Policies, Sync Waves, and Sync Hooks
> **Subsection:** Sync Policies; prune: true — Critical Production Decision

---

**Q23 — ArgoCD Sync Waves vs Hooks**

You're deploying a stack that has a database, a backend API, and a frontend — all in one ArgoCD Application. The backend keeps failing because the database isn't ready. How do you fix this using ArgoCD-native features, and what's the difference between the two mechanisms available to you?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 5 — Sync Policies, Sync Waves, and Sync Hooks
> **Subsection:** Sync Waves ⚠️; Sync Hooks; Hook vs Sync Wave — Common Confusion

---

**Q54 — ArgoCD Health Checks**

Your ArgoCD Application shows Healthy but the actual service is returning 500 errors to users. How is this possible, what's the root cause, and how do you fix ArgoCD's health assessment for custom resources?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 6 — Health Checks and Resource Tracking
> **Subsection:** Health Status Values; Custom Health Checks; ignoreDifferences

---

**Q76 — ArgoCD ignoreDifferences**

Your ArgoCD Application keeps showing OutOfSync every few minutes even though nobody is making changes. You sync it, it goes green, then 3 minutes later it's OutOfSync again. What are the two most likely causes and how do you fix each one permanently?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 6 — Health Checks and Resource Tracking
> **Subsection:** ignoreDifferences — Handling Noise; Tricky Edge Cases and Gotchas

---

**Q74 — ArgoCD Projects and Multi-Tenancy**

You have 5 teams sharing one ArgoCD instance. Team A accidentally deploys to Team B's production namespace. How does this happen by default in ArgoCD, and what's the exact configuration that prevents it?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 4 — Core Concepts: Applications, Projects, AppSets
> **Subsection:** AppProject (ArgoCD Projects); default Project Warning

---

**Q80 — ArgoCD App of Apps Bootstrap**

You're setting up a brand new Kubernetes cluster. You want everything — including ArgoCD application configuration — to be GitOps-managed. But you need ArgoCD running before it can manage itself. How do you solve this chicken-and-egg problem?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 10 — App of Apps Pattern
> **Subsection:** What It Is; Implementation; App of Apps vs ApplicationSet

---

**Q43 — App of Apps vs ApplicationSets**

You manage 50 microservices across 3 environments — dev, staging, production. That's 150 ArgoCD Applications to manage. Walk me through two different ArgoCD-native approaches to handle this at scale and when you'd choose each one.

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 10 — App of Apps Pattern; 11 — ApplicationSets — Fleet Management
> **Subsection:** App of Apps vs ApplicationSet; Generator Types

---

**Q60 — ArgoCD ApplicationSet Generators**

You need to automatically create one ArgoCD Application per directory in your Git repo, and also one per registered cluster — then deploy every app to every cluster. Walk me through which generators you'd use and how you'd combine them.

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 11 — ApplicationSets — Fleet Management
> **Subsection:** Git Generator; Cluster Generator; Matrix Generator

---

**Q36 — ArgoCD Multi-Cluster**

You need to deploy the same application to 20 Kubernetes clusters across 3 regions. How does ArgoCD handle this, how does it authenticate to external clusters, and what happens to build performance at scale?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 12 — Multi-Cluster Deployments
> **Subsection:** Registering External Clusters; How ArgoCD Authenticates to External Clusters; Application Controller Sharding

---

**Q37 — ArgoCD RBAC and SSO**

Your company has 200 engineers across 10 teams. Each team should only be able to deploy to their own namespaces and repos. How do you implement this in ArgoCD — walk me through the exact configuration including what happens if a team tries to deploy outside their boundaries.

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 13 — RBAC and SSO Integration
> **Subsection:** RBAC in ArgoCD; Custom RBAC Policy Syntax; SSO Integration

---

**Q35 — ArgoCD Secrets Management**

Your security team says no plaintext secrets in Git. You're using ArgoCD. Walk me through three different solutions, their tradeoffs, and which you'd recommend for an AWS-native environment.

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 14 — Secrets Management with ArgoCD
> **Subsection:** Sealed Secrets; External Secrets Operator; Vault + AVP; Comparison Table

---

**Q51 — ArgoCD CI+CD Integration**

Draw me the complete flow from a developer pushing code to it running in production — using Jenkins for CI and ArgoCD for CD. Where exactly does Jenkins stop and ArgoCD start, and why is that boundary placed there specifically?

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 15 — CI + CD Integration: Jenkins + ArgoCD
> **Subsection:** The Complete CI/CD Flow; Jenkins Pipeline for GitOps; ArgoCD Token for CI — Best Practice

---

**Q38 — Argo Rollouts Progressive Delivery**

What's the difference between ArgoCD and Argo Rollouts? A candidate says 'ArgoCD does canary deployments.' Are they correct? Walk me through exactly how a canary deployment works when both tools are used together.

> 📄 **Document:** `Complete.md` (ArgoCD Guide)
> **Section:** 16 — Argo Rollouts: Progressive Delivery
> **Subsection:** What is Argo Rollouts; Canary Deployment; ArgoCD + Argo Rollouts Integration

---

## PHASE 11: Troubleshooting Scenarios

---

**Q39 — Build Passes Locally, Fails in Jenkins**

A developer is frustrated — their tests pass locally every time but fail in Jenkins on every run. They've checked the code three times. Walk me through your systematic diagnosis — what are you checking and in what order?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.1 — Build Passes Locally but Fails in Jenkins
> **Section:** Root Causes and Diagnosis; Fix

---

**Q98 — Workspace Pollution**

A build fails with 'file already exists' error. The same build on a fresh agent passes perfectly. This has happened three times this week on the same persistent agent. What's the root cause, what are the three ways to fix it, and which is the most reliable for production?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.6 — Pipeline Fails Due to Workspace Pollution
> **Section:** Root Causes; Fix

---

**Q46 — Pipeline Hanging**

A pipeline has been showing 'Running' for 6 hours with no console output after the line 'Starting deployment.' No errors. No failures. Just silence. Walk me through your diagnosis and the three most likely root causes.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.2 — Jenkins Pipeline Stuck / Hanging Indefinitely
> **Section:** Root Causes; Diagnosis Steps; Fix

---

**Q50 — OOM Crash**

Your Jenkins Controller crashes every Monday morning around 9am with OutOfMemoryError. It's been happening for 3 weeks. Walk me through your root cause analysis and the permanent fix.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.3 — Out of Memory (OOM) / Jenkins Controller Crash
> **Section:** Root Causes; Diagnosis Steps; Fix

---

**Q77 — Flaky Tests**

Your pipeline has a 15% flake rate — roughly 1 in 7 builds fails randomly on the same test, then passes on retry. A developer says 'just add retry(3) and it's fixed.' Why is this the wrong answer, what's the correct treatment, and how do you measure flakiness systematically?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.4 — Flaky Tests — Tests That Randomly Pass/Fail
> **Section:** Root Causes; Fix; Gotcha

---

**Q61 — Credentials Visible in Logs**

During a security audit, the auditor finds API keys visible in Jenkins build logs from 6 months ago. The developer swears they used withCredentials. How did the secret leak despite using the correct mechanism, and what are the three bypass vectors that defeat Jenkins masking?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.5 — Credentials/Secrets Showing in Build Logs
> **Section:** What NOT to Do; Correct Approach; If a Secret Is Already Leaked

---

**Q75 — Agent Disconnects Mid-Build**

Mid-build, your Jenkins pipeline fails with 'Agent went offline.' This has happened 3 times this week on different builds. Walk me through your systematic diagnosis — what are you checking on the agent, on the network, and in Kubernetes?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.7 — Jenkins Agent Goes Offline / Disconnects During Build
> **Section:** Root Causes; Diagnosis Steps; Fix

---

**Q79 — Docker Build Fails**

A pipeline fails with 'permission denied while trying to connect to the Docker daemon at unix:///var/run/docker.sock'. The developer says Docker is installed on the agent. What are the three possible causes and fixes, and which approach should you NOT use in production Kubernetes?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.8 — Docker Build Fails in Jenkins Pipeline
> **Section:** Root Causes; Fix; Gotcha

---

**Q99 — Git Authentication Failures**

A pipeline suddenly fails with `fatal: could not read Username for 'https://github.com'` — it was working fine yesterday. Walk me through every possible cause in priority order and how you diagnose each one without breaking other pipelines.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.9 — Git Checkout Fails / Authentication Issues
> **Section:** Root Causes; Fix

---

**Q83 — Shared Library Not Loading**

A pipeline fails with `groovy.lang.MissingPropertyException: No such property: deployApp`. The developer insists the shared library exists and the @Library annotation is correct. Walk me through every possible cause — from directory structure to sandbox restrictions.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.10 — Pipeline Shared Library Not Loading / Import Errors
> **Section:** Root Causes; Correct Library Structure; Diagnosis

---

**Q100 — SonarQube Quality Gate Hangs**

A pipeline hangs indefinitely at the `waitForQualityGate` step — it never times out, never fails, just waits forever. The SonarQube analysis stage completed successfully. What's the single most likely cause, and what's the one-time setup step that fixes it permanently?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.11 — SonarQube Quality Gate Blocking Pipeline
> **Section:** Root Causes; Fix

---

**Q85 — (see Phase 6 — Nexus)**

> *(Already covered under Jenkins Integrations — Topic 6.5)*

---

**Q78 — Multi-Branch Not Detecting Branches**

A developer pushes a new feature branch but no pipeline appears in Jenkins after 30 minutes. They check — the Jenkinsfile exists on the branch. Walk me through every possible cause in priority order.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.13 — Multi-Branch Pipeline Not Detecting New Branches
> **Section:** Root Causes; Fix

---

**Q101 — Parallel Stage Race Condition**

Two parallel stages both write to a file called `output.txt` in the workspace. Intermittently one overwrites the other's results — builds pass but reports are wrong. What's happening, why is this a race condition, and what are the two correct fixes?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.14 — Parallel Stages Failing with Confusing Error Messages
> **Section:** Common Gotcha; Fix

---

**Q82 — Stash/Unstash Bottleneck**

A pipeline stashes a 200MB Docker context in the Build stage and unstashes it in the Deploy stage on a different agent. The pipeline is extremely slow and the Jenkins Controller becomes unresponsive during peak hours. What's happening architecturally, and what's the production-correct alternative?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.15 — Stash/Unstash Failing Across Agents
> **Section:** How Stash Works; Fix; Alternative for Large Artifacts

---

**Q84 — kubectl rollout undo Overridden by ArgoCD**

An on-call engineer runs `kubectl rollout undo deployment/my-app` to rollback a bad deployment. It works for exactly 3 minutes, then the bad version comes back. They're confused and panicking. What's happening and what should they have done instead?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.16 — Blue/Green Deployment Rollback Triggered
> **Section:** Rollback Strategy; Warning — kubectl rollout undo + ArgoCD selfHeal conflict

---

**Q102 — JCasC Not Applying**

A platform engineer updates the JCasC YAML file, commits it, and restarts Jenkins. The changes don't appear. They check the file — it looks correct. Walk me through every possible reason the configuration isn't applying and how you verify each one.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.17 — Jenkins Job DSL / JCasC Configuration Not Applying
> **Section:** Fix; Force JCasC Reload

---

**Q103 — Kubernetes Pod Agent Stuck in Pending**

A pipeline says 'waiting for agent to connect' for 10 minutes then times out. You run `kubectl get pods -n jenkins` and see the agent pod in Pending state. Walk me through your diagnosis — what does `kubectl describe pod` tell you, and what are the four most common reasons a Jenkins pod agent gets stuck in Pending?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.18 — Kubernetes Pod Agent Not Starting
> **Section:** Diagnosis; Fix

---

**Q104 — Deploy Without Approval**

A post-incident review reveals that build #47 deployed to production without triggering the manual approval gate — even though the gate exists in the Jenkinsfile. The developer says 'the approval was there last week.' Walk me through how this could happen and how you make approval gates truly bulletproof.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.20 — Deployment to Production Without Proper Approval
> **Section:** Root Cause Analysis; Correct Implementation

---

**Q86 — High Build Queue**

It's 9am Monday. 30 builds have been queued for over an hour. Agents appear to be online and available. Your Slack is exploding. Walk me through your incident response — what do you check first, second, and third, and what are the four possible root causes?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.21 — High Build Queue — Builds Waiting for Hours
> **Section:** Root Causes; Diagnosis; Fix; Scaling Fix

---

**Q81 — Canary Rollback Response**

You deployed a canary to 10% of traffic. Error rate jumped from 0.1% to 2%. Walk me through your response in exact phases with timings — what do you do in the first 2 minutes, 2-5 minutes, and 5-10 minutes?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.22 — Canary Deployment Showing Errors — How to Respond
> **Section:** Step-by-Step Response; Automated Canary Analysis in Pipeline

---

**Q105 — Database Migration Fails in Pipeline**

Your deployment pipeline runs a database migration, then deploys the new application version. The migration succeeds but the app crashes on startup with 'column not found.' Meanwhile rolling back the app deployment fails because the old version can't read the new schema either. You're stuck. How did you get here and how do you prevent this permanently?

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.23 — Pipeline Fails on Database Migration
> **Section:** Root Cause Patterns; Safe Migration Pattern; Interview Answer

---

**Q106 — Jenkins Upgrade Breaks Pipelines**

After a Jenkins LTS upgrade on a Friday afternoon, 30% of pipelines start failing with Groovy syntax errors that didn't exist before. Your team is panicking. Walk me through your immediate response, how you safely test Jenkins upgrades in future, and what the most common upgrade break patterns are.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.24 — Jenkins Upgrade Breaks Pipelines
> **Section:** Prevention and Fix; Common Upgrade Break Patterns

---

**Q40 — Dependency Failures**

Your pipeline was green yesterday. No code changes today. It's now failing with 'Could not resolve dependency.' Walk me through every possible root cause and your prevention strategy.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.26 — Build Dependency Failures
> **Section:** Case A — External Registry Down; Case B — SNAPSHOT Dependency; Case C — Transitive Conflict; Case D — Corrupted Cache; Prevention Checklist

---

**Q107 — Pipeline Failure Analysis Framework**

A senior engineer walks into a war room. A critical pipeline has been failing for 2 hours and nobody can find the root cause. They've been randomly trying fixes. The senior engineer says 'stop — you're doing this wrong.' Walk me through the structured 5-step framework for diagnosing any pipeline failure systematically.

> 📄 **Document:** `11_Troubleshooting.md`
> **Scenario:** 11.25 — Complete Pipeline Failure Analysis Framework
> **Section:** The Systematic Approach; Interview Cheat Sheet — What Interviewers Are Really Testing

---

*Total: 107 Questions | Sequence: Fundamentals → Architecture → Pipelines → Plugins → Integrations → Security → Scaling → Design Patterns → ArgoCD → Troubleshooting*