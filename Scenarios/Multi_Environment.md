**Scenario:** Your company has *Dev*, *Staging*, and *Production* environments. How would you design a CI/CD pipeline that ensures proper testing before code reaches production?

- Jenkinsfile https://github.com/nawab312/CI_CD/blob/main/Scenarios/Multi_Environment_Jenkinsfile

### Source Code Management (SCM) Strategy ###
- Use **GitFlow branching strategy** (e.g., `feature`, `develop`, `staging`, `main`)

```bash
main
│
├── develop (for Dev environment)
│   ├── feature/feature-1
│   ├── feature/feature-2
│   ├── bugfix/bugfix-1
│   ├── bugfix/bugfix-2
│
├── release (for Staging environment)
│
└── hotfix/hotfix-1 (for urgent production fixes)
```

- **Feature Branches (`feature/*`)**
  - Who works here? Developers working on new features.
  - Naming Convention: `feature/new-login-ui`, `feature/add-cart-api`.
  - Workflow:
    - A developer creates a new branch from `develop`
    - Commits code frequently and pushes changes.
    - Runs unit *tests & static analysis* before merging.
    - Creates a *Pull Request (PR)* for review.
    - After approval, the feature branch is merged into develop.
- **Develop Branch (`develop`)**
  - Purpose: The integration branch where all feature branches merge.
  - What happens here?
    - CI Pipeline runs → Code quality checks, security scanning, unit tests.
    - Developers validate new features before pushing them forward.
    - If stable, merge into `staging` for QA testing.
- **Staging Branch (`staging`)**
  - Purpose: Acts as a pre-production environment.
  - What happens here?
    - Automated integration & end-to-end (E2E) tests.
    - Performance & security testing before production.
    - If approved → merge into main for production deployment.
- **Main Branch (`main`)**
  - Purpose: Production-ready code only.
  - What happens here?
    - Code is deployed to production via Blue-Green Deployment or Canary Release.
    - If issues arise → rollback using `git revert` or Kubernetes `kubectl rollout undo`.
- **Hotfix Branch (`hotfix/*`)**
  - Purpose: For urgent bug fixes in production.
  - Workflow:
    - Branch out from main (`hotfix/fix-login-error`).
    - Fix the issue → Test → Merge into main and develop.
    - Deploy immediately.

### CI Pipeline (Build & Test in Dev) ###
- This CI pipeline is triggered on every commit to the `develop` branch and consists of:
  - **Linting & Static Analysis**
    - *Linting* checks if the code follows certain rules or *"coding standards"* (like naming conventions or formatting), making it easier for the team to understand and maintain the code.
    - *Static analysis* looks at the code for any problems without actually running it, such as bugs, security vulnerabilities, or bad practices.
    - Tools: SonarQube (Full Static Analysis),  Checkstyle (Java Code Linting Test)
  ```
  # Java: Run Checkstyle & SonarQube
  mvn checkstyle:check
  mvn sonar:sonar -Dsonar.projectKey=myapp
  ```
  - **Unit Tests** → Unit tests are a type of software testing where individual components or units of code, such as functions or methods, are tested in isolation to ensure they work as expected. The primary goal of unit testing is to validate that each unit of the software performs correctly under various conditions. Unit tests are typically executed before the build in a CI pipeline for a few reasons *although this is not always the case*.
    - Unit tests are executed before the build in a CI pipeline to catch errors early, ensuring only code that passes basic functionality checks moves forward to the more resource-intensive build and deployment steps. This saves time and resources by preventing unnecessary builds when tests fail.
  ```
  # Java: Run JUnit Tests
  mvn test
  ```
  - **Build Artifacts** → Create deployable binaries
    - Converts source code into a deployable package.
    - Generates `.jar`, `.war`, or Docker images.
    - Tools: Maven/Gradle (for Java .jar/.war builds), Docker (for containerized apps)
  ```
  # Java: Build a .jar file using Maven
  mvn clean package -DskipTests # -DskipTests: This is a system property passed to Maven that tells Maven to skip running unit tests during the build phase.
  ```
  - **Package & Push to Repository** → Store build outputs for deployment
    - Stores the built artifacts for later use in CD pipelines.
    - Docker Images → Pushed to DockerHub or AWS ECR.
    - JAR/WAR Files → Pushed to Nexus, JFrog Artifactory, S3.

### Deployment to Dev in CI/CD ###

### Deployment to Staging (Automated Testing & Approval) in CI/CD ###
The staging environment is a pre-production space where the application is deployed and tested before it moves to production. Simulates a real-world production environment. Runs integration, performance, and security tests. Ensures stability, security, and reliability before going live. Requires manual or automated approval before production deployment. Once code is merged into the `staging` branch, the following steps occur:
- **Deploy to Staging Environment**
- **Integration & End-to-End (E2E) Tests**
  - Ensures different modules work together correctly. Simulates real user interactions.
  - Tools: Selenium (Automated UI Testing), Postman/Newman (API Testing)
- **Performance & Load Testing**
  - Ensures the system can handle high traffic before production.
  - Identifies slow queries, crashes, and resource limitations.
  - Tools: Apache JMeter (Load Testing)
- **Security Scanning**
  - Detects vulnerabilities before deploying to production.
  - Ensures compliance with OWASP, GDPR, and ISO standards.
  - Tools: OWASP ZAP (Web App Security), Trivy (Container Security)
- **Approval Stage (Manual or Automated)**
  - Ensures the staging build is stable before moving to production.
  - Can be manual (QA team review) or automated (if all tests pass).
  - **Automated Approval:** If all tests pass, automatically merge to `main`
    ```bash
    git checkout main
    git merge staging
    git push origin main
    ```
  - **Manual Approval:** Requires QA sign-off before production. Example Jenkins Pipeline Approval
    ```groovy
    stage('Approval') {
      input {
          message "Deploy to Production?"
          ok "Proceed"
      }
    }
    ```

### Production Deployment (Blue-Green or Canary) in CI/CD ###
Production deployment is the final step in a CI/CD pipeline where the application is released for real users. Key Goals:
- Deploy without downtime.
- Ensure the new version is stable before full rollout.
- Provide a *rollback strategy* in case of failure.
Once the code is merged into the main branch, deployment is triggered using one of these strategies:
- **Blue-Green Deployment**: Two identical production environments: Blue (Current Version - Live), Green (New Version - Staging)
  - **Steps**
    - Deploy the new version to the Green environment.
    - Perform testing & health checks.
    - If stable → switch traffic from Blue → Green.
    - If issues arise → rollback by switching back to Blue.
  - **Use Case:**
    - Zero-downtime deployments.
    - Easy rollback by switching traffic back to Blue.
- **Canary Deployment**: Deploy the new version to a small % of users before full rollout. Gradually increase traffic to the new version while monitoring.
  - **Use Case:**
    - Reduces risk by slowly rolling out changes. Allows monitoring before full deployment.
- **Smoke Tests & Health Checks**: Ensures the new deployment is *healthy & functional* before serving all users.
  - **Tools:** Prometheus (Monitoring), Grafana (Visualization), Kubernetes Liveness & Readiness Probes
- **Rollback Strategy (Keep Last Stable Release)** If the new deployment fails, the system must quickly rollback to the last working version.



    
