### Declarative vs Scripted Pipeline ####
**Declarative Pipeline**
- Declarative is the modern, structured way to write pipelines. It has a fixed structure and is easier to read.
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
**Scripted Pipeline**
- Scripted is the older, flexible way. It is pure Groovy code, giving you full programming power.
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

<img width="646" height="313" alt="image" src="https://github.com/user-attachments/assets/ecd18aaf-1e95-4f16-b8a3-f55c58bb1091" />

- Using Groovy Script INSIDE Declarative: You don't have to choose one completely. You can use Groovy logic inside Declarative using the `script {}` block.
```groovy
pipeline {
    agent any

    stages {
        stage('Smart Deploy') {
            steps {
                script {
                    // full Groovy available here
                    def branch = env.BRANCH_NAME
                    def environment = branch == 'main' ? 'prod' : 'dev'

                    if (environment == 'prod') {
                        sh 'kubectl apply -f k8s/prod/'
                    } else {
                        sh 'kubectl apply -f k8s/dev/'
                    }
                }
            }
        }
    }
}
```

---
---

### Pipeline Syntax Deep Dive ###
**agent**
— Where the pipeline runs
```groovy
// Run on any available agent
agent any

// Run on a specific labeled agent
agent { label 'linux-agent' }

// Run inside a Docker container
agent {
    docker {
        image 'maven:3.8-openjdk-17'
        args '-v /root/.m2:/root/.m2'  // mount maven cache
    }
}

// Run as a Kubernetes pod
agent {
    kubernetes {
        yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: maven
                image: maven:3.8-openjdk-17
                command: [sleep, infinity]
              - name: docker
                image: docker:20-dind
                securityContext:
                  privileged: true
        '''
    }
}

// No agent at top level (define per stage)
agent none
```
- `agent none` usefule when:
  - Different stages need different agents: Linux, Windows, Docker
  - Save resources: Agent only allocated when stage runs
  - Flexibility: Each stage picks best agent for its job

---

- `environment` — Define variables
```groovy
pipeline {
    agent any

    environment {
        // static values
        APP_NAME = 'myapp'
        REGION   = 'us-east-1'

        // pulling from Jenkins credentials store
        DOCKER_PASS = credentials('dockerhub-password')

        // dynamic value using Groovy
        BUILD_TAG = "v${BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            environment {
                // stage-level env var (only available in this stage)
                MAVEN_OPTS = '-Xmx1024m'
            }
            steps {
                sh 'echo $APP_NAME'
                sh 'mvn package $MAVEN_OPTS'
            }
        }
    }
}
```
- `stages` and `stage` — Pipeline steps
```groovy
pipeline {
    agent any

    stages {
        // each stage is one logical step in your pipeline
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/myorg/myapp.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
                // publish test results
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        docker build -t myapp:${BUILD_NUMBER} .
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
                        docker push myapp:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }
}
```
- `when` — Conditional stages: Run a stage only when certain conditions are met.
```groovy
stages {
    stage('Deploy to Prod') {
        when {
            // only run when branch is main
            branch 'main'
        }
        steps {
            sh 'kubectl apply -f k8s/prod/'
        }
    }

    stage('Deploy to Dev') {
        when {
            // only run when branch is NOT main
            not { branch 'main' }
        }
        steps {
            sh 'kubectl apply -f k8s/dev/'
        }
    }

    stage('Run Integration Tests') {
        when {
            // multiple conditions — ALL must be true
            allOf {
                branch 'main'
                environment name: 'RUN_INTEGRATION', value: 'true'
            }
        }
        steps {
            sh './run-integration-tests.sh'
        }
    }
}
```
- `post`- What happens after pipeline finishes
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
    }

    post {
        always {
            // runs no matter what — success or failure
            echo 'Pipeline finished'
            cleanWs()  // clean workspace
        }

        success {
            // runs only on success
            slackSend channel: '#deployments',
                      message: "${APP_NAME} deployed successfully"
        }

        failure {
            // runs only on failure
            slackSend channel: '#alerts',
                      message: "${APP_NAME} pipeline failed"
            mail to: 'team@company.com',
                 subject: 'Build Failed',
                 body: "Check Jenkins: ${BUILD_URL}"
        }

        unstable {
            // runs when build is unstable (test failures)
            echo 'Tests failed — build is unstable'
        }

        aborted {
            // runs when pipeline is manually aborted
            echo 'Pipeline was aborted'
        }
    }
}
```

---

**options**
— Pipeline-level settings
```groovy
pipeline {
    agent any

    options {
        // fail if pipeline runs longer than 30 minutes
        timeout(time: 30, unit: 'MINUTES')

        // keep only last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))

        // dont run same pipeline twice at same time
        disableConcurrentBuilds()

        // retry entire pipeline on failure
        retry(2)

        // add timestamps to console output
        timestamps()
    }

    stages { ... }
}
```
- `disableConcurrentBuilds`:
  - Prevents multiple builds of the same job from running simultaneously. When a new build is triggered while another is already running, Jenkins will either queue it or abort it. You can configure the behavior:
  ```groovy
  disableConcurrentBuilds(abortPrevious: true)  // kills the running build
  disableConcurrentBuilds(abortPrevious: false) // queues the new build (default)
  ```
  - When to use it: Deployment pipelines where two deploys at once would conflict, jobs that write to shared resources (databases, files), or integration tests that can't run in parallel.

---

**lock**
- Imagine you have 3 different Jenkins jobs that all deploy to the same staging server
```bash
    Job A  ──deploys to──▶  staging-server  ◀──deploys to──  Job B
                                  ▲
                                  │
                            Job C deploys too
```
- If all 3 run at the same time → chaos. Files overwrite each other, configs clash, tests fail randomly.
- `disableConcurrentBuilds` won't help here — it only blocks the same job from running twice. These are different jobs.
- You give the shared resource a name, and any job that wants to use it must acquire the lock first. Only one job holds it at a time
```groovy
    // Job A
    steps {
        lock('staging-server') {
            sh './deploy.sh'
        }
    }
    
    // Job B
    steps {
        lock('staging-server') {  // same name!
            sh './deploy.sh'
        }
    }
    
    // Job C
    steps {
        lock('staging-server') {  // same name!
            sh './deploy.sh'
        }
    }
```

---

- `parameters`: User inputs before pipeline runs
```groovy
pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG',
               defaultValue: 'latest',
               description: 'Docker image tag to deploy')

        choice(name: 'ENVIRONMENT',
               choices: ['dev', 'staging', 'prod'],
               description: 'Target environment')

        booleanParam(name: 'RUN_TESTS',
                     defaultValue: true,
                     description: 'Run tests before deploy?')
    }

    stages {
        stage('Deploy') {
            steps {
                echo "Deploying ${params.IMAGE_TAG} to ${params.ENVIRONMENT}"

                script {
                    if (params.RUN_TESTS) {
                        sh 'mvn test'
                    }
                }

                sh "kubectl set image deployment/myapp app=myapp:${params.IMAGE_TAG} -n ${params.ENVIRONMENT}"
            }
        }
    }
}
```

**Parallel Stages**
- Run multiple stages at the same time to speed up your pipeline.
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        // these 3 tests run simultaneously
        stage('Run Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test -Dtest=UnitTests'
                    }
                }

                stage('Integration Tests') {
                    steps {
                        sh 'mvn test -Dtest=IntegrationTests'
                    }
                }

                stage('Security Scan') {
                    steps {
                        sh 'trivy image myapp:latest'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}
```
- Without parallel:
  - Unit Tests        → 5 mins
  - Integration Tests → 8 mins
  - Security Scan     → 3 mins
  - Total             → 16 mins 
- With parallel:
  - All three run at same time
  -  Total → 8 mins (longest one)
- Parallel with Different Agents: Each parallel branch can run on a different agent.
```groovy
stage('Test on Multiple Platforms') {
    parallel {
        stage('Test on Linux') {
            agent { label 'linux' }
            steps {
                sh './run-tests.sh'
            }
        }

        stage('Test on Windows') {
            agent { label 'windows' }
            steps {
                bat 'run-tests.bat'
            }
        }

        stage('Test on Mac') {
            agent { label 'mac' }
            steps {
                sh './run-tests.sh'
            }
        }
    }
}
```
- Fail Fast in Parallel: If one parallel branch fails, stop all others immediately.
```groovy
stage('Parallel Tests') {
    failFast true  // stop all branches if one fails

    parallel {
        stage('Unit Tests') {
            steps {
                sh 'mvn test -Dtest=UnitTests'
            }
        }

        stage('Integration Tests') {
            steps {
                sh 'mvn test -Dtest=IntegrationTests'
            }
        }
    }
}
```

**Shared Libraries**
- You have 5 microservices, each with their own Jenkinsfile. They all do the same thing:
```goorvy
// Jenkinsfile in service-A
pipeline {
    stages {
        stage('Build') { steps { sh 'docker build .' } }
        stage('Test')  { steps { sh './run-tests.sh' } }
        stage('Deploy'){ steps { sh './deploy.sh production' } }
    }
    post {
        failure { slackSend message: 'Build failed!' }
    }
}

// Jenkinsfile in service-B  ← exact same copy
// Jenkinsfile in service-C  ← exact same copy
// Jenkinsfile in service-D  ← exact same copy
// Jenkinsfile in service-E  ← exact same copy
```
- Now your team changes the Slack message format. You update 5 files across 5 repos.
- Shared Library solves this — write once, use everywhere
```code
my-shared-library/          ← separate Git repo
├── vars/                   ← functions you call directly in Jenkinsfile
│   ├── buildApp.groovy
│   ├── deployApp.groovy
│   └── notifySlack.groovy
├── src/                    ← Groovy classes (advanced use)
│   └── org/company/
│       └── Utils.groovy
└── resources/              ← static files (shell scripts, templates)
    └── deploy.sh
```

Step 1 — Write the Shared Logic
- vars/buildApp.groovy
```groovy
def call(String imageName) {
    sh "docker build -t ${imageName} ."
    sh "./run-tests.sh"
}
```
- vars/buildApp.groovy
```groovy
def call(String imageName) {
    sh "docker build -t ${imageName} ."
    sh "./run-tests.sh"
}
```
- vars/notifySlack.groovy
```groovy
def call(String status) {
    def color = status == 'success' ? 'good' : 'danger'
    slackSend color: color, message: "Build ${status}!"
}
```

Step 2 — Register the Library in Jenkins
- Go to *Jenkins → Manage Jenkins → Configure System → Global Pipeline Libraries*
```code
Name:           my-shared-library       ← name you'll use in Jenkinsfile
Default Version: main                   ← branch/tag to use
Source:         Git
URL:            https://github.com/your-org/my-shared-library.git
```

Step 3 — Use it in Jenkinsfile
```groovy
@Library('my-shared-library') _       // import the library

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                buildApp('service-a')              // from vars/buildApp.groovy
            }
        }
        stage('Deploy') {
            steps {
                deployApp('service-a', 'production') // from vars/deployApp.groovy
            }
        }
    }
    post {
        success { notifySlack('success') }         // from vars/notifySlack.groovy
        failure { notifySlack('failure') }
    }
}
```
- The `_` After @Library. The `_` is just a dummy variable — it means "import everything from vars/, I don't need a specific object". You'll always write it this way for `vars/` usage.
- Loading Specific Version
```groovy
@Library('my-shared-library@v2.1') _      // specific tag
@Library('my-shared-library@feature-x') _ // specific branch
@Library('my-shared-library@abc1234') _   // specific commit
```
- Multiple Libraries
```groovy
@Library(['my-shared-library', 'another-library']) _
```

**Pipeline Best Practices**
- Always use Declarative over Scripted
- Never hardcode secrets'
- Always clean workspace
- Use timeouts to prevent hanging pipelines
- Use parallel stages for independent tasks
- Use shared libraries — never copy-paste pipelines
- Pin library versions
- Keep Jenkinsfile in the same repo as code
