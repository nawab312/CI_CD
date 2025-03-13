- SonarQube is an open-source platform used for **static code analysis** to detect *code quality issues*, *security vulnerabilities*, *bugs*, and *code smells*. It supports multiple programming languages and integrates seamlessly into CI/CD pipelines to enforce quality gates before deploying code to production.
- SonarQube performs an automatic code review that provides detailed reports on:
  - Bugs: Potential issues in your code that could cause it to fail or behave unexpectedly.
  - Vulnerabilities: Security flaws or weaknesses that could be exploited.
  - Code Smells: Pieces of code that might not be wrong but could be improved for better maintainability and readability.
  - Duplications: Sections of code that are duplicated and should be refactored to improve maintainability.
  - Test Coverage: The percentage of the code that is covered by unit tests, helping ensure that the application is well-tested.

- In Jenkins CI/CD, SonarQube is typically used in the **build stage** to analyze source code before proceeding with further deployment. The process includes:
  - **Pre-Build Setup:**
    - SonarQube Server is set up, and the SonarScanner CLI or SonarQube Jenkins plugin is installed.
    - SonarQube tokens and credentials are configured in Jenkins for authentication.
  - **Build Stage:**
    - The Jenkins pipeline triggers a *SonarScanner* analysis after compiling the code.
    - The analysis results (bugs, vulnerabilities, duplications, etc.) are sent to the SonarQube server.
  - **Quality Gate Validation:**
    - SonarQube applies quality gates (e.g., max allowable bugs, code coverage threshold).
    - If the code fails the quality gate, the Jenkins pipeline can be configured to stop further stages (e.g., test, deploy).
  - **Reporting & Feedback:**
    - The results are stored in the SonarQube UI for developers and DevOps teams to review.
    - Jenkins can fetch the SonarQube report and fail the pipeline if issues exceed thresholds.

```bash
pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/example/repo.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
    }
}
```

