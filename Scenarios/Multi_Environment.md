**Scenario:** Your company has *Dev*, *Staging*, and *Production* environments. How would you design a CI/CD pipeline that ensures proper testing before code reaches production?

**Source Code Management (SCM) Strategy**
- Use Git branching strategy (e.g., `feature`, `develop`, `staging`, `main`)
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

**CI Pipeline (Build & Test in Dev)**
- This CI pipeline is triggered on every commit to the `develop` branch and consists of:
  - **Linting & Static Analysis** → Detect code issues early
    - Ensures code follows coding standards.
    - Detects security vulnerabilities & bad practices.
    - Tools: SonarQube (Full static analysis), Pylint (Python), Checkstyle (Java), SAST (Security Analysis with OWASP Dependency Check)
  ```
  # Java: Run Checkstyle & SonarQube
  mvn checkstyle:check
  mvn sonar:sonar -Dsonar.projectKey=myapp
  # JavaScript: Run ESLint
  eslint src/
  # Python: Run Pylint
  pylint my_project/
  ```
  - **Unit Tests** → Unit tests are a type of software testing where individual components or units of code, such as functions or methods, are tested in isolation to ensure they work as expected. The primary goal of unit testing is to validate that each unit of the software performs correctly under various conditions. Unit tests are typically executed before the build in a CI pipeline for a few reasons *although this is not always the case*.
    - Unit tests are executed before the build in a CI pipeline to catch errors early, ensuring only code that passes basic functionality checks moves forward to the more resource-intensive build and deployment steps. This saves time and resources by preventing unnecessary builds when tests fail.
  ```
  # Java: Run JUnit Tests
  mvn test
  # JavaScript: Run Jest Tests
  npm test
  # Python: Run PyTest
  pytest --cov=my_project
  ```
  - **Build Artifacts** → Create deployable binaries
    - Converts source code into a deployable package.
    - Generates `.jar`, `.war`, or Docker images.
    - Tools: Maven/Gradle (for Java .jar/.war builds), Docker (for containerized apps)
  ```
  # Java: Build a .jar file using Maven
  mvn clean package
  # Java: Build a .war file using Gradle
  gradle build
  # Node.js: Build an optimized app
  npm run build
  # Docker: Build a container image
  docker build -t myapp:latest .
  ```
  - **Package & Push to Repository** → Store build outputs for deployment
    - Stores the built artifacts for later use in CD pipelines.
    - Docker Images → Pushed to DockerHub or AWS ECR.
    - JAR/WAR Files → Pushed to Nexus, JFrog Artifactory.
