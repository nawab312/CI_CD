## Secrets Management ##

Scenario: Your pipeline needs to access AWS credentials and database passwords. How would you securely manage these secrets in a CI/CD pipeline?

Follow-ups:
- Would you store secrets in environment variables, Vault, AWS Secrets Manager, or another solution? Why?
- How would you prevent secrets from being leaked into logs?

**Solution**
- **AWS Secrets Manager (Recommended)**:
  - Why? AWS Secrets Manager securely stores, rotates, and manages access to secrets.
  - How? Use the AWS Secrets Manager Plugin in Jenkins to fetch secrets dynamically during pipeline execution
- **Jenkins Credentials Store**
  - Why? Jenkins has a built-in credentials store that encrypts secrets, ensuring they are not exposed in plain text.
  - How? Store credentials as "Secret Text", "Username/Password", or "AWS Credentials" and retrieve them using the `withCredentials{}` block.
 
**Preventing Secret Leakage**
- **Mask Secrets in Logs**
  - Use Jenkins `Mask Passwords` plugin to prevent secrets from appearing in console logs.
  - Example:
  ```groovy
  withCredentials([string(credentialsId: 'db-password', variable: 'DB_PASSWORD')]) {
    sh 'echo "****"'
  }
  ```
- **Restrict Access to Secrets**
  - Use RBAC (Role-Based Access Control) to limit who can view or modify credentials in Jenkins.
  - Restrict access to credentials at the folder/job level in Jenkins.
- **Use IAM Roles Instead of Static Credentials**
  - Instead of storing AWS credentials, assign IAM roles with least privilege access to Jenkins agents.


### AWS Secrets Manger Methods ###

**Approach 1: Using AWS Secrets Manager Plugin**
- Install *AWS Secrets Manager Credentials Provider* Plugin
- Configure AWS IAM Permissions: The Jenkins instance (or agent) should have an IAM role or AWS access keys with these permissions:
```json
{
  "Effect": "Allow",
  "Action": [
    "secretsmanager:GetSecretValue",
    "secretsmanager:ListSecrets"
  ],
  "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:*"
}
```
- Use the Secret in Jenkins Pipeline
  - Store a secret in *AWS Secrets Manager* (e.g., `my-db-password`).
```groovy
pipeline {
    agent any
    environment {
        DB_PASSWORD = credentials('my-db-password')  // Fetch from AWS Secrets Manager
    }
    stages {
        stage('Use Secret') {
            steps {
                sh 'echo "Database Password: ****"'  // Secret is masked
            }
        }
    }
}
```

**Approach 2: Using AWS CLI in Jenkins**
- If the plugin is not available, AWS CLI can fetch secrets dynamically.
- Ensure AWS CLI is Installed on Jenkins agents.
- Set Up IAM Permissions (Same as above).

```groovy
pipeline {
    agent any
    stages {
        stage('Retrieve Secret') {
            steps {
                script {
                    def secret = sh(script: "aws secretsmanager get-secret-value --secret-id my-db-password --query SecretString --output text", returnStdout: true).trim()
                    env.DB_PASSWORD = secret  // Store secret in environment variable
                }
            }
        }
        stage('Use Secret') {
            steps {
                sh 'echo "DB Password: ****"'  // Avoid printing raw secrets
            }
        }
    }
}
```


