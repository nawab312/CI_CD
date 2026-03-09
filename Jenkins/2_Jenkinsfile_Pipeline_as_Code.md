**Declarative vs Scripted Pipeline**
- Declarative Pipeline: Declarative is the modern, structured way to write pipelines. It has a fixed structure and is easier to read.
```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'myapp'
        DOCKER_REGISTRY = 'myregistry.com'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```
- Scripted Pipeline: Scripted is the older, flexible way. It is pure Groovy code, giving you full programming power.
```groovy
node {
    def appName = 'myapp'
    def dockerRegistry = 'myregistry.com'

    try {
        stage('Build') {
            sh 'mvn clean package'
        }

        stage('Test') {
            sh 'mvn test'
        }

        stage('Deploy') {
            sh 'kubectl apply -f k8s/'
        }

        echo 'Pipeline succeeded!'

    } catch (Exception e) {
        echo "Pipeline failed: ${e.message}"
        throw e

    } finally {
        echo 'This always runs'
        cleanWs()
    }
}
```
```
| Feature | Declarative | Scripted |
|---|---|---|
| Syntax | Structured, fixed | Pure Groovy, flexible |
| Error handling | `post {}` block | `try/catch/finally` |
| Learning curve | Easy | Harder |
| Validation | Validated before run | Fails at runtime |
| Flexibility | Limited | Full Groovy power |
| Recommended for | Most use cases | Complex logic only |
| Starts with | `pipeline {}` | `node {}` |
```
