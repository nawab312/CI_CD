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

- **How to Handle Multi Environment-Specific Configurations in a CI/CD pipeline** https://github.com/nawab312/CI_CD/blob/main/Scenarios/Multi_Environment.md

- **Zero-Downtime Deployment for a Kubernetes App** https://github.com/nawab312/CI_CD/blob/main/Scenarios/Zero-Downtime%20_Deployment_for_Kubernetes_App.md


