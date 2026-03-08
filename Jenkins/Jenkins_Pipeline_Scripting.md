**input**
- Pauses the pipeline and waits for a human to approve before continuing.
```groovy
pipeline {
    stages {
        stage('Deploy to Production') {
            steps {
                input 'Are you sure you want to deploy to production?'
            }
        }
    }
}
```
- Pipeline enters a waiting state
- Build executor is released. Means Jenkins Pauses execution, Saves pipeline state, Releases the executor, Waits for approval. So the agent is no longer blocked. That means: The node becomes free. Another job can use that executor
- UI shows “Proceed” / “Abort”

```groovy
input {
    message "Deploy to production?"
    ok "Yes, deploy!"  
    submitter "alice,bob"         
    submitterParameter "APPROVER" 
    parameters {
        choice(name: 'ENV', choices: ['prod', 'staging'], description: 'Target env')
        string(name: 'VERSION', description: 'Version to deploy')
    }
}
```
- `ok` — Custom label for the Proceed button (default is "Proceed")
- `submitter` — Restricts who can approve. Others will see it but can't click.
- `submitterParameter` — Captures the name of whoever approved into a variable. Later use: `echo "Approved by ${APPROVED_BY}"`
- `parameters` — Lets the approver provide input values before proceeding

- Capturing Input Values
```groovy
stage('Approve') {
    steps {
        script {
            def userInput = input(
                message: 'Deploy?',
                parameters: [
                    string(name: 'VERSION', description: 'Which version?'),
                    choice(name: 'ENV', choices: ['prod', 'staging'])
                ]
            )
            echo "Deploying version: ${userInput.VERSION}"
            echo "Target env: ${userInput.ENV}"
        }
    }
}
```

- Timeout — Don't Wait Forever. By default `input` waits indefinitely. Combine it with `timeout` to auto-abort:
```groovy
stage('Approve') {
    options {
        timeout(time: 24, unit: 'HOURS') // auto-abort after 24 hours
    }
    steps {
        input message: 'Deploy to production?', ok: 'Deploy'
    }
}
```

### Who Can Approve ###

**RBAC Control**
```groovy
input message: "Deploy?",
      submitter: "DevOps-Team"
```
- `submitter` restricts who can click “Proceed.”
- `devops-team` is not magically an LDAP group. It must match a user or group recognized by Jenkins Security Realm + Authorization strategy.

**How To Tie Approval to LDAP / AD Groups (Proper Way)**
- Step 1: Configure LDAP/AD in Jenkins
  - Go to: *Manage Jenkins → Security → Security Realm*
  - Select: LDAP or Active Directory. Use the LDAP plugin or Active Directory plugin.
  - This connects Jenkins to: Corporate LDAP, Active Directory
- Step 2: Configure Authorization Strategy
  - *Manage Jenkins → Authorization*
  - Use one of: Matrix-based security or Role-Based Strategy Plugin
  - With Role-Based Strategy Plugin:
    - Create role: `prod-approvers`
    - Assign LDAP group: `CN=DevOps-Team,OU=Groups,DC=company,DC=com`
    - Now Jenkins knows: Anyone in this LDAP group has this role.
- You can also capture who approved using `submitterParameter`
  ```groovy
  stage('Production Approval') {
    steps {
        input message: "Deploy to Production?",
              submitter: "DevOps-Team",
              submitterParameter: "APPROVED_BY"
    }
  }
  ```
  - Then: `echo "Approved by: ${APPROVED_BY}"`
 
### Timeout Risk ###
- What if no one approves?
```groovy
timeout(time: 1, unit: 'HOURS') {
    input message: 'Approve?'
}
```

---

**disableConcurrentBuilds**
- Prevents multiple builds of the same job from running simultaneously.
```groovy
pipeline {
    options {
        disableConcurrentBuilds()
    }
}
```
- When a new build is triggered while another is already running, Jenkins will either queue it or abort it. You can configure the behavior:
```groovy
disableConcurrentBuilds(abortPrevious: true)  // kills the running build
disableConcurrentBuilds(abortPrevious: false) // queues the new build (default)
```
- When to use it: Deployment pipelines where two deploys at once would conflict, jobs that write to shared resources (databases, files), or integration tests that can't run in parallel.

**lock**
- Imagine you have 3 different Jenkins jobs that all deploy to the same staging server
```code
Job A  ──deploys to──▶  staging-server  ◀──deploys to──  Job B
                              ▲
                              │
                        Job C deploys too
```
- If all 3 run at the same time → chaos. Files overwrite each other, configs clash, tests fail randomly.
- `disableConcurrentBuilds` won't help here — it only blocks the *same job* from running twice. These are *different jobs*.
- You give the shared resource a name, and any job that wants to use it must acquire the lock first. Only one job holds it at a time.
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

- What happens at runtime:
```
Job A  →  grabs lock  →  deploys  →  releases lock
Job B  →  WAITS...    →  grabs lock  →  deploys  →  releases lock
Job C  →  WAITS...    →  WAITS...    →  grabs lock  →  deploys
```
- There's no actual resource registered — it's just an agreed-upon name. Any job that uses the same string will compete for the same lock.
```groovy
lock('whatever-you-want')  // the name is up to you
```

**quietPeriod**
- Adds a deliberate delay (in seconds) before a build actually starts after being triggered.
```groovy
pipeline {
    options {
        quietPeriod(30)  // wait 30 seconds before starting
    }
}
```
- During the quiet period, if another trigger fires for the same job, Jenkins resets or merges them — so you end up with one build instead of many rapid-fire ones.
- When to use it: SCM polling jobs where rapid successive commits should be batched into a single build, or webhook-heavy repos where you want to debounce triggers.
