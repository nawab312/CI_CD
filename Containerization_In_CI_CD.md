### Jenkins Pipeline for Building & Pushing a Docker Image to Docker Hub ###
**Jenkins Agent must have Docker installed.**
- Install Docker on Jenkins agent:
```bash
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
```

**Store Docker Hub credentials in Jenkins:**
- DOCKER_HUB_USERNAME (Type: Secret Text): `sid3121997`
- DOCKER_HUB_PASSWORD (Type: Secret Text): `Sidd97112#`

```groovy
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "sid3121997/sample-app"
        DOCKER_TAG = "latest"  // Docker tags should be lowercase
    }
    stages {
        stage('Pull the Code') {
            steps {
                git branch: 'main', url: 'git@github.com:nawab312/Sample.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:$DOCKER_TAG ."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    string(credentialsId: 'DOCKER_HUB_USERNAME', variable: 'DOCKER_HUB_USERNAME'),
                    string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'DOCKER_HUB_PASSWORD')
                ]) {
                    script {
                        sh """
                        echo "$DOCKER_HUB_PASSWORD" | docker login -u "$DOCKER_HUB_USERNAME" --password-stdin
                        docker push "$DOCKER_IMAGE:$DOCKER_TAG"
                        """
                    }
                }
            }
        }
    }
}
```
