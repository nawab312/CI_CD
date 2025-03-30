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
