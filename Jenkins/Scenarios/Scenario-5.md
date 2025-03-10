Using Configuration Files for Each Environment

- You can create separate configuration files for different environments (e.g., `config.dev.json`, `config.staging.json`, `config.prod.json`)

```groovy
pipeline {
    agent any
    environment {
        CONFIG_FILE = "config-dev.json"  // Default to dev
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo.git'
            }
        }
        stage('Select Config') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.CONFIG_FILE = "config-prod.json"
                    } else if (env.BRANCH_NAME == 'staging') {
                        env.CONFIG_FILE = "config-staging.json"
                    }
                }
                sh "cp config/${CONFIG_FILE} config.json"
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying using ${CONFIG_FILE}"
                sh "kubectl apply -f deployment.yaml"
            }
        }
    }
}
```
