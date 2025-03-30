**Secrets Management (Vault, AWS Secrets Manager, Jenkins Credentials)**
Managing secrets securely is critical in CI/CD pipelines to prevent unauthorized access, data breaches, and accidental exposure of sensitive information. 
In a CI/CD environment, secrets include API keys, database credentials, SSH keys, and cloud access tokens, which must be stored securely and retrieved dynamically during pipeline execution.


![image](https://github.com/user-attachments/assets/7068e95a-2f68-4096-ae04-fd1166ec2b1e)

*Jenkins Credentials Store*

Jenkins provides a built-in credentials store to securely store and manage secrets used in pipelines.
- Go to Jenkins Dashboard → Manage Jenkins → Manage Credentials.
- Select Global credentials (or another appropriate scope).
- Click Add Credentials and enter:
  - Type: Secret text, username/password, or a file.
  - ID: Unique identifier (e.g., `MY_SECRET`).
```groovy
pipeline {
    agent any
    environment {
        API_KEY = credentials('MY_SECRET') // Retrieves secret securely
    }
    stages {
        stage('Deploy') {
            steps {
                withCredentials([string(credentialsId: 'MY_SECRET', variable: 'SECRET')]) {
                    sh 'echo "Using secret safely without exposing it in logs"'
                }
            }
        }
    }
}
```

*AWS Secrets Manager*

AWS Secrets Manager securely stores, retrieves, and rotates secrets such as database passwords, API keys, and encryption keys.
- Secrets are encrypted and stored securely in AWS.
- IAM roles ensure only authorized users/services can access them.
- Automatic secret rotation for credentials like RDS passwords.

How to Use AWS Secrets Manager in a CI/CD Pipeline
- Go to AWS Console → Secrets Manager → Store a New Secret.
- Choose a secret type (Key-Value, RDS credentials, etc.).
- Assign a name (e.g., `/dev/api_key`).
- Configure IAM permissions to allow Jenkins to access secrets.
```groovy
pipeline {
    agent any
    stages {
        stage('Fetch Secrets') {
            steps {
                script {
                    def secretValue = sh(script: "aws secretsmanager get-secret-value --secret-id /dev/api_key --query SecretString --output text", returnStdout: true).trim()
                    env.API_KEY = secretValue
                }
                sh 'echo "Using secret securely in pipeline..."'
            }
        }
    }
}
```

*HashiCorp Vault*

HashiCorp Vault is a dynamic secrets management tool designed for multi-cloud and on-prem environments.
- Secrets are not stored in CI/CD systems—retrieved dynamically.
- Support for automatic expiration and revocation of secrets.
- Supports AWS, Kubernetes, databases, and more.

How to Integrate HashiCorp Vault in CI/CD Pipelines
- Deploy Vault server and enable secrets engine (e.g., `kv-v2`).
- Create a secret at `secret/data/my-app`:
```json
{
    "api_key": "my-secure-api-key"
}
```
- Set up authentication for Jenkins (e.g., AppRole, Token).
```groovy
withVault(configuration: [vaultUrl: 'https://vault.example.com'], secrets: [
    [path: 'secret/data/my-app', engineVersion: 2, secretValues: [
        [envVar: 'API_KEY', vaultKey: 'api_key']
    ]]
]) {
    sh 'echo "Using API Key securely..."'
}
```
