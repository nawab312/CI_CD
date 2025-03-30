### CI/CD Fundamentals ###
- **What is CI/CD?**
  - **Continuous Integration (CI), Continuous Deployment (CD), Continuous Delivery (CD)**
  - **CI/CD Benefits & Workflow**
- **CI/CD Best Practices**
- **CI/CD Pipeline Components**
  - **Source Code Management (SCM)**
  - **Build**
  - **Test (Unit, Integration, Functional, Security)**
  - **Deployment**
  - **Monitoring & Feedback Loop**

2. Jenkins Fundamentals

    What is Jenkins?

    Jenkins Architecture

        Master-Agent Setup

        Jenkins Jobs & Pipelines

    Installation & Setup

        Installing Jenkins on Linux, Windows, Docker

        Jenkins Configuration

        Managing Plugins

        Security Best Practices (User Roles, Authentication)

        Jenkins Backup & Restore

### Jenkins Pipeline & Scripting ###
- **Jenkins Pipeline Basics**
  - **Freestyle Jobs vs Pipelines**
  - **Declarative vs Scripted Pipelines** 
- **Jenkinsfile Syntax**
  - **Stages & Steps**
  - **Environment Variables & Parameters**
  - **Post Build Actions**
- **Pipeline as Code**
  - **Writing & Storing Jenkinsfiles in Git**
  - **Shared Libraries in Pipelines**
- **Advanced Groovy Scripting in Jenkins**
  - **Dynamic Pipeline Creation**
  - **Looping & Conditional Execution**
- **Pipeline Execution Optimization**
  - **Throttle Concurrent Builds and Cancel Previous Running Jobs**
  - **Using the quietPeriod directive**
  - **Handling Job Queue & Resource Utilization**

4. Source Code Management (SCM)

    Jenkins & Git Integration

        Webhooks & Polling

        Multi-Branch Pipelines

        GitHub & GitLab Integration

    Branching Strategies

        Feature Branching

        Gitflow

        Trunk-based Development

### Build & Test Stages ###
- **Build Tools**
 - **Maven, Gradle, NPM, Makefile**
 - **Dependency Management**
- **Automated Testing**
 - **Unit Tests (JUnit, PyTest)**
 - **Integration Tests**
 - **Code Quality & Security Scanning**
 - **SonarQube & SAST Tools Integration**

6. Artifact Management

    Storing Build Artifacts

        Nexus, Artifactory, AWS S3, Docker Hub

    Versioning & Tagging Releases

    Artifact Promotion Strategies

### Containerization & Orchestration in CI/CD ###
- **Jenkins with Docker**
 - **Building & Pushing Docker Images**
 - **Running Jenkins in Docker**
- **Jenkins with Kubernetes**
 - **Deploying Jenkins on Kubernetes**
 - **Running CI/CD Pipelines in Kubernetes (Jenkins Agents on Kubernetes)**
- **Helm & Jenkins**
 - **Managing Helm Charts in Jenkins**
- **Jenkins X for Cloud-Native CI/CD**

### Deployment Strategies ###
- **Traditional vs Modern Deployments**
- **Blue-Green Deployment**
- **Canary Deployment**
- **Rolling Deployments**
- **Feature Toggles & Dark Launching**
- **Zero Downtime Deployment Strategies**

9. Infrastructure as Code (IaC) in CI/CD

    Jenkins & Terraform

        Automating AWS Infrastructure Deployment

    Jenkins & Ansible

        Configuration Management

        Automated Deployments

### Security in CI/CD ###
- **Jenkins Security Best Practices**
 - **User Authentication & Role-Based Access Control (RBAC)**
 - **Secrets Management (Vault, AWS Secrets Manager, Jenkins Credentials)**
- **DevSecOps in Jenkins**
 - **Static & Dynamic Security Scans (SAST & DAST)**
 - **OWASP Dependency-Check, Trivy, Aqua Security**
 - **Penetration Testing (Pentesting) in CI/CD**
- **Software Supply Chain Security**
 - **Signed Commits & Code Reviews**
 - **SBOM (Software Bill of Materials)**
 - **Signing Artifacts & Images**  

11. Monitoring & Observability in CI/CD

    Jenkins Logging & Monitoring

        Logging with ELK / EFK

        Monitoring with Prometheus & Grafana

    Alerting Mechanisms

        Slack / Email / PagerDuty Notifications

12. Jenkins & Cloud Integrations

    Jenkins on AWS

        AWS EC2, S3, IAM, ECR, ECS, Lambda

    Jenkins with AWS CodePipeline

    Jenkins on Azure & GCP

    Serverless CI/CD Pipelines

### Advanced Jenkins Topics ###
- **Jenkins Performance Optimization**
 - **Distributed Builds**
 - **Parallel Execution**
- **Self-Healing Pipelines**
 - **Auto-retry Mechanisms**
- **CI/CD Metrics & Analytics**
 - **DORA Metrics (Deployment Frequency, Lead Time, MTTR, Change Failure Rate)**

