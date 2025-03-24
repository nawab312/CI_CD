### Fundamentals of CI/CD ###
- **What is CI/CD?**
- **Importance of CI/CD in Software Development**
- **CI vs CD vs Continuous Deployment**
- **CI/CD Pipeline Architecture**
- **Challenges in CI/CD Implementation**
- https://github.com/nawab312/CI_CD/blob/main/Fundamentals_of_CI_CD.md

### Version Control in CI/CD ###
- **Git Basics (Commit, Merge, Rebase, Cherry-Pick)**
- **Git Workflows (GitFlow, Trunk-Based Development)**
- **Feature Branching and Pull Requests**
- **Webhooks & Git Triggers in CI/CD**
- https://github.com/nawab312/CI_CD/blob/main/GIT.md

### CI/CD Tools and Platforms ####
- **CI/CD Servers:**
  - **Jenkins**
  - **GitHub Actions**
  - **GitLab CI/CD**
  - **Bitbucket Pipelines**
  - **Azure DevOps**
  - **CircleCI**
  - **Travis CI**
- **CD Tools:**
  - **ArgoCD**
  - **Spinnaker**
  - **FluxCD**
  - **Tekton**

4. CI/CD Pipeline Components

    Source Code Management (SCM) Integration

    Build Automation

    Testing Automation (Unit, Integration, E2E, Security Tests)

    Artifact Management (Nexus, Artifactory, Docker Registry)

    Deployment Automation

5. Jenkins for CI/CD

    Jenkins Installation and Setup

    Jenkins Pipeline (Declarative vs Scripted)

    Managing Jenkins Plugins

    Jenkinsfile: Syntax and Best Practices

    Jenkins Agents and Distributed Builds

    Jenkins Security (RBAC, Credentials, Secrets Management)

6. Infrastructure as Code (IaC) in CI/CD

    Terraform in CI/CD Pipelines

    Ansible for Configuration Management in CI/CD

    CloudFormation for AWS Deployments

    Pulumi in CI/CD

7. Continuous Integration (CI)

    Code Quality Checks (Linting, Code Formatting)

    Automated Builds (Maven, Gradle, NPM, Go, Python)

    Unit Testing & Code Coverage

    Static Code Analysis (SonarQube, Checkstyle)

    Handling Merge Conflicts in CI

8. Continuous Delivery (CD)

    Blue-Green Deployment

    Canary Deployment

    Rolling Deployments

    Feature Flags and Dark Launching

    Progressive Delivery

9. Containerization in CI/CD

    Dockerfile Best Practices

    Multi-Stage Builds in Docker

    Docker Build Caching & Optimization

    Container Image Scanning (Trivy, Clair, Anchore)

10. Kubernetes in CI/CD

    Deploying Applications on Kubernetes

    Helm for Kubernetes Deployment

    Kubernetes Rolling Updates & Rollbacks

    CI/CD Pipeline for Kubernetes with ArgoCD

    Kubernetes Operators for CI/CD

11. Security in CI/CD (DevSecOps)

    Secrets Management (Vault, AWS Secrets Manager, KMS)

    Security Scanning (Snyk, Trivy, Aqua Security)

    Dependency Scanning & SBOM (Software Bill of Materials)

    Compliance and Policy as Code (Open Policy Agent)

    Zero Trust CI/CD

12. Monitoring & Observability in CI/CD

    Logging in CI/CD (ELK, EFK)

    Monitoring Pipelines (Prometheus, Grafana)

    Distributed Tracing in CI/CD (Jaeger, OpenTelemetry)

    Alerting in CI/CD Pipelines

13. Database CI/CD

    Database Schema Migrations in CI/CD

    Tools for Database CI/CD (Flyway, Liquibase)

    Managing Database Rollbacks in CI/CD

14. Debugging & Troubleshooting CI/CD Pipelines

    Common CI/CD Failures and Fixes

    Debugging Jenkins Pipelines

    Troubleshooting Kubernetes Deployments

    Handling Build Failures in CI/CD

15. Advanced CI/CD Topics

    AI & ML in CI/CD (Automated Pipeline Optimization)

    Self-Healing CI/CD Pipelines

    Serverless CI/CD (AWS Lambda, Google Cloud Functions)

    GitOps vs Traditional CI/CD

**Continuous Integration (CI):** Automates the integration of code changes from multiple contributors into a single project. It involves: 
- Frequent code commits to a shared repository.
- Automated build and testing processes.

**Continuous Delivery (CD):** Extends CI by automating the release process, ensuring that software can be deployed at any time. Stages include automated deployment to staging environments, testing, and final release.

**Continuous Deployment (CD):**
- Goes a step further than Continuous Delivery.
- Every change that passes automated tests is automatically deployed to production without manual approval.

![image](https://github.com/user-attachments/assets/44b04fd9-a71e-4a71-a092-c80e646f71c1) ![image](https://github.com/user-attachments/assets/d390e893-2e95-48dd-8307-e81a6cc1defc)

**How do you resolve dependencies in a Continuous Integration (CI) pipeline?**

- CI/CD dependency resolution and build consistency: https://github.com/nawab312/CI_CD/blob/main/Jenkins/Scenarios/Scenario7.md

Dependency resolution is a critical step to ensure reproducibility and consistency across different environments. The approach depends on the technology stack, but in general, we follow these best practices:
- **Using Dependency Lock Files:**
  - *Why Do We Need Lock Files?* When we install dependencies in a project, they are downloaded from the internet. But if we don’t specify exact versions, the dependencies *can update automatically*.
  - Example Problem (Without a Lock File):
    - Today, you run `npm install`, and it installs *lodash v4.17.21*.
    - Next week, another developer runs npm `install`, but lodash updated to *v4.18.0* automatically.
    - This could break the project because the new version may have changes or bugs.
  - We rely on lock files (`package-lock.json` for Node.js, `requirements.txt` or `poetry.lock` for Python, `Gemfile.lock` for Ruby, `pom.xml` with a specific version for Maven, etc.).
  - This ensures that every CI run installs the exact *same dependency versions as in development*, avoiding inconsistencies.
- **Caching Dependencies to Improve Performance:**
  - When we run a CI/CD pipeline, it often downloads the same dependencies (libraries, packages) every time. This can slow down builds. To make the process faster, we *cache dependencies*, so they don’t need to be re-downloaded in every pipeline run.
- **Using a Centralized Package Repository**
  - In CI/CD pipelines, dependencies (libraries, frameworks, tools) are downloaded from the internet (e.g., npm registry, Maven Central, PyPI). But this can cause problems:
    - External Failures → If the internet is down or the public repository is unavailable, builds fail.
    - Security Risks → Public dependencies may contain vulnerabilities.
    - Slow Builds → Downloading dependencies every time increases build time.
  - *Solution: Use a Centralized Package Repository*: Instead of downloading dependencies directly from the internet, companies host their own internal repository using tools like: Nexus, JFrog Artifactory
    - Example: Using an Internal Repository:
      - For Maven (Java projects): Instead of downloading from Maven Central, we use an internal repository. This tells Maven to fetch dependencies from the internal Nexus/Artifactory repo instead of public sources.
        ```xml
        <repositories>
          <repository>
            <id>internal-repo</id>
            <url>https://nexus.mycompany.com/repository/maven-public/</url>
          </repository>
        </repositories>
        ```
- **Running Dependency Security & Vulnerability Scans**
  - Security scanning tools check for *vulnerable or outdated dependencies* in CI/CD.
  - Popular Security Tools:
    - *Snyk* `snyk test`
    - *Trivy (For Docker & Kubernetes)* Scans container images and dependencies for security issues.
    - *OWASP Dependency-Check* Checks for known vulnerabilities using OWASP’s security database.

- **Jenkins** https://github.com/nawab312/CI_CD/blob/main/Jenkins/Notes.md

- **Multi-Environment CI/CD Pipeline with Jenkins (GITFLOW  BRANCHING STRATERGY)** https://github.com/nawab312/CI_CD/blob/main/Scenarios/Multi_Environment.md

- **Zero-Downtime Deployment for a Kubernetes App** https://github.com/nawab312/CI_CD/blob/main/Scenarios/Zero-Downtime%20_Deployment_for_Kubernetes_App.md

- **Sonarqube** https://github.com/nawab312/CI_CD/blob/main/SonarQube/Notes.md

### CI/CD Pipeline Troubleshooting & Debugging ###
- **Debugging Failed Builds**
- **Fixing Deployment Issues**
- **Handling CI/CD Pipeline Failures**
- https://github.com/nawab312/CI_CD/blob/main/CI_CD_%20Pipeline_Troubleshooting_%26_Debugging/Notes.md


