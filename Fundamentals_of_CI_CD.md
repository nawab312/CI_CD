**Continuous Integration (CI):** Continuous Integration (CI) is a software development practice where developers frequently merge (integrate) their code changes into a shared repository, and every merge triggers an automated build and automated tests.
- Automatic build is the process where the CI system compiles your code, installs dependencies, and packages the application automatically.
- When you push code to Git a CI tool like Jenkins, GitHub Actions, GitLab CI, Azure DevOps automatically:
  - Pulls the latest code
  - Installs dependencies(e.g., npm install, pip install, mvn install)
  - Compiles the application
  - Runs tests
  - Creates artifacts(JAR, WAR, Docker image, ZIP, binaries, etc.)

**Continuous Delivery (CD):** Extends CI by automating the release process, ensuring that software can be deployed at any time. Stages include automated deployment to staging environments, testing, and final release.

**Continuous Deployment (CD):**
- Goes a step further than Continuous Delivery.
- Every change that passes automated tests is automatically deployed to production without manual approval.

![image](https://github.com/user-attachments/assets/44b04fd9-a71e-4a71-a092-c80e646f71c1) ![image](https://github.com/user-attachments/assets/d390e893-2e95-48dd-8307-e81a6cc1defc)
