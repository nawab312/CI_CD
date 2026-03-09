### GitHub, GitLab, Bitbucket Integration ###
SCM means Source Code Management. Jenkins needs to connect to your Git hosting platform to clone code and trigger builds.

**How Jenkins connects to GitHub/GitLab/Bitbucket**
- There are two things you need to set up:
```bash
1. Credentials   → so Jenkins can clone your private repos
2. Webhook/Token → so GitHub/GitLab can trigger Jenkins builds
```

- Step 1 — Store Git Credentials in Jenkins
  - For HTTPS (username + token):
    ```bash
    GitHub → Settings → Developer Settings → Personal Access Token → Generate
                                                        ↓
                                         copy this token
                                                        ↓
    Jenkins → Manage Jenkins → Credentials → Add
      Kind:     Username with Password
      Username: your-github-username
      Password: ghp_yourPersonalAccessToken
      ID:       github-creds
    ```
  - For SSH:
    ```bash
    Generate SSH key pair on your machine:
      ssh-keygen -t rsa -b 4096 -C "jenkins@company.com"
    
    Public key  → paste into GitHub/GitLab → Settings → SSH Keys
    Private key → paste into Jenkins → Credentials → SSH Username with Private Key
    ```
- Step 2 — Use Credentials in Jenkinsfile
  ```groovy
  pipeline {
      agent any
  
      stages {
          stage('Checkout') {
              steps {
                  // HTTPS
                  git branch: 'main',
                      credentialsId: 'github-creds',
                      url: 'https://github.com/myorg/myapp.git'
  
                  // or just use checkout scm
                  // Jenkins auto-uses credentials configured in job
                  checkout scm
              }
          }
      }
  }
  ```

**Platform-Specific Plugins**
- Each platform has a dedicated Jenkins plugin that gives extra features beyond basic Git.
- GitHub Plugin gives you:
  - Build status reported back to GitHub
  - GitHub webhook support
  - PR builds
  - GitHub user authentication for Jenkins login
- GitLab Plugin gives you:
  - Build status reported back to GitLab pipelines
  - Merge request builds
  - GitLab webhook support
  - GitLab OAuth login for Jenkins
 
---
---

### Webhooks vs Polling ###
Both achieve the same goal — trigger a Jenkins build when code changes — but in completely different ways.

**Polling** 
— Jenkins asks GitHub repeatedly
```bash
Jenkins wakes up every 5 minutes
        ↓
Jenkins calls GitHub API: "did anything change?"
        ↓
GitHub: "no"
        ↓
Jenkins sleeps 5 minutes

... 5 minutes later ...

Jenkins wakes up again
        ↓
Jenkins calls GitHub API: "did anything change?"
        ↓
GitHub: "yes, new commit on main"
        ↓
Jenkins triggers build
```
- How to configure polling in Jenkinsfile:
```groovy
pipeline {
    agent any

    triggers {
        // cron syntax: check every 5 minutes
        pollSCM('H/5 * * * *')

        // check every 15 minutes
        pollSCM('H/15 * * * *')

        // check once per hour
        pollSCM('H * * * *')
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
    }
}
```
- Problems with polling:
  - Developer pushes code at 10:00
  - Jenkins polls at 10:04  → no change yet (missed it)
  - Jenkins polls at 10:05  → finds change, triggers build
  - Build starts 5 minutes late
  - Jenkins is making API calls every 5 minutes even when nothing changed — wasted resources
  - 100 repos × poll every 5 mins = 20 API calls per minute. GitHub may *rate-limit* you
- Use Polling when:
  - Jenkins is behind a firewall with no public URL
  - GitHub/GitLab cannot reach your Jenkins
  - On-premise Jenkins with no inbound internet access

---

**Webhooks 
— GitHub tells Jenkins immediately**
```bash
Developer pushes code
        ↓
GitHub immediately sends HTTP POST to Jenkins URL
        ↓
Jenkins receives the event
        ↓
Jenkins triggers build immediately
```
- How to set up a webhook:
```bash
GitHub → Your Repo → Settings → Webhooks → Add Webhook

  Payload URL:    http://your-jenkins.com/github-webhook/
  Content type:   application/json
  Secret:         (optional but recommended)
  Events:         Just the push event
                  or: Pull requests, Pushes (choose what triggers builds)
```
- Jenkinsfile with webhook trigger:
```groovy
pipeline {
    agent any

    triggers {
        // tells Jenkins to accept GitHub webhook events
        githubPush()
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
    }
}
```

---
---

### Multibranch Pipeline ###
A Multibranch Pipeline is a Jenkins job that automatically creates a pipeline for every branch in your repo.

The Problem it Solves
- Without Multibranch:
  - You create Jenkins job for main branch
  - Developer creates feature/login branch
  - Developer creates feature/payment branch
  - Developer creates bugfix/crash branch
  - You have to manually create Jenkins jobs for each branch. Total 4 jobs to manage. Branches come and go — constant manual work
- With Multibranch:
  - One Multibranch Pipeline job
  - Jenkins scans the repo
  - Automatically creates pipeline for every branch
  - Branch deleted? Jenkins removes the job automatically
- How it works
  ```bash
  You create one Multibranch Pipeline job in Jenkins
          ↓
  Jenkins scans your GitHub/GitLab repo
          ↓
  Finds branches: main, develop, feature/login, feature/payment
          ↓
  Automatically creates:
    myapp/main
    myapp/develop
    myapp/feature/login
    myapp/feature/payment
          ↓
  Each branch runs the Jenkinsfile found in that branch
          ↓
  New branch pushed? Jenkins auto-discovers and creates job 
  Branch deleted? Jenkins removes job automatically
  ```

---
---

### Branch Discovery Strategies ###
Branch discovery tells Jenkins which branches to scan and build in a Multibranch pipeline. You don't always want to build every single branch.

**Strategy 1 — Exclude branches that are also filed as PRs**
- Build all branches EXCEPT ones that already have a PR open. Why?
  - If feature/login has a PR open, Jenkins will build the PR itself (with merge preview)
  - No need to also build the branch separately
  - Avoids duplicate builds

**Strategy 2 — Only branches that are filed as PRs**
- Only build a branch when it has an open PR. Why?
  - Developers may create many experimental branches. You only care about branches that are being reviewed
  - Saves CI resources
 
**Strategy 3 — All branches**
- Build every branch regardless of PR status. Why?
  - You want visibility on all work in progress.
  - Common in small teams where every branch matters
 
**Filtering branches by name**
- You can tell Jenkins to only discover branches matching a pattern
- In Multibranch job config → Branch Sources → Property strategy
  ```bash
  Include:  main develop release/* feature/*
  Exclude:  experimental/* wip/*
  ```

---
---

### Pull Request Builds and Status Checks ###
This is one of the most valuable Jenkins features — building and validating PRs before they are merged.

**How PR Builds Work**
```bash
Developer opens Pull Request: feature/login → main
        ↓
GitHub sends webhook to Jenkins (pull_request event)
        ↓
Jenkins creates a special PR build
        ↓
Jenkins checks out a MERGE PREVIEW
(what the code will look like after merge, not just the branch)
        ↓
Runs full pipeline: build, test, security scan
        ↓
Reports result back to GitHub PR "Proceed" or "Abort"
        ↓
GitHub shows status check on the PR
        ↓
Team can enforce: PR cannot be merged unless Jenkins passes
```

**What Developer Sees on GitHub**
```bash
Pull Request: feature/login → main

Checks:
    Jenkins/build       — Build passed
    Jenkins/unit-tests  — All 142 tests passed
    Jenkins/security    — 2 critical vulnerabilities found

    Merging is blocked until all checks pass
```

**Jenkinsfile for PR builds**
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Code Quality') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        sonar-scanner \
                            -Dsonar.login=$SONAR_TOKEN \
                            -Dsonar.pullrequest.key=${CHANGE_ID} \
                            -Dsonar.pullrequest.branch=${CHANGE_BRANCH} \
                            -Dsonar.pullrequest.base=${CHANGE_TARGET}
                    '''
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh 'trivy fs . --exit-code 1 --severity HIGH,CRITICAL'
            }
        }

        // only deploy when merging to main, not on PRs
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'kubectl apply -f k8s/prod/'
            }
        }
    }

    post {
        success {
            // report green check back to GitHub PR
            githubNotify status: 'SUCCESS',
                         description: 'All checks passed'
        }
        failure {
            // report red X back to GitHub PR
            githubNotify status: 'FAILURE',
                         description: 'Build failed — check Jenkins'
        }
    }
}
```

**Important PR Environment Variables**
- Jenkins gives you special variables inside PR builds:
```groovy
env.CHANGE_ID       // PR number         e.g. 42
env.CHANGE_BRANCH   // source branch     e.g. feature/login
env.CHANGE_TARGET   // target branch     e.g. main
env.CHANGE_AUTHOR   // who opened PR     e.g. john
env.CHANGE_TITLE    // PR title

// check if this build is a PR or a branch build
if (env.CHANGE_ID) {
    echo "This is PR #${env.CHANGE_ID}"
} else {
    echo "This is a branch build: ${env.BRANCH_NAME}"
}
```

**Protecting Main Branch with Required Status Checks**
```bash
GitHub → Repo → Settings → Branches → Branch Protection Rules

  Branch name pattern: main

  Require status checks to pass before merging
       Jenkins/build
       Jenkins/unit-tests
       Jenkins/security-scan

  Require branches to be up to date before merging
  Restrict who can push to matching branches
```
- Now nobody can merge a PR unless Jenkins gives a green signal
  





