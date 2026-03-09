1. Jenkins Architecture (Always asked)

Master/Controller vs Agent/Node architecture
How builds are distributed across agents
Inbound vs Outbound agents (JNLP vs SSH)
Jenkins in Kubernetes (dynamic agents with pods)
How it compares to GitHub Actions runners

2. Jenkinsfile & Pipeline as Code (Core topic)

Declarative vs Scripted pipeline — differences & when to use
Pipeline syntax deep dive (agent, stages, steps, post)
Shared Libraries — structure, usage, why they matter
Reusable pipeline templates
Parallel stages and parallel execution
Pipeline best practices

3. Agents & Executors (Scenario-based)

Static vs dynamic agents
Docker agents in pipelines
Kubernetes agents (pod templates, Jenkins K8s plugin)
Agent labeling and restricting jobs to specific agents
Executor count and build queue management

4. SCM Integration (Practical)

GitHub, GitLab, Bitbucket integration
Webhooks vs polling — differences and when to use which
Multibranch pipelines
Branch discovery strategies
Pull request builds and status checks

5. Credentials & Secrets Management (Security round)

Jenkins credentials store types (username/password, SSH, secret text, certificates)
Using credentials in pipelines (withCredentials)
Vault integration with Jenkins
Masking secrets in logs
Comparison to GitHub Actions secrets

6. Jenkins Plugins (Practical knowledge)

Most important plugins to know
Pipeline plugin ecosystem
Blue Ocean
Managing and updating plugins safely
Plugin conflicts and troubleshooting

7. Build Triggers (Commonly asked)

SCM polling vs webhooks
Upstream/downstream job triggers
Cron-based triggers
Remote trigger via API
GitHub PR triggers

8. Artifact Management (CI/CD flow)

Archiving artifacts in Jenkins
Integration with Nexus, Artifactory, S3
Docker image building and pushing
Stashing and unstashing between stages

9. Test Integration & Reporting (Quality rounds)

JUnit test result publishing
Code coverage reports (JaCoCo, Istanbul)
SonarQube integration
Test parallelization strategies
Failing builds based on quality gates

10. Jenkins + Docker (Very commonly asked)

Building Docker images in pipelines
Docker-in-Docker (DinD) — what it is and risks
Using Docker agents
Pushing to DockerHub, ECR, GCR
Layer caching strategies

11. Jenkins + Kubernetes (Senior-level)

Running Jenkins on Kubernetes
Dynamic pod-based agents
Kubernetes plugin configuration
Persistent volume for Jenkins home
Jenkins HA on Kubernetes

12. Jenkins + Cloud (DevOps rounds)

Jenkins with AWS (ECR, EKS, CodeArtifact, S3)
Jenkins with Azure DevOps / AKS
IAM roles for Jenkins agents (IRSA on EKS)
Cloud-specific plugins

13. Security in Jenkins (Security-focused rounds)

Matrix-based security vs Project-based security
Role Strategy Plugin
CSRF protection
Script Security and Groovy sandbox
Securing Jenkins behind a reverse proxy
Audit logging

14. Jenkins High Availability & Scaling (Architectural rounds)

Jenkins HA options (active-passive)
CloudBees HA vs open source limitations
Scaling agents horizontally
Build queue management
Jenkins backup strategies (ThinBackup, config-as-code)

15. Jenkins Configuration as Code (JCasC) (Modern Jenkins)

What is JCasC plugin
Defining Jenkins config in YAML
Managing credentials, agents, plugins via code
GitOps for Jenkins itself
Comparison to how GitHub Actions is configured

16. Shared Libraries (Senior must-know)

Structure of a shared library
vars/ vs src/ vs resources/
Loading libraries (implicit vs explicit)
Versioning shared libraries
Writing reusable steps and functions

17. Pipeline Optimization (Performance rounds)

Reducing pipeline execution time
Parallel stages
Caching dependencies (Maven, npm, pip)
Workspace cleanup strategies
Avoiding redundant stages

18. Monitoring & Observability (Ops rounds)

Jenkins Prometheus plugin
Key metrics: build duration, queue length, agent utilization
Grafana dashboards for Jenkins
Build failure alerting (email, Slack)
Log management for Jenkins

19. Troubleshooting Jenkins (Scenario-based rounds)

Build stuck in queue — how to debug
Out of disk space on master
Agent goes offline mid-build
Pipeline hangs — how to identify
Groovy java.io.NotSerializableException error
Common pipeline anti-patterns


