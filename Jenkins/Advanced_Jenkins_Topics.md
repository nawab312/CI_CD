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
