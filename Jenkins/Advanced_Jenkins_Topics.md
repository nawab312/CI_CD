### Jenkins Performance Optimization ###

**Pipeline Design Optimization**
- Pipeline as Code (`Jenkinsfile`):
  - All pipelines are defined declaratively using `Jenkinsfile`, which ensures consistency and version control.
  - Shared Libraries are used to centralize common logic and avoid duplication across multiple pipelines.
- Pipeline Stage Timeouts:
  - Implement `timeout()` for each stage to prevent jobs from hanging and blocking executors indefinitely.

*Shared Libraries*

Jenkins Shared Libraries are reusable, version-controlled Groovy scripts that centralize common pipeline logic (like build, deploy, notifications) across multiple `Jenkinsfiles`. They help eliminate code duplication, improve maintainability, and enforce standard CI/CD practices across teams.

Imagine you’re working on 5 microservices. Each one has a Jenkins pipeline that does this:
- Checkout code
- Build Docker image
- Push image to ECR
- Deploy to Kubernetes

Now, repeating this same logic in 5 `Jenkinsfiles` means duplicating code. That’s a maintenance nightmare. Shared Library = Place to write common Jenkins code once, and use it in all Jenkinsfiles. Step-by-Step Example
- Create the Shared Library Repo
  ```bash
  git@github.com:your-org/jenkins-shared-libraries.git
  ```
- Folder Structure
  ```bash
  jenkins-shared-libraries/
  └── vars/
    └── deployApp.groovy
  ```
- Inside `deployApp.groovy`
  ```groovy
  def call(String serviceName) {
    stage("Build Docker Image") {
        sh "docker build -t ${serviceName}:latest ."
    }
    stage("Push to ECR") {
        sh "docker push ${serviceName}:latest"
    }
    stage("Deploy to Kubernetes") {
        sh "kubectl apply -f k8s/${serviceName}-deployment.yaml"
    }
  }
  ```
- In Your Jenkinsfile (for any service)
```groovy
@Library('jenkins-shared-lib') _   // Load the shared library

pipeline {
    agent any

    stages {
        stage("CI/CD Pipeline") {
            steps {
                deployApp("orders-service")   // Reuse the shared function
            }
        }
    }
}
```
- How to Register the Library in Jenkins
  - Manage Jenkins → Configure System → Global Pipeline Libraries
  - Name: `jenkins-shared-lib`
  - Default version: `main` (or tag/branch)
  - Source Code Management: Git
  - Project repository: `https://github.com/your-org/jenkins-shared-libraries.git`

*timeout()* 

timeout is a safety mechanism in pipeline stages that limits how long a stage or job is allowed to run. If it exceeds the time, the job is automatically aborted. This prevents:
- Hanging jobs
- Blocked runners/executors
- Wasted compute resources
- Pipeline deadlocks
```groovy
pipeline {
  agent any
  stages {
    stage('Test') {
      options {
        timeout(time: 15, unit: 'MINUTES')
      }
      steps {
        sh './run-tests.sh'
      }
    }
  }
}
```

**Job Configuration and Artifact Management**
- Archiving Only Needed Artifacts:
  - Avoid archiving unnecessary files to reduce storage I/O and Jenkins load.
- Retention Policies:
  - Set rules to keep only the latest N successful and failed builds to maintain a clean and lightweight system.
- Post-Build Cleanup:
  - Automatically clean up workspace using `Workspace Cleanup Plugin` or scripted logic.
 
**Executor and Load Management**
- Dedicated Agents:
  - Master node runs with zero executors to handle orchestration only.
  - Builds are delegated to labeled agents based on resource needs (e.g., memory-intensive, GPU builds).
- Load Distribution:
  - Jenkins Queue Management and appropriate node labels ensure heavy workloads don’t overwhelm specific agents.
 
**Plugin and Dependency Management**
- Minimal Plugin Footprint:
  - Only essential plugins are installed to reduce memory usage, security vulnerabilities, and upgrade complexities.
- Frequent Plugin Updates:
  - Regularly update plugins to take advantage of performance improvements and security patches.


**Parallel Execution**

Parallel execution in Jenkins CI/CD pipelines allows multiple independent tasks (such as builds, tests, deployments) to run simultaneously instead of sequentially. This significantly reduces overall execution time and improves pipeline efficiency. Why Parallel Execution?
- Speeds up the pipeline → Tasks run at the same time instead of waiting for each other.
- Efficient resource utilization → Jenkins agents can process multiple jobs simultaneously.
- Scalability → Supports distributed execution across multiple Jenkins agents.
- Best for running independent tasks (e.g., Unit Tests, Integration Tests, UI Tests).

How Jenkins Handles Parallel Execution
- Pipeline is triggered → Jenkins master parses the pipeline script.
- Stages are evaluated → Jenkins identifies parallel stages that can run concurrently.
- Jenkins assigns each stage to an available agent → If multiple agents are configured, Jenkins distributes tasks across them.
- Tasks execute in parallel → Each task runs in its own workspace, environment, and container (if using Kubernetes).
- Completion check → The pipeline waits for all parallel stages to complete before proceeding to the next stage.

```groovy
pipeline {
    agent any
    stages {
        stage('Parallel Test Execution') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'run-unit-tests.sh'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'run-integration-tests.sh'
                    }
                }
                stage('UI Tests') {
                    steps {
                        sh 'run-ui-tests.sh'
                    }
                }
            }
        }
    }
}
```

```groovy
pipeline {
    agent none
    stages {
        stage('Parallel Test Execution') {
            parallel {
                stage('Unit Tests') {
                    agent { label 'test-agent' }  // Assigns a dedicated agent
                    steps {
                        sh 'run-unit-tests.sh'
                    }
                }
                stage('Integration Tests') {
                    agent { label 'test-agent' }
                    steps {
                        sh 'run-integration-tests.sh'
                    }
                }
            }
        }
    }
}
```

Handling Failures in Parallel Execution
- Fail the pipeline immediately if any stage fails
```groovy
pipeline {
    agent any
    stages {
        stage('Parallel Execution') {
            failFast true  // Stops all parallel executions if any stage fails
            parallel {
                stage('Test A') {
                    steps {
                        sh 'exit 1'  // Simulating failure
                    }
                }
                stage('Test B') {
                    steps {
                        sh 'echo "Test B Running..."'
                    }
                }
            }
        }
    }
}
```

- Continue execution even if one stage fails
```groovy
pipeline {
    agent any
    stages {
        stage('Parallel Execution') {
            parallel(
                'Safe Stage A': {
                    catchError(buildResult: 'SUCCESS') {
                        sh 'exit 1'  // Simulating failure
                    }
                },
                'Safe Stage B': {
                    sh 'echo "This stage will run even if Stage A fails"'
                }
            )
        }
    }
}
```

When to Use Parallel Execution in Jenkins?
- Running multiple independent test suites (Unit, API, UI).
- Executing performance tests while running functional tests.
- Deploying to multiple environments (Dev, QA, Staging, Production) simultaneously.

When Not to Use Parallel Execution in Jenkins?
- Stages depend on each other (e.g., Build → Test → Deploy).
- You are sharing a workspace, as parallel execution might cause conflicts.
- Tests are resource-intensive and may overload Jenkins agents.



**Self-Healing Pipelines**

A self-healing pipeline is a CI/CD pipeline that automatically detects and recovers from failures without manual intervention. It ensures high availability, reliability, and stability in software delivery.

What are Auto-Retry Mechanisms?

Auto-retry mechanisms allow a pipeline to automatically retry failed steps caused by transient issues, such as:
- Network glitches
- Temporary service unavailability
- Flaky tests
- Intermittent database connection issues
- Infrastructure failures (e.g., Kubernetes pod restarts)

Implementing Auto-Retry in Jenkins Pipelines

Using the `retry` Directive in Declarative Pipelines
```groovy
pipeline {
    agent any
    stages {
        stage('Test API') {
            steps {
                retry(3) {  // Retries this block up to 3 times
                    sh 'curl -s http://unstable-api.com/health'  // API might fail due to network issues
                }
            }
        }
    }
}
```

Using Try-Catch Blocks
- For advanced retry handling, a try-catch block can be used to catch failures and retry based on conditions.
- Retries the API test up to 3 times before marking the build as failed.
- Uses `sleep(5)` to wait 5 seconds before retrying.
- If all 3 attempts fail, it explicitly marks the build as failed with `error`.
```groovy
pipeline {
    agent any
    stages {
        stage('Test API') {
            steps {
                script {
                    def maxRetries = 3
                    def attempt = 0
                    while (attempt < maxRetries) {
                        try {
                            sh 'curl -s http://unstable-api.com/health'  // May fail due to network issues
                            echo "API Test Passed on Attempt ${attempt + 1}"
                            break  // Exit loop if successful
                        } catch (Exception e) {
                            attempt++
                            echo "Attempt ${attempt} failed, retrying..."
                            sleep(5)  // Wait 5 seconds before retrying
                        }
                    }
                    if (attempt == maxRetries) {
                        error "API Test failed after ${maxRetries} attempts"
                    }
                }
            }
        }
    }
}
```

Using post Block for Conditional Retries
- The `post` block can automatically trigger a retry or rollback if a stage fails.
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy to Staging') {
            steps {
                script {
                    try {
                        sh './deploy.sh'
                    } catch (Exception e) {
                        echo 'Deployment failed, retrying...'
                        sh './deploy.sh'  // Retry deployment once
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment succeeded! Notifying the team...'
            sh 'curl -X POST -H "Content-Type: application/json" --data \'{"text": "Deployment Succeeded!"}\' https://slack-webhook-url'
        }
        failure {
            echo 'Deployment failed! Triggering rollback...'
            sh './rollback.sh'  // Example: Rollback if deployment fails
        }
        always {
            echo 'Cleaning up temporary files...'
            sh 'rm -rf /tmp/build-artifacts'  // Clean up workspace
        }
    }
}
```
