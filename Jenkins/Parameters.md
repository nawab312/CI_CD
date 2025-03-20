Jenkins parameters allow you to pass values dynamically to your Jenkins jobs at runtime. This is useful for customizing builds without modifying the pipeline code.

![image](https://github.com/user-attachments/assets/4f2be46c-2988-418c-83a3-104716ce78da)

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'USERNAME', defaultValue: 'admin', description: 'Enter your username')
        booleanParam(name: 'DEPLOY_TO_PROD', defaultValue: false, description: 'Deploy to production?')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Select environment')
    }
    stages {
        stage('Build') {
            steps {
                echo "Building application for ${params.ENV}"
            }
        }
        stage('Deploy') {
            when {
                expression { params.DEPLOY_TO_PROD }
            }
            steps {
                echo "Deploying to production..."
            }
        }
    }
}
```

### Special Parameter Types ###

- **File Parameter** Allows users to upload a file and process it during the build.
```groovy
pipeline {
    agent any
    parameters {
        file(name: 'UPLOAD_FILE', description: 'Upload a file')
    }
    stages {
        stage('Process File') {
            steps {
                script {
                    def filePath = params.UPLOAD_FILE
                    echo "File uploaded: ${filePath}"
                }
            }
        }
    }
}
```

- **Password Parameter** Hides sensitive values in the UI and logs.
```groovy
pipeline {
    agent any
    parameters {
        password(name: 'SECRET_PASSWORD', description: 'Enter your password')
    }
    stages {
        stage('Use Secret') {
            steps {
                echo "Using secret password (hidden in logs)"
            }
        }
    }
}
```

### Where to Use ###

**If you want to skip multiple stages at once, use a String parameter and split it into a list.**
```groovy
pipeline {
    agent any
    parameters {
        string(name: 'SKIP_STAGES', defaultValue: '', description: 'Comma-separated list of stages to skip (e.g., Build,Test)')
    }
    stages {
        stage('Build') {
            when {
                expression { !params.SKIP_STAGES.tokenize(',').contains('Build') }
            }
            steps {
                echo "Building..."
            }
        }
        stage('Test') {
            when {
                expression { !params.SKIP_STAGES.tokenize(',').contains('Test') }
            }
            steps {
                echo "Testing..."
            }
        }
        stage('Deploy') {
            when {
                expression { !params.SKIP_STAGES.tokenize(',').contains('Deploy') }
            }
            steps {
                echo "Deploying..."
            }
        }
    }
}
```
