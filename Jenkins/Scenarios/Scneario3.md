Selectively execute specific stages based on a user-provided parameter.

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'stagesToExecute', defaultValue: "")
    }
    stages {
        stage('Stage1') {
            when {
                expression { params.stagesToExecute.contains('Stage1') }
            }
            steps {
                echo 'Executing Stage 1'
            }
        }
        stage('Stage2') {
            when {
                expression { params.stagesToExecute.contains('Stage2') }
            }
            steps {
                echo 'Executing Stage 2'
            }
        }
    }
}
```
