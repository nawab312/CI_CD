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
