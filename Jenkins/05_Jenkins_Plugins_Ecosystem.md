# 🔌 Jenkins Plugins Ecosystem: Complete Interview Mastery Guide
### Category 5 | DevOps/SRE/Platform Engineer Interview Prep

---

> **How to use this guide:**
> Every topic follows the full teaching structure: Simple Definition → Why it exists → How it works → Key concepts → Short interview answer → Deep dive → Real-world example → Interview Q&A → Gotchas → Connections.
>
> ⚠️ = Frequently misunderstood or heavily tested.
>
> Category 5 is where **operational depth** is tested. "Anyone can install a plugin. Can you configure it correctly, explain its internals, troubleshoot it at scale, and know which plugin to choose when?" That's the senior-level bar. JCasC (5.7) and the Kubernetes plugin (5.5) are the two most heavily tested topics here — they reveal whether you've actually run Jenkins in production.

---

# 📑 TABLE OF CONTENTS

1. [Topic 5.1 — Essential Plugins](#topic-51--essential-plugins)
2. [Topic 5.2 — Build Trigger Plugins](#topic-52--build-trigger-plugins)
3. [Topic 5.3 — Notification Plugins](#topic-53--notification-plugins)
4. [Topic 5.4 — Security Plugins — RBAC and Matrix Auth](#topic-54--security-plugins--rbac-and-matrix-auth)
5. [Topic 5.5 — Kubernetes Plugin ⚠️](#topic-55--kubernetes-plugin-)
6. [Topic 5.6 — Docker Plugin vs Docker Pipeline Plugin](#topic-56--docker-plugin-vs-docker-pipeline-plugin)
7. [Topic 5.7 — Configuration as Code (JCasC) Plugin ⚠️](#topic-57--configuration-as-code-jcasc-plugin-)
8. [Topic 5.8 — Job DSL Plugin](#topic-58--job-dsl-plugin)
9. [Category 5 Summary & Self-Quiz](#-category-5-summary--quick-reference)

---
---

# Topic 5.1 — Essential Plugins

## 🟢 Beginner | The Foundation Stack

---

### 📌 What It Is — In Simple Terms

Jenkins' core functionality is intentionally minimal — it's a generic automation framework. Almost everything useful comes from plugins. A fresh Jenkins installation can barely do anything; it's the plugin ecosystem (1,800+ plugins) that makes it a CI/CD platform. The "essential plugins" are the ones every production Jenkins installation has — the foundation layer that everything else builds on.

---

### ⚙️ The Essential Plugin Stack — Detailed Reference

#### Git Plugin
```
Plugin ID: git
What it does:
  - SCM integration: clone, fetch, checkout from Git repositories
  - Supports HTTP(S) and SSH remotes
  - Branch tracking, tag discovery, submodule support
  - Provides GIT_COMMIT, GIT_BRANCH, GIT_URL environment variables
  - Implements CloneOption (shallow clone, depth control)
  - Works with the Pipeline SCM step

Key configuration options:
  shallow clone:  reduce clone time for large repos (no full history)
  sparse checkout: clone only specific directories (monorepo optimization)
  credentials:    HTTP basic auth or SSH key credentials
  refspec:        custom refspec for fetching (e.g., PRs from GitHub)

In Jenkinsfile:
  // Simple:
  git url: 'https://github.com/myorg/myapp.git',
      branch: 'main',
      credentialsId: 'github-creds'

  // Full control with extensions:
  checkout([
      $class: 'GitSCM',
      branches: [[name: env.BRANCH_NAME]],
      extensions: [
          [$class: 'CloneOption',
           shallow: true, depth: 1, noTags: false, timeout: 10],
          [$class: 'CleanBeforeCheckout'],          // git clean -fdx
          [$class: 'PruneStaleBranch'],             // remove stale remote branches
          [$class: 'SparseCheckoutPaths',           // monorepo: only checkout what's needed
           sparseCheckoutPaths: [[path: 'services/auth'], [path: 'shared/']]]
      ],
      userRemoteConfigs: [[
          url:           'https://github.com/myorg/myapp.git',
          credentialsId: 'github-creds',
          refspec:       '+refs/heads/*:refs/remotes/origin/*'
      ]]
  ])
```

#### Pipeline Plugins (the Core Suite)
```
pipeline-model-definition (Declarative Pipeline)
  - Provides the pipeline{} block syntax
  - Stage visualization
  - Pre-execution syntax validation
  - when{}, post{}, options{}, parameters{}, triggers{} directives

workflow-aggregator (Pipeline Suite)
  - Meta-plugin that installs the full Pipeline plugin suite:
    workflow-cps         ← CPS engine, program.dat serialization
    workflow-job         ← Pipeline job type
    workflow-step-api    ← Step API (sh, echo, timeout, etc.)
    workflow-scm-step    ← checkout scm step
    workflow-support     ← Stash, unstash, node allocation
    workflow-durable-task-step ← durable sh/bat that survives agent reconnect

Pipeline: Shared Groovy Libraries
  - @Library annotation support
  - vars/, src/, resources/ structure
  - Library version pinning
  - Global library configuration in Jenkins system settings
```

#### Credentials Plugin
```
Plugin ID: credentials
What it does:
  - Secure credential storage in JENKINS_HOME/credentials.xml (AES-256 encrypted)
  - Credential types: Username+Password, Secret Text, Secret File, SSH Key, Certificate
  - Credential scoping: System, Global, Folder
  - Folder-scoped credentials for team isolation

Credentials Binding Plugin (companion):
  Plugin ID: credentials-binding
  - withCredentials{} step
  - environment{} credentials() function
  - Automatic masking of credential values in logs
  - Unsets credential env vars after withCredentials block

// Usage in pipeline:
withCredentials([
    usernamePassword(credentialsId: 'docker-creds',
                     usernameVariable: 'DOCKER_USER',
                     passwordVariable: 'DOCKER_PASS'),
    string(credentialsId: 'api-token', variable: 'API_TOKEN'),
    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG'),
    sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'SSH_KEY')
]) {
    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
}
```

#### Blue Ocean
```
Plugin ID: blueocean
What it does:
  - Modern pipeline visualization UI (replaces classic Stage View)
  - Visual pipeline editor (for Declarative pipelines)
  - Parallel stage visualization with branch-level status
  - Test results display (integrated with JUnit publisher)
  - PR/branch-aware build history
  - Git activity view showing commit → build mapping

Architecture:
  Blue Ocean is a suite of 30+ individual plugins:
    blueocean-core-js       ← React-based frontend
    blueocean-rest-api      ← REST API backend
    blueocean-pipeline-api  ← Pipeline-specific API
    blueocean-git-pipeline  ← Git integration
    blueocean-github-pipeline ← GitHub-specific features
    blueocean-events        ← Server-Sent Events for real-time updates

Usage note:
  Blue Ocean is display-only — it reads from the same Jenkins data
  Classic Jenkins UI and Blue Ocean show the same builds
  Blue Ocean doesn't change how pipelines execute, only how they're displayed

Status (2024):
  Blue Ocean is in maintenance mode — no new features
  Recommended alternative for visualization: Pipeline: Stage View (lighter)
  Many organizations use Jenkins without Blue Ocean entirely
```

#### Workspace Cleanup Plugin
```
Plugin ID: ws-cleanup
What it does:
  - cleanWs() step: delete workspace contents after build
  - Configurable cleanup conditions: on success, failure, unstable, aborted
  - Can preserve specific files/patterns (e.g., keep test reports)
  - Handles cleanup retry on locked files (Windows)
  - Can run before build (clean before checkout)

// Usage:
post {
    always {
        cleanWs(
            cleanWhenSuccess:  true,   // Clean on success (default)
            cleanWhenFailure:  false,  // KEEP workspace on failure (for debugging)
            cleanWhenAborted:  true,
            cleanWhenUnstable: true,
            notFailBuild:      true,   // Don't fail build if cleanup fails
            patterns: [
                [pattern: 'target/**', type: 'INCLUDE'],   // Delete Maven target/
                [pattern: '**/*.log',  type: 'INCLUDE'],   // Delete log files
                [pattern: '*.xml',     type: 'EXCLUDE']    // Keep XML reports at root
            ]
        )
    }
}

// Pre-build cleanup (via checkout extension):
checkout([
    $class: 'GitSCM',
    extensions: [[$class: 'CleanBeforeCheckout']]  // git clean -fdx before checkout
])
```

#### Build Discarder (Built-in, managed via Pipeline options)
```groovy
// Not a separate plugin — part of Jenkins core
// But critical to configure: without it, JENKINS_HOME grows unboundedly

options {
    buildDiscarder(logRotator(
        numToKeepStr:          '20',   // Keep last 20 builds
        daysToKeepStr:         '30',   // Keep builds for 30 days
        artifactNumToKeepStr:  '5',    // Keep artifacts for last 5 builds
        artifactDaysToKeepStr: '7'     // Keep artifacts for 7 days
    ))
}

// At the system level:
// Manage Jenkins → System → Global Build Discarder
// Sets default for ALL jobs that don't specify their own
```

#### Timestamper
```
Plugin ID: timestamper
What it does:
  - Prepends timestamps to every log line in the build console
  - Helps diagnose which step is slow
  - Adds relative time (time since build start) or absolute time

options {
    timestamps()
}

// Console output becomes:
// [00:00:01] + mvn clean package
// [00:00:45] + docker build -t myapp .
// [00:01:23] + docker push myapp
// Immediately visible: Maven took 44s, Docker build took 38s
```

#### AnsiColor
```
Plugin ID: ansicolor
What it does:
  - Renders ANSI color codes in Jenkins build console
  - Many tools output colorized output (Maven, npm, test frameworks)
  - Without this plugin, color codes appear as garbage characters

options {
    ansiColor('xterm')   // or 'gnome-terminal', 'vga', 'css'
}
// Now Maven test output shows red FAILURE, green SUCCESS in Jenkins console
```

---

### 🔑 Plugin Management — Operations

```bash
# ── INSTALLING PLUGINS ────────────────────────────────────────────
# Method 1: Jenkins UI (Manage Jenkins → Plugin Manager)
# Method 2: Jenkins CLI
java -jar jenkins-cli.jar -s https://jenkins.example.com install-plugin \
    git workflow-aggregator credentials-binding ws-cleanup \
    timestamper ansicolor \
    -restart   # Restart Jenkins after install

# Method 3: Plugin Installation Manager Tool (for Docker/K8s)
# JENKINS_HOME/plugins.txt or plugins.yaml approach
# During Jenkins container startup, plugins.txt is read and plugins installed

# plugins.txt format:
cat > plugins.txt << 'EOF'
git:5.2.2
workflow-aggregator:latest
credentials:1337.v60b_d7b_c7b_9e4
credentials-binding:657.v2b_19db_7d6e6d
ws-cleanup:0.47
blueocean:1.27.10
timestamper:1.26
ansicolor:1.0.4
EOF

# In Dockerfile:
FROM jenkins/jenkins:lts-jdk17
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

# ── PLUGIN VERSION PINNING ────────────────────────────────────────
# ALWAYS pin plugin versions in production
# Floating 'latest' causes unplanned behavior changes during updates

# Check installed plugins and their versions:
java -jar jenkins-cli.jar -s https://jenkins.example.com list-plugins

# ── PLUGIN DEPENDENCY RESOLUTION ─────────────────────────────────
# Every plugin has dependencies on other plugins
# Jenkins automatically installs dependencies
# BUT: dependency conflicts can cause issues
# Use: https://plugins.jenkins.io/  to check compatibility matrix

# ── SAFE PLUGIN UPDATES ───────────────────────────────────────────
# Never update all plugins in one shot in production
# Protocol:
# 1. Update on dev/staging Jenkins first
# 2. Run full build against staging Jenkins
# 3. Verify no regressions
# 4. Update production in a maintenance window
# 5. Have rollback plan: restore plugins/ directory backup
```

---

### 💬 Short Crisp Interview Answer

> *"The essential Jenkins plugin stack has five tiers: SCM (Git plugin for repository integration), Pipeline (workflow-aggregator suite for Declarative and Scripted pipeline execution), Credentials (encrypted secret storage + binding for safe injection), Build Management (Workspace Cleanup, Build Discarder for JENKINS_HOME hygiene), and UX (Timestamper for log timing, AnsiColor for readable console output, Blue Ocean for pipeline visualization). The most critical operational practice: pin plugin versions in production. Floating `latest` versions cause surprise behavior changes on Jenkins restarts. For Kubernetes-based Jenkins, plugins are installed during container image build using a `plugins.txt` file processed by `jenkins-plugin-cli` — this makes the plugin set version-controlled, reproducible, and rollback-able."*

---

### ⚠️ Tricky Edge Cases & Gotchas

- **Blue Ocean is in maintenance mode.** As of 2024, the Blue Ocean project receives only security fixes — no new features. It's still widely installed but the Jenkins project recommends new installations consider Pipeline: Stage View as a lighter alternative. Mentioning this in an interview shows you track the ecosystem.
- **Plugin dependencies are transitive and can cause conflicts.** Installing a plugin silently installs 5-20 dependency plugins. If two plugins require conflicting versions of a shared dependency, one of them fails to load. Check `Manage Jenkins → Manage Plugins → Installed` for plugins with warning icons (load failures).
- **`cleanWs(cleanWhenFailure: false)` is a debugging superpower.** Keeping the workspace on failure lets engineers SSH to agents and inspect the state after a failed build. Balance this against disk usage on agents.
- **Updating the Git plugin can change default checkout behavior.** Major Git plugin updates have changed shallow clone defaults, changed how `GIT_COMMIT` is set, and modified refspec behavior. Always test Git plugin updates against your pipelines first.

---
---

# Topic 5.2 — Build Trigger Plugins

## 🟡 Intermediate | Getting Builds to Start at the Right Time

---

### 📌 What It Is — In Simple Terms

Build trigger plugins connect external events (code pushes, pull requests, scheduled times, external API calls) to Jenkins pipeline execution. They're the "ears" of Jenkins — listening for signals that a build should start. Getting triggers right is critical for CI: too few triggers means builds don't start when they should; too many means wasted resources.

---

### ⚙️ Generic Webhook Trigger Plugin

```
Plugin ID: generic-webhook-trigger

This is the most powerful and flexible trigger plugin.
It can receive webhooks from ANY system that can make HTTP POST requests
and extract any field from the JSON payload as a pipeline variable.

Webhook URL pattern:
  https://jenkins.example.com/generic-webhook-trigger/invoke?token=YOUR-TOKEN

Security:
  The token in the URL authenticates the webhook sender
  Set a random, high-entropy token: uuidgen | tr -d '-'
  Optionally add IP allowlisting at the reverse proxy level
```

```groovy
// Full Generic Webhook Trigger configuration in Jenkinsfile
pipeline {
    agent any

    triggers {
        GenericTrigger(
            // ── EXTRACT VARIABLES FROM PAYLOAD ───────────────────────
            genericVariables: [
                // JSONPath expressions to extract from the webhook body
                [key: 'BRANCH',    value: '$.ref',
                 expressionType: 'JSONPath',
                 defaultValue: '',
                 regexpFilter: '^refs/heads/'],     // Strip 'refs/heads/' prefix
                [key: 'COMMIT_ID', value: '$.after'],
                [key: 'PUSHER',    value: '$.pusher.name'],
                [key: 'REPO_URL',  value: '$.repository.clone_url'],
                [key: 'COMPARE',   value: '$.compare'],
                [key: 'COMMIT_MSG',value: '$.head_commit.message'],
            ],

            // ── EXTRACT FROM HTTP HEADERS ─────────────────────────────
            genericHeaderVariables: [
                [key: 'X_GITHUB_EVENT', value: 'X-GitHub-Event'],
                [key: 'X_GITHUB_DELIVERY', value: 'X-GitHub-Delivery']
            ],

            // ── AUTHENTICATION TOKEN ──────────────────────────────────
            token:              'my-super-secret-random-token-abc123xyz',

            // ── FILTER: Only trigger on specific conditions ────────────
            // Optional: only fire if this text matches this regex
            regexpFilterText:  '$BRANCH',
            regexpFilterExpression: '^(main|develop|release/.+)$',
            // ↑ Only trigger for main, develop, or release/* branches

            // ── CAUSE STRING (shown in build description) ─────────────
            causeString:    'Triggered by push to $BRANCH by $PUSHER',

            // ── LOGGING ───────────────────────────────────────────────
            printContributedVariables: true,   // Log extracted variables
            printPostContent:          false,  // Don't log payload (may contain secrets)

            // ── SILENCE: Don't trigger if already running ─────────────
            // silentResponse: false,  // Set to true to suppress HTTP response
        )
    }

    stages {
        stage('Build') {
            steps {
                echo "Branch: ${env.BRANCH}"
                echo "Commit: ${env.COMMIT_ID}"
                echo "Pusher: ${env.PUSHER}"
                // The extracted variables are available as env vars
                sh "git checkout ${env.COMMIT_ID}"
                sh 'mvn clean package'
            }
        }
    }
}
```

```bash
# Testing your webhook from the command line:
curl -X POST \
  "https://jenkins.example.com/generic-webhook-trigger/invoke?token=my-super-secret-random-token-abc123xyz" \
  -H "Content-Type: application/json" \
  -H "X-GitHub-Event: push" \
  -d '{
    "ref": "refs/heads/main",
    "after": "abc123def456",
    "pusher": {"name": "alice"},
    "repository": {"clone_url": "https://github.com/myorg/myapp.git"},
    "head_commit": {"message": "Fix payment service bug"},
    "compare": "https://github.com/myorg/myapp/compare/old...new"
  }'

# Response includes: {"jobs": {"myapp-pipeline": {"triggered": true, "id": 42}}}
```

---

### ⚙️ GitHub Plugin

```
Plugin ID: github
What it adds over Generic Webhook:
  - github-specific webhook verification (HMAC signature check)
  - githubPush() trigger in Declarative Pipeline
  - Commit status reporting (green/red dot on commits)
  - GitHub PR build triggering
  - GitHub Organization Folder integration

Commit status reporting:
  After each build, the GitHub plugin automatically posts build status
  to the commit's GitHub page (the green ✓ or red ✗ next to commits)
  GitHub users see this directly in PR pages and commit history.

Configuration:
  1. Manage Jenkins → Configure System → GitHub
     → GitHub Server: api.github.com
     → Credentials: GitHub PAT with 'repo:status' scope
     → Test Connection
  
  2. On GitHub repo:
     Settings → Webhooks → Add webhook
     Payload URL: https://jenkins.example.com/github-webhook/
     Content type: application/json
     Secret: (optional but recommended)
     Events: Push + Pull requests
```

```groovy
// GitHub plugin trigger in pipeline:
triggers {
    githubPush()    // Fires on every push to the repo
    // Note: For Multi-Branch Pipelines, webhook configuration
    // is handled at the job level, not in Jenkinsfile
}

// Posting custom status to GitHub from pipeline:
// (via GitHub plugin's step)
githubNotify(
    context:     'Jenkins CI / Unit Tests',
    description: 'Tests passed',
    status:      'SUCCESS',     // SUCCESS, FAILURE, PENDING, ERROR
    targetUrl:   "${BUILD_URL}testReport/"
)

// Or: using the Checks API (more feature-rich):
publishChecks(
    name:    'Unit Tests',
    title:   '142 passed, 0 failed',
    summary: "All tests passed in ${currentBuild.durationString}",
    status:  'COMPLETED',
    conclusion: 'SUCCESS'
)
```

---

### ⚙️ GitLab Plugin

```
Plugin ID: gitlab-plugin
What it does:
  - Accepts GitLab webhooks (Push, MR, Tag events)
  - Reports build status back to GitLab (pipeline/merge request status)
  - Supports GitLab Community and Enterprise editions
  - Auto-cancels outdated MR builds when new commits are pushed

Configuration:
  1. Manage Jenkins → Configure System → GitLab
     → GitLab connection name
     → GitLab URL: https://gitlab.example.com
     → Credentials: GitLab API token (with api scope)
     → Test Connection

  2. On GitLab project:
     Settings → Webhooks → Add webhook
     URL: https://jenkins.example.com/project/my-pipeline
     Secret Token: (from Jenkins GitLab plugin settings)
     Triggers: Push events + Merge request events + Tag push events
```

```groovy
// GitLab trigger in pipeline:
triggers {
    gitlab(
        triggerOnPush:             true,
        triggerOnMergeRequest:     true,
        triggerOnNoteRequest:      false,   // Trigger on MR comment (e.g., /test)
        noteRegex:                 '.*test.*',
        skipWorkInProgressMergeRequest: true,  // Skip WIP/Draft MRs
        triggerOnAcceptedMergeRequest: false,
        ciSkip:                    true,    // Skip if commit message contains [ci skip]
        setBuildDescription:       true,
        branchFilterType:          'All',   // All, NameBasedFilter, RegexBasedFilter
        includeBranchesSpec:       'main develop release/*',
        secretToken:               '${env.GITLAB_WEBHOOK_TOKEN}',
        addNoteOnMergeRequest:     true,    // Post build result as MR comment
        addCiMessage:              true
    )
}
```

---

### ⚙️ Parameterized Trigger Plugin

```
Plugin ID: parameterized-trigger
What it does:
  - Trigger downstream Jenkins jobs with parameters
  - More flexible than the built-in build{} step for complex scenarios
  - Can pass current build parameters + additional ones to downstream

// In pipeline — prefer the built-in build{} step:
stage('Trigger Integration Tests') {
    steps {
        build(
            job:    'integration-test-pipeline',
            wait:   true,
            parameters: [
                string(name: 'IMAGE_TAG',  value: env.IMAGE_TAG),
                string(name: 'TARGET_ENV', value: 'staging'),
                booleanParam(name: 'FULL_SUITE', value: true)
            ]
        )
    }
}

// Parameterized Trigger plugin adds:
// - Trigger multiple downstream jobs in parallel
// - Complex parameter mapping (copy all current params to downstream)
// - Conditional triggering based on build result
```

---

### ⚙️ Build Authorization Token Root Plugin

```
Plugin ID: build-token-root
What it does:
  - Allows triggering builds via a URL with an authentication token
  - Simpler than Generic Webhook Trigger for basic "just start a build" use cases
  - Used for simple HTTP-based triggers from external scripts

URL: https://jenkins.example.com/buildByToken/build?job=myapp&token=MY-TOKEN

// For parameterized builds:
// https://jenkins.example.com/buildByToken/buildWithParameters?job=myapp&token=MY-TOKEN&PARAM=value

// Configure in job settings: "Trigger builds remotely" → enter token
```

---

### 🔑 Comparison: Which Trigger Plugin to Use

| Use Case | Plugin | Why |
|----------|--------|-----|
| GitHub.com push webhook | GitHub plugin | Native GitHub integration, HMAC verification |
| GitLab push/MR webhook | GitLab plugin | Native MR status reporting |
| Any other webhook source | Generic Webhook Trigger | Flexible JSONPath extraction, any HTTP source |
| Cron schedule | Built-in `cron()` trigger | No plugin needed |
| Upstream job completion | Built-in `upstream()` trigger | No plugin needed |
| External script trigger | Build Token Root | Simple URL-based trigger |
| Jira / Confluence trigger | Generic Webhook Trigger | JSONPath extracts Jira event data |
| ArgoCD trigger | Generic Webhook Trigger | Extract sync status from Argo webhook |

---

### 💬 Short Crisp Interview Answer

> *"The main build trigger plugins are: GitHub plugin for GitHub.com push events with HMAC signature verification and automatic commit status reporting; GitLab plugin for GitLab webhook events with native MR status reporting; and Generic Webhook Trigger for anything else — it accepts a POST from any system and uses JSONPath expressions to extract variables from the payload. Generic Webhook Trigger is the most flexible because it works with any JSON-speaking system: GitHub, GitLab, Jira, PagerDuty, Argo CD, custom scripts. You configure a secret token in the URL for authentication, define JSONPath expressions to extract fields as pipeline variables, and optionally add regex filters to only trigger on specific conditions. The key production requirement: always use webhook-based triggers over SCM polling. Webhooks are push-based — builds start immediately on code push. Polling is pull-based — the Jenkins Controller makes outbound requests on a schedule, creating unnecessary load and latency."*

---

### ⚠️ Tricky Edge Cases & Gotchas

- **⚠️ Generic Webhook Trigger token in URL vs body.** The token should be in the URL query parameter, not in the webhook payload body. GitHub lets you configure a webhook secret separately — use that for HMAC signature verification (GitHub plugin). Generic Webhook Trigger uses its own token system.
- **`githubPush()` doesn't work in Multibranch Pipeline Jenkinsfile.** In a Multibranch Pipeline, webhook handling is at the Multi-Branch job level, not in individual branch Jenkinsfiles. Adding `triggers { githubPush() }` to a Multibranch Jenkinsfile has no effect.
- **GitLab plugin and webhook secret token confusion.** The GitLab plugin's webhook secret token is configured in Manage Jenkins → Configure System → GitLab connection settings, AND in the GitLab webhook settings. Both must match. A mismatch means webhooks are silently ignored.
- **Generic Webhook `regexpFilterText` can reference variables.** The `$BRANCH` syntax in `regexpFilterText: '$BRANCH'` references the extracted variable named `BRANCH`. This is evaluated after variable extraction, so you can filter based on extracted values.

---
---

# Topic 5.3 — Notification Plugins

## 🟡 Intermediate | Closing the Feedback Loop

---

### 📌 What It Is — In Simple Terms

Notification plugins send build results to people and systems — Slack messages, emails, PagerDuty incidents, GitHub PR comments. They close the feedback loop between CI/CD pipeline execution and the humans responsible for the code. Well-configured notifications mean developers learn about failures within seconds; poorly configured ones create notification fatigue that gets ignored.

---

### ⚙️ Slack Notification Plugin

```
Plugin ID: slack
What it does:
  - Posts build messages to Slack channels
  - Supports message threading, reactions, file uploads
  - Color-coded messages (good=green, warning=yellow, danger=red)
  - Block Kit support (rich formatted messages with buttons/sections)
  - Returns a SlackSendResponse for message updates (start/end patterns)
```

```groovy
// ── SETUP ─────────────────────────────────────────────────────────
// 1. Create Slack App or Incoming Webhook at api.slack.com
// 2. Manage Jenkins → Configure System → Slack:
//    Workspace:    mycompany
//    Credential:   slackToken (Secret Text with Bot Token)
//    Default channel: #build-notifications

// ── BASIC USAGE ────────────────────────────────────────────────────
post {
    success {
        slackSend(
            channel: '#deployments',
            color:   'good',       // good=green, warning=yellow, danger=red, #HEX
            message: "✅ *${env.JOB_NAME}* #${env.BUILD_NUMBER} succeeded\n${env.BUILD_URL}"
        )
    }
    failure {
        slackSend(
            channel: '#build-failures',
            color:   'danger',
            message: "❌ *${env.JOB_NAME}* #${env.BUILD_NUMBER} FAILED\n${env.BUILD_URL}"
        )
    }
}

// ── ADVANCED: MESSAGE WITH THREAD (start/update/finish pattern) ────
stage('Deploy') {
    steps {
        script {
            // Post initial message and capture the response
            def msg = slackSend(
                channel:   '#deployments',
                color:     '#439FE0',
                message:   "🚀 Deploying *${env.APP_NAME}* `${env.IMAGE_TAG}` to production..."
            )

            try {
                sh 'helm upgrade --install myapp ./chart --namespace production --wait'

                // Update the same message thread on success
                slackSend(
                    channel:    msg.channelId,
                    timestamp:  msg.ts,   // Thread reply
                    color:      'good',
                    message:    "✅ Deploy complete — <https://prod.example.com|View Service>"
                )
            } catch (Exception e) {
                slackSend(
                    channel:    msg.channelId,
                    timestamp:  msg.ts,
                    color:      'danger',
                    message:    "❌ Deploy FAILED: ${e.message}"
                )
                throw e
            }
        }
    }
}

// ── RICH BLOCK KIT MESSAGE ─────────────────────────────────────────
slackSend(
    channel: '#deployments',
    blocks: [
        [
            "type": "header",
            "text": ["type": "plain_text", "text": "Production Deployment"]
        ],
        [
            "type": "section",
            "fields": [
                ["type": "mrkdwn", "text": "*Service:*\n${env.APP_NAME}"],
                ["type": "mrkdwn", "text": "*Version:*\n`${env.IMAGE_TAG}`"],
                ["type": "mrkdwn", "text": "*Status:*\n✅ Deployed"],
                ["type": "mrkdwn", "text": "*Duration:*\n${currentBuild.durationString}"]
            ]
        ],
        [
            "type": "actions",
            "elements": [
                [
                    "type": "button",
                    "text": ["type": "plain_text", "text": "View Build"],
                    "url": env.BUILD_URL
                ],
                [
                    "type": "button",
                    "text": ["type": "plain_text", "text": "View Service"],
                    "url": "https://prod.example.com"
                ]
            ]
        ]
    ]
)
```

---

### ⚙️ Email-ext Plugin (Extended Email Notification)

```
Plugin ID: email-ext
Why email-ext over built-in Mailer:
  - Configurable per-build triggers (vs Mailer: always on failure/unstable)
  - Groovy-templated email bodies (HTML or plain text)
  - Dynamic recipient lists (committers, build requestor, specific users)
  - Per-project SMTP configuration override
  - Attachment support (test reports, logs)
  - Throttling: don't send more than 1 email per N minutes
```

```groovy
// ── BASIC EMAIL NOTIFICATION ────────────────────────────────────────
post {
    failure {
        emailext(
            subject: "BUILD FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                <html>
                <body>
                <h2>Build Failed</h2>
                <p><b>Job:</b> ${env.JOB_NAME}</p>
                <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
                <p><b>Branch:</b> ${env.BRANCH_NAME}</p>
                <p><b>Git Commit:</b> ${env.GIT_COMMIT}</p>
                <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                <p>See the <a href="${env.BUILD_URL}console">console output</a> for details.</p>
                </body>
                </html>
            """,
            mimeType:   'text/html',
            to:         "\${DEFAULT_RECIPIENTS}",
            // Dynamic recipient providers:
            recipientProviders: [
                culprits(),      // People who committed since last successful build
                requestor(),     // Person who triggered the build
                developers(),    // All developers who changed code in this build
                upstreamDevelopers()
            ],
            attachLog:         false,    // Don't attach full console log
            compressLog:       false,
            attachmentsPattern: 'target/surefire-reports/**/*.xml'
        )
    }

    // Only send "fixed" notification when build goes back to green:
    fixed {
        emailext(
            subject: "✅ Build Fixed: ${env.JOB_NAME}",
            body:    "Build is back to green. Build URL: ${env.BUILD_URL}",
            to:      "\${DEFAULT_RECIPIENTS}"
        )
    }
}

// ── GROOVY TEMPLATE FOR RICH EMAIL ────────────────────────────────
// Create template file at: JENKINS_HOME/email-templates/my-template.groovy
emailext(
    subject:    "Build \${build.result}: \${project.name} #\${build.number}",
    body:       '\${SCRIPT, template="my-template.groovy"}',
    to:         "\${DEFAULT_RECIPIENTS}",
    recipientProviders: [culprits(), requestor()]
)
```

---

### ⚙️ PagerDuty Plugin

```
Plugin ID: pagerduty
What it does:
  - Create/resolve PagerDuty incidents from Jenkins build results
  - Trigger incident when production build fails
  - Resolve incident automatically when build recovers
  - Supports PagerDuty Events API v2
```

```groovy
// ── PAGERDUTY INCIDENT ON PRODUCTION FAILURE ───────────────────────
post {
    failure {
        script {
            // Only page on-call for main branch production failures
            if (env.BRANCH_NAME == 'main') {
                pagerduty(
                    resolve:        false,     // false = create incident, true = resolve
                    serviceKey:     env.PAGERDUTY_SERVICE_KEY,
                    incDescription: "Production Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    incDetails:     "Build URL: ${env.BUILD_URL}\nGit commit: ${env.GIT_COMMIT}\nFailed at: ${currentBuild.description ?: 'unknown stage'}",
                    numBuilds:      '1'        // Number of consecutive failures before paging
                )
            }
        }
    }

    // Auto-resolve the incident when build recovers:
    fixed {
        script {
            if (env.BRANCH_NAME == 'main') {
                pagerduty(
                    resolve:    true,   // Resolve the open incident
                    serviceKey: env.PAGERDUTY_SERVICE_KEY,
                    incDescription: "Build Recovered: ${env.JOB_NAME}"
                )
            }
        }
    }
}

// ── DIRECT PAGERDUTY API CALL (without plugin) ─────────────────────
// More control, no plugin dependency:
def triggerPagerDuty(String summary, String severity = 'critical') {
    withCredentials([string(credentialsId: 'pagerduty-integration-key', variable: 'PD_KEY')]) {
        sh """
            curl -X POST https://events.pagerduty.com/v2/enqueue \
                -H 'Content-Type: application/json' \
                -d '{
                    "routing_key": "\$PD_KEY",
                    "event_action": "trigger",
                    "payload": {
                        "summary":   "${summary}",
                        "severity":  "${severity}",
                        "source":    "jenkins",
                        "custom_details": {
                            "job":         "${env.JOB_NAME}",
                            "build_number":"${env.BUILD_NUMBER}",
                            "build_url":   "${env.BUILD_URL}"
                        }
                    }
                }'
        """
    }
}
```

---

### ⚙️ Notification Strategy — Production Patterns

```groovy
// ── ANTI-PATTERNS TO AVOID ────────────────────────────────────────
// ❌ Notify on every build → notification fatigue → notifications ignored
post {
    always {
        slackSend channel: '#builds', message: "Build completed"  // NOISE
    }
}

// ❌ Notify success for ALL branches → constant green spam
post {
    success {
        slackSend channel: '#builds', message: "Build passed"  // NOISE
    }
}

// ── PRODUCTION NOTIFICATION STRATEGY ──────────────────────────────
post {
    // Only notify on FAILURE — always worth knowing
    failure {
        slackSend channel: '#build-failures', color: 'danger',
                  message: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}\n${env.BUILD_URL}"

        // Email the culprits (people who committed code since last success)
        emailext(
            subject: "Build Failed: ${env.JOB_NAME}",
            body: "See ${env.BUILD_URL}",
            recipientProviders: [culprits()]
        )
    }

    // Only notify when status CHANGES (not every green build)
    fixed {
        slackSend channel: '#build-failures', color: 'good',
                  message: "✅ FIXED: ${env.JOB_NAME} — back to green"
    }

    // Regression (success → failure) — extra visibility
    regression {
        slackSend channel: '#build-failures', color: 'danger',
                  message: "📉 REGRESSED: ${env.JOB_NAME} — was passing"

        // Only page on-call for main branch regressions
        script {
            if (env.BRANCH_NAME == 'main') {
                pagerduty(resolve: false, serviceKey: env.PD_KEY,
                          incDescription: "Production build broken: ${env.JOB_NAME}")
            }
        }
    }

    // Production deploys worth notifying (but not feature branch deploys)
    success {
        script {
            if (env.BRANCH_NAME == 'main') {
                slackSend channel: '#deployments', color: 'good',
                          message: "🚀 Deployed: ${env.APP_NAME} `${env.IMAGE_TAG}` to production"
            }
        }
    }
}
```

---

### 🔑 Key Concepts

| Concept | Meaning |
|---------|---------|
| `culprits()` | Email recipients: everyone who committed since last successful build |
| `requestor()` | The person who manually triggered the build |
| `developers()` | All developers who changed code in this build |
| `fixed` condition | Build went from failure/unstable to success |
| `regression` condition | Build went from success to failure |
| `changed` condition | Build result different from previous build (either direction) |
| SlackSendResponse | Return value from `slackSend()` — use `.ts` and `.channelId` for threading |
| Block Kit | Slack's rich message format with buttons, sections, headers |
| PagerDuty `resolve: true` | Auto-resolve the incident when build recovers |
| Notification fatigue | Too many notifications → ignored notifications → missed real failures |

---

### 💬 Short Crisp Interview Answer

> *"Notification plugins close the feedback loop between pipeline execution and engineers. The three main ones: Slack for team notifications (color-coded messages, rich Block Kit formatting, message threading for start/update/complete patterns), email-ext for email notifications with dynamic recipients via `culprits()` and `requestor()` providers, and PagerDuty for incident creation on critical failures with auto-resolve on recovery. The most important production discipline: avoid notification fatigue. Don't notify on every build — notify on `failure`, on `fixed` (build went green again), and on `regression` (main branch just broke). Reserve PagerDuty for production main-branch failures only. Using `culprits()` as email recipients sends failure emails to exactly the people who committed code since the last green build — the most likely authors of the regression — without spamming the whole team."*

---

### ⚠️ Tricky Edge Cases & Gotchas

- **`slackSend` returns `null` if not configured.** If the Slack plugin credentials aren't configured in Jenkins system settings, `slackSend()` returns `null` and calling `.ts` on it throws NPE. Always check your Slack integration works before using the threading pattern.
- **`culprits()` can be empty on the first build.** If there's no previous successful build to compare against, `culprits()` returns an empty list. This can result in no email recipients for the first failure. Add a fallback: `recipientProviders: [culprits(), requestor()]`.
- **PagerDuty `numBuilds` prevents false alerting.** Setting `numBuilds: '3'` means the incident is only created if the build has failed 3 consecutive times — reducing false pages from transient failures. Balance this against alert latency.
- **Email-ext `\${DEFAULT_RECIPIENTS}` uses backslash-dollar.** In Groovy GStrings, `${VAR}` is Groovy interpolation. To use email-ext's own template variables (evaluated by email-ext, not Groovy), escape with `\${VAR}`. This is a very common source of "notifications going to wrong people" bugs.

---
---

# Topic 5.4 — Security Plugins — RBAC and Matrix Auth

## 🟡 Intermediate | Who Can Do What

---

### 📌 What It Is — In Simple Terms

Security plugins control **authorization** in Jenkins — who can view jobs, who can trigger builds, who can configure pipelines, and who can administer the system. Jenkins ships with basic authorization options, but production organizations need role-based access control (RBAC) that maps to their team structure and satisfies compliance requirements.

---

### 🔍 Jenkins Authorization Models — The Options

```
1. Anyone can do anything  — Development/testing ONLY. Never production.
2. Legacy mode             — Admin = all, non-admin = read-only.
3. Logged-in users can do anything — Still too permissive for production.
4. Matrix-based security   — Granular per-user/group per-permission matrix.
5. Project-based Matrix    — Per-job access control (extension of matrix).
6. Role-based strategy     — Plugin: Role-Based Authorization Strategy
                             Most common for teams. Roles → permissions, users → roles.
```

---

### ⚙️ Matrix-Based Security (Built-in)

```
Plugin: matrix-auth (Matrix Authorization Strategy Plugin)
Plugin ID: matrix-auth

Creates a permission matrix:
  Rows: users and groups
  Columns: permissions (Overall/Read, Job/Build, Job/Configure, etc.)

Permission categories:
  Overall:    Administer, ConfigureUpdateCenter, Read, RunScripts, UploadPlugins
  Credentials: Create, Delete, ManageDomains, Update, View
  Agent:      Build, Configure, Connect, Create, Delete, Disconnect
  Job:        Build, Cancel, Configure, Create, Delete, Discover, Move, Read, Workspace
  Run:        Delete, Replay, Update
  View:       Configure, Create, Delete, Read
  SCM:        Tag
```

```yaml
# JCasC — Matrix-based security configuration
jenkins:
  authorizationStrategy:
    globalMatrix:
      permissions:
        # Format: "Permission/Category:user_or_group"
        - "Overall/Administer:jenkins-admins"    # LDAP group
        - "Overall/Read:authenticated"           # All logged-in users
        - "Job/Read:authenticated"               # All users can see jobs
        - "Job/Build:developers"                 # Developer group can trigger
        - "Job/Configure:team-leads"             # Team leads can configure jobs
        - "Job/Cancel:developers"
        - "Credentials/View:developers"
        - "Credentials/Create:jenkins-admins"
        - "Agent/Build:developers"               # Developers can run builds on agents
        - "View/Read:authenticated"

  securityRealm:
    ldap:
      configurations:
        - server: "ldap://ldap.example.com:389"
          rootDN: "dc=example,dc=com"
          userSearchBase: "ou=users"
          userSearch: "uid={0}"
          groupSearchBase: "ou=groups"
          groupMembershipStrategy:
            fromGroupSearch:
              filter: "member={0}"
          managerDN: "cn=jenkins,ou=service-accounts,dc=example,dc=com"
          managerPasswordSecret: "${LDAP_MANAGER_PASSWORD}"
```

---

### ⚙️ Role-Based Authorization Strategy (RBAC)

```
Plugin ID: role-strategy
What it adds over Matrix Auth:
  - Named roles with permission sets
  - Assign multiple users/groups to a role
  - Global roles (apply to whole Jenkins)
  - Item/Project roles (apply to specific jobs matching a regex pattern)
  - Agent roles (apply to specific agents)
  - Role assignment via LDAP groups
  - Easy to understand: "alice is a Developer, bob is a Read-Only"
```

```yaml
# JCasC — Role-Based Authorization Strategy
jenkins:
  authorizationStrategy:
    roleBased:
      roles:
        # ── GLOBAL ROLES ────────────────────────────────────────────
        global:
          - name: "admin"
            description: "Jenkins system administrators"
            permissions:
              - "Overall/Administer"
            assignments:
              - "jenkins-admins"       # LDAP group

          - name: "developer"
            description: "Software developers — can build, view, cancel"
            permissions:
              - "Overall/Read"
              - "Job/Build"
              - "Job/Cancel"
              - "Job/Read"
              - "Job/Workspace"
              - "View/Read"
              - "Agent/Build"
              - "Run/Replay"           # Can replay builds
            assignments:
              - "developers"           # LDAP group
              - "alice"                # Individual user

          - name: "viewer"
            description: "Read-only access (auditors, external stakeholders)"
            permissions:
              - "Overall/Read"
              - "Job/Read"
              - "View/Read"
            assignments:
              - "auditors"
              - "product-team"

        # ── ITEM ROLES (per-project access control) ──────────────────
        # Regex matches against job names
        items:
          - name: "team-alpha-developer"
            description: "Full access to Team Alpha's jobs"
            pattern: "team-alpha/.*"    # Regex: matches all jobs in team-alpha folder
            permissions:
              - "Job/Build"
              - "Job/Cancel"
              - "Job/Configure"          # Team Alpha can configure their own jobs
              - "Job/Read"
              - "Job/Create"
              - "Job/Delete"
              - "Run/Replay"
              - "Credentials/View"
            assignments:
              - "team-alpha-members"

          - name: "team-beta-developer"
            description: "Full access to Team Beta's jobs"
            pattern: "team-beta/.*"
            permissions:
              - "Job/Build"
              - "Job/Cancel"
              - "Job/Configure"
              - "Job/Read"
              - "Run/Replay"
            assignments:
              - "team-beta-members"

          - name: "production-deployer"
            description: "Can trigger production deployments (restricted role)"
            pattern: ".*/deploy-production"   # Only production deploy jobs
            permissions:
              - "Job/Build"
              - "Job/Read"
            assignments:
              - "release-managers"
              - "sre-team"

        # ── AGENT ROLES ───────────────────────────────────────────────
        agents:
          - name: "agent-user"
            description: "Can use agents for builds"
            pattern: ".*"
            permissions:
              - "Agent/Build"
            assignments:
              - "developers"
```

---

### ⚙️ Folder-Based Access Control

```
Jenkins Folders (cloudbees-folder plugin) + Role Strategy = Team Isolation

Structure:
  /team-alpha/           ← Folder (Team Alpha owns this)
    myapp-pipeline
    payments-pipeline
  /team-beta/            ← Folder (Team Beta owns this)
    auth-pipeline
  /shared/               ← Shared pipelines (viewable by all)
    integration-tests

With item role pattern "team-alpha/.*":
  - team-alpha members can: build, configure, create, delete jobs in /team-alpha/
  - team-alpha members CANNOT: see /team-beta/ jobs (unless they have a 'viewer' global role)

This achieves:
  ✅ Team autonomy: each team configures their own pipelines
  ✅ Isolation: teams can't accidentally (or intentionally) break each other's pipelines
  ✅ Compliance: access to production deployment jobs restricted to release managers
```

---

### ⚙️ Audit Trail Plugin

```
Plugin ID: audit-trail
What it does:
  - Logs all user actions to an audit log
  - Records: who did what, when, what job, what result
  - Essential for SOC2, ISO 27001, FedRAMP compliance
  - Logs to file (rotating) or syslog

// Actions logged:
// - Login/logout
// - Job configuration changes (who changed what parameter)
// - Build triggers (who started what build)
// - Credential access (which credential was used by which job)
// - Plugin installations/updates
// - User management changes

// JCasC configuration:
unclassified:
  auditTrailPlugin:
    logBuildCause: true
    pattern: ".*"      # Log all events
    loggers:
      - file:
          count: 10
          limit: 50    # MB per file
          log: "/var/log/jenkins/audit.log"
```

---

### ⚙️ Credentials Plugin Scoping — Security Boundary

```
Credentials scoping prevents credential theft between teams:

GLOBAL scope:
  Credential accessible to ALL jobs on Jenkins
  ❌ Bad for production credentials: any job can use production AWS keys

SYSTEM scope:
  Only Jenkins infrastructure can use (agent connections, email server)
  Not accessible from pipeline Groovy code at all

FOLDER scope:
  Only jobs inside that specific folder can access
  ✅ Best for team credentials: team-alpha can only use their own credentials

Configuration:
  When creating a credential:
    Scope: Folder (select the folder this team owns)
  
  In pipeline: reference credential by ID as normal
    withCredentials([...credentialsId: 'team-alpha-db-password'...])
    
  Jobs in other folders: get "No such credentials found" error
  ✅ Team Beta cannot access Team Alpha's database password even if they know the ID
```

---

### 🔑 Key Concepts

| Concept | Meaning |
|---------|---------|
| **Authentication** | Verifying WHO you are (LDAP, GitHub OAuth, SAML) |
| **Authorization** | Controlling WHAT you can do (Matrix Auth, Role Strategy) |
| **Matrix Auth** | Per-user/group, per-permission table — fine-grained but complex |
| **Role Strategy** | Named roles with permissions → users assigned to roles — easier to manage |
| **Item roles** | Role Strategy roles scoped to specific jobs matching a regex |
| **Folder isolation** | Teams only see/access their own folder's jobs |
| **Credential scoping** | Folder-scoped credentials prevent cross-team credential access |
| **Audit trail** | Log of all user actions — required for compliance |
| **Least privilege** | Give each user/group only the permissions they need |
| **`Overall/Administer`** | Full Jenkins control — give to very few accounts |

---

### 💬 Short Crisp Interview Answer

> *"Jenkins has two main authorization plugins: Matrix Authorization Strategy provides a flat permission matrix — each user/group gets specific permissions globally. Role-Based Authorization Strategy (Role Strategy plugin) is more scalable — you define named roles with permission sets, then assign users/groups to roles. Role Strategy also supports item roles with regex patterns for per-job access control. My production setup: a 'developer' global role with build/cancel/read permissions, an 'admin' global role for Jenkins admins, and item roles like `team-alpha/.*` that give Team Alpha full Configure+Create+Delete on their folder's jobs while preventing access to other teams' jobs. Combine with folder-scoped credentials to complete the isolation — Team Alpha can't reference Team Beta's production credentials even if they know the ID. The audit trail plugin is mandatory for compliance — it logs every user action, build trigger, and credential access to a file that security can review."*

---

### ⚠️ Tricky Edge Cases & Gotchas

- **⚠️ `Overall/Read` is required for basically everything.** Even just to see that a job exists, users need `Overall/Read`. Forgetting to grant this to a group results in them getting a blank Jenkins page with no error — just empty. It looks like Jenkins isn't working rather than an access denied error.
- **Item role patterns are full Java regexes.** `team-alpha/.*` is correct. `team-alpha/*` (glob syntax) does NOT work — it doesn't match anything. This causes silent access denial that's hard to debug. Always test regex patterns.
- **JCasC permission format.** In JCasC YAML, permissions use the format `"Permission/Category:user_or_group"`. The slash direction matters, the colon separates permission from assignee, and the exact permission name must match. Typos cause silent failures where the permission just isn't applied.
- **Role Strategy `Run/Replay` permission controversy.** Replay allows re-running a build with a modified Jenkinsfile — effectively running arbitrary Groovy code on your agents. Giving `Run/Replay` to all developers is a security risk in highly regulated environments. Consider restricting it to senior developers or leads.
- **Folder-scoped credentials lookup path.** Jenkins looks for credentials in: current folder → parent folder → root → System. A credential in `team-alpha/` folder is NOT accessible to jobs in `team-alpha/subteam/` unless the subteam folder also has that credential or inherits from parent. This surprises people migrating from flat job structures.

---

### 🔗 Connections to Other Topics

| Connected Topic | How It Connects |
|-----------------|-----------------|
| JCasC (5.7) | Security config defined as code — RBAC, users, credentials all in YAML |
| Credentials (3.12) | Folder scoping provides the security boundary |
| Input Steps (4.4) | `submitter` parameter leverages the user/group system |
| Jenkins HA/DR (2.7) | Backup must include security realm + authorization strategy config |
| Org Folders (4.2) | Org folder items respect folder-scoped credentials and RBAC |

---
---
---

# Topic 5.5 — Kubernetes Plugin ⚠️

## 🔴 Advanced | Dynamic Elastic Build Infrastructure

---

### 📌 What It Is — In Simple Terms

The **Kubernetes Plugin** enables Jenkins to run each pipeline build in a dedicated Kubernetes Pod, rather than on a pre-provisioned static agent VM. The pod is created when a build starts and deleted when it finishes — perfectly elastic, automatically scaled, with no idle agent cost.

This is the dominant Jenkins agent model for teams running Kubernetes. It fundamentally changes how you think about build infrastructure: from "how many VMs do I need?" to "does the cluster have enough CPU/memory to run N builds in parallel?"

---

### 🔍 Why Kubernetes Agents — The Problem They Solve

```
Static agents (VMs/bare metal):
  ┌─────────────────────────────────────────────────────────────┐
  │ Problems:                                                   │
  │   - Peak capacity: must provision for maximum concurrent    │
  │     builds (e.g., 50 agents for Black Friday deploy spree)  │
  │   - Off-peak: 48 of 50 agents idle, still paying for VMs   │
  │   - "Works on my agent" bugs: stale tools, dirty state     │
  │   - Agent provisioning: slow (minutes to start new VM)      │
  │   - Tool version drift: agents get out of sync over time   │
  └─────────────────────────────────────────────────────────────┘

Kubernetes agents:
  ┌─────────────────────────────────────────────────────────────┐
  │ Solutions:                                                  │
  │   - Elastic: 0 agents at night, 100 at peak, autoscaled    │
  │   - Ephemeral: each build gets a fresh container (clean!)   │
  │   - Declarative: pod spec in Jenkinsfile = version-controlled│
  │   - Fast: pod starts in ~30 seconds (vs 5-10 min for VM)   │
  │   - Multi-container: maven + docker + trivy in same pod     │
  │   - Cost: pay only for actual build compute time           │
  └─────────────────────────────────────────────────────────────┘
```

---

### ⚙️ How the Kubernetes Plugin Works Internally

```
Build start sequence:
  1. Jenkins scheduler: build queued, needs agent with label 'kubernetes'
  2. Kubernetes plugin:
     a. Reads pod template (from Jenkinsfile or global config)
     b. Calls Kubernetes API: kubectl create pod jenkins-agent-xyz-42
     c. Pod starts: jnlp sidecar container pulls inbound-agent image
     d. jnlp container: connects to Jenkins Controller via WebSocket/TCP
     e. Jenkins: marks agent as online
     f. Jenkins: dispatches build to this agent
  3. Pipeline executes inside pod containers
  4. Build finishes
  5. Kubernetes plugin: kubectl delete pod jenkins-agent-xyz-42
  6. Pod deleted (workspace gone)

Pod structure:
  Every build pod has:
    - jnlp container: Jenkins agent sidecar (REQUIRED)
      - Handles Controller ↔ Agent communication
      - Manages workspace lifecycle
    - build containers: your tools (maven, docker, node, etc.)
      - Defined in podTemplate
      - Share workspace via emptyDir volume
```

---

### ⚙️ Pod Template Configuration — Complete Reference

```groovy
// ── INLINE POD TEMPLATE (in Jenkinsfile) ──────────────────────────
// Best for: service-specific needs, version-pinned tools

pipeline {
    agent {
        kubernetes {
            // Optional: inherit from a globally defined template
            inheritFrom 'base-pod'

            // Pod label (also serves as agent label)
            label "myapp-build-${UUID.randomUUID().toString()[0..7]}"

            // Kubernetes namespace for the build pod
            // Default: same namespace as Jenkins Controller
            namespace 'jenkins-builds'

            // Default container for steps that don't specify container()
            defaultContainer 'maven'

            // The pod spec — standard Kubernetes YAML
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-build
    team: myteam
  annotations:
    cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
spec:
  # Use a service account with limited permissions
  serviceAccountName: jenkins-build-sa

  # Tolerate spot/preemptible nodes for cost savings
  tolerations:
  - key: "spot"
    operator: "Exists"
    effect: "NoSchedule"

  # Prefer spot nodes
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: node.kubernetes.io/lifecycle
            operator: In
            values: ["spot"]

  # Resource limits apply PER CONTAINER
  containers:
  # ── JNLP SIDECAR (required) ─────────────────────────────────────
  - name: jnlp
    image: jenkins/inbound-agent:3206.vb_15dcf73f6b_9-1-jdk17
    resources:
      requests:
        cpu: "200m"
        memory: "256Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"
    # DO NOT set command/args for jnlp — it must use default entrypoint

  # ── MAVEN BUILD CONTAINER ────────────────────────────────────────
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command: ["cat"]    # Keep container alive (cat blocks on stdin)
    tty: true           # Needed for interactive shell
    resources:
      requests:
        cpu: "1000m"    # 1 CPU
        memory: "2Gi"
      limits:
        cpu: "2000m"    # 2 CPU
        memory: "4Gi"
    env:
    - name: MAVEN_OPTS
      value: "-Xmx2g -XX:+TieredCompilation -XX:TieredStopAtLevel=1"
    volumeMounts:
    - name: maven-cache
      mountPath: /root/.m2/repository  # Cache Maven local repo

  # ── DOCKER BUILD CONTAINER ───────────────────────────────────────
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.21.0-debug
    command: ["sleep"]
    args: ["infinity"]
    resources:
      requests:
        cpu: "500m"
        memory: "1Gi"
      limits:
        cpu: "1000m"
        memory: "2Gi"
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker

  # ── SECURITY SCANNING CONTAINER ─────────────────────────────────
  - name: trivy
    image: aquasec/trivy:0.48.3
    command: ["cat"]
    tty: true
    resources:
      requests:
        cpu: "200m"
        memory: "512Mi"
      limits:
        cpu: "500m"
        memory: "1Gi"

  # ── DEPLOY CONTAINER ─────────────────────────────────────────────
  - name: helm
    image: alpine/helm:3.14.0
    command: ["cat"]
    tty: true
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"

  # ── SHARED VOLUMES ────────────────────────────────────────────────
  volumes:
  # Maven cache: persist between builds for faster dependency resolution
  - name: maven-cache
    persistentVolumeClaim:
      claimName: maven-cache-pvc    # Pre-created PVC
  # Kaniko Docker credentials
  - name: kaniko-secret
    secret:
      secretName: docker-registry-credentials
      items:
      - key: .dockerconfigjson
        path: config.json
'''
        }
    }

    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Test') {
            steps {
                container('maven') {
                    sh 'mvn test'
                }
            }
            post {
                always { junit 'target/surefire-reports/*.xml' }
            }
        }

        stage('Scan') {
            parallel {
                stage('Build Image') {
                    steps {
                        container('kaniko') {
                            sh '''
                                /kaniko/executor \
                                    --context=dir:///workspace \
                                    --dockerfile=/workspace/Dockerfile \
                                    --destination=registry.example.com/myapp:${BUILD_NUMBER} \
                                    --cache=true
                            '''
                        }
                    }
                }
                stage('Security Scan') {
                    steps {
                        container('trivy') {
                            sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                container('helm') {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh 'helm upgrade --install myapp ./chart --namespace staging --wait'
                    }
                }
            }
        }
    }
}
```

---

### ⚙️ Global Pod Templates (Reusable Base Templates)

```yaml
# JCasC — Define shared pod templates once, reference in Jenkinsfiles
jenkins:
  clouds:
    - kubernetes:
        name: "kubernetes"
        serverUrl: ""            # Empty = use in-cluster config (Jenkins runs in K8s)
        namespace: "jenkins"
        jenkinsUrl: "http://jenkins.jenkins.svc.cluster.local:8080"
        jenkinsTunnel: "jenkins-agent.jenkins.svc.cluster.local:50000"
        connectTimeout: 5
        readTimeout: 15
        containerCapStr: "50"   # Max concurrent pods this cloud can create

        # ── GLOBAL POD TEMPLATES ──────────────────────────────────────
        templates:
          # Base template — inherited by all others
          - name: "base-pod"
            namespace: "jenkins-builds"
            nodeUsageMode: "EXCLUSIVE"
            serviceAccount: "jenkins-build-sa"
            containers:
              - name: jnlp
                image: "jenkins/inbound-agent:3206.vb_15dcf73f6b_9-1-jdk17"
                resourceRequestCpu: "200m"
                resourceRequestMemory: "256Mi"
                resourceLimitCpu: "500m"
                resourceLimitMemory: "512Mi"
            annotations:
              - key: "cluster-autoscaler.kubernetes.io/safe-to-evict"
                value: "false"
            podRetention: "never"          # Delete pod when build finishes
            activeDeadlineSeconds: 3600    # Kill pod after 1 hour (safety)

          # Maven Java build template
          - name: "maven-jdk17"
            inheritFrom: "base-pod"       # Inherits jnlp from base
            containers:
              - name: maven
                image: "maven:3.9-eclipse-temurin-17"
                command: "cat"
                ttyEnabled: true
                resourceRequestCpu: "1"
                resourceRequestMemory: "2Gi"
                resourceLimitCpu: "2"
                resourceLimitMemory: "4Gi"
                envVars:
                  - envVar:
                      key: "MAVEN_OPTS"
                      value: "-Xmx2g"
            volumes:
              - persistentVolumeClaim:
                  claimName: "maven-cache-pvc"
                  mountPath: "/root/.m2/repository"

          # Node.js build template
          - name: "nodejs-18"
            inheritFrom: "base-pod"
            containers:
              - name: node
                image: "node:18-alpine"
                command: "cat"
                ttyEnabled: true
                resourceRequestCpu: "500m"
                resourceRequestMemory: "1Gi"
                resourceLimitCpu: "1"
                resourceLimitMemory: "2Gi"

          # Python ML/data template (GPU-capable)
          - name: "python-ml"
            inheritFrom: "base-pod"
            nodeSelector: "gpu=true"
            tolerations:
              - key: "nvidia.com/gpu"
                operator: "Exists"
                effect: "NoSchedule"
            containers:
              - name: python
                image: "pytorch/pytorch:2.1.0-cuda11.8-cudnn8-runtime"
                command: "cat"
                ttyEnabled: true
                resourceRequestCpu: "2"
                resourceRequestMemory: "8Gi"
                resourceLimitCpu: "4"
                resourceLimitMemory: "16Gi"
                resourceRequestGpu: "1"    # Requires NVIDIA device plugin
                resourceLimitGpu: "1"
```

---

### ⚙️ Docker-in-Docker vs Kaniko — The Container Build Decision

```
Building Docker images inside a Kubernetes pod:

Option 1: DinD (Docker in Docker)
  - Run a Docker daemon INSIDE the pod as a sidecar
  - Requires privileged: true
  - Security risk: privileged container has host root access
  ❌ Avoid in production unless absolutely necessary

  containers:
  - name: dind
    image: docker:24-dind
    securityContext:
      privileged: true     ← Security hole
    env:
    - name: DOCKER_TLS_CERTDIR
      value: /certs

Option 2: Docker-outside-of-Docker (DooD)
  - Mount the host's Docker socket into the pod
  - Shares the host Docker daemon
  - Slightly less risky than DinD but still host root access
  ❌ Also risky: any build can interfere with other build's images

  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
  containers:
  - name: docker
    image: docker:24
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock

Option 3: Kaniko (recommended)
  - Build Docker images without a Docker daemon
  - Runs as a normal (unprivileged) container
  - Reads Dockerfile, builds layers, pushes to registry
  - No host access required
  ✅ Recommended for Kubernetes

  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.21.0-debug
    command: ["sleep"]
    args: ["infinity"]
    # NO securityContext.privileged needed

  # Usage:
  container('kaniko') {
      sh '''
          /kaniko/executor \
              --context=dir:///workspace \
              --dockerfile=/workspace/Dockerfile \
              --destination=registry.io/myapp:${BUILD_NUMBER} \
              --cache=true \
              --cache-repo=registry.io/myapp/cache
      '''
  }

Option 4: BuildKit / BuildX
  - Can run in rootless mode
  - More feature-rich than Kaniko (better caching, cross-platform builds)
  - More complex setup
  ✅ Good alternative to Kaniko for teams using multi-platform builds
```

---

### ⚙️ Spot/Preemptible Instance Optimization

```yaml
# Build pods on spot instances = 60-90% cost reduction
# Risk: pod may be evicted mid-build if spot node is preempted

# Strategy: run on spot, but prevent eviction during active builds

spec:
  # Prefer spot nodes
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: node.kubernetes.io/lifecycle
            operator: In
            values: ["spot", "preemptible"]

  # Tell cluster autoscaler: DON'T evict this pod for scale-down
  # (still evicted by node termination, but not by autoscaler scale-down)
  annotations:
    cluster-autoscaler.kubernetes.io/safe-to-evict: "false"

  # Tolerate spot node taints
  tolerations:
  - key: "spot"
    operator: "Exists"
    effect: "NoSchedule"
  - key: "cloud.google.com/gke-spot"
    operator: "Exists"
    effect: "NoSchedule"
```

---

### 🔑 Key Concepts

| Concept | Meaning |
|---------|---------|
| **Pod template** | Blueprint for the build pod — containers, resources, volumes, tolerations |
| **jnlp container** | Required sidecar handling Controller↔Agent communication |
| **`container('name')`** | Run steps inside a specific pod container |
| **`defaultContainer`** | Which container `sh` steps run in by default |
| **`inheritFrom`** | Inherit another pod template, override specific fields |
| **Kaniko** | Docker image builder that runs without Docker daemon (recommended) |
| **DinD** | Docker in Docker — requires privileged — security risk |
| **emptyDir volume** | Shared workspace between containers in the same pod |
| **`safe-to-evict: "false"`** | Prevent cluster autoscaler from evicting active build pods |
| **`activeDeadlineSeconds`** | Hard timeout: Kubernetes kills the pod if build exceeds this |
| **In-cluster config** | Jenkins running in K8s can talk to K8s API without explicit credentials |

---

### 💬 Short Crisp Interview Answer

> *"The Kubernetes Plugin creates a Pod per build, with the pipeline steps running inside pod containers. This makes build infrastructure elastic — zero pods when no builds are running, as many as the cluster can handle at peak. The pod has a required `jnlp` sidecar container that handles Controller-to-Agent communication, plus your build tool containers (maven, node, trivy, helm). All containers share the workspace via an emptyDir volume. You switch between containers with `container('maven')` wrapping steps. For building Docker images inside Kubernetes pods, I use Kaniko — it builds container images without a Docker daemon, running unprivileged. DinD requires `privileged: true` which grants host root access — avoid it in production. For cost optimization, build pods run on spot/preemptible nodes with `safe-to-evict: false` annotation to prevent the cluster autoscaler from evicting an active build. Pod templates are defined globally in JCasC with `inheritFrom` for composition, and overridden per-pipeline in the Jenkinsfile for service-specific needs."*

---

### ⚠️ Tricky Edge Cases & Gotchas

- **⚠️ `command: ["cat"] + tty: true` on every non-jnlp container.** Without this, containers exit immediately when they start (no process to keep them alive). The `cat` command blocks on stdin, keeping the container running. The `tty: true` is needed for interactive shell features. This pattern is required for every build tool container. Forgetting it causes "container exited" errors.
- **⚠️ `jnlp` container must use default command/args.** If you set `command` or `args` on the jnlp container, it breaks the agent connection. The inbound-agent image has a specific entrypoint that handles the Jenkins connection — overriding it prevents the agent from ever connecting.
- **Resource requests vs limits.** In Kubernetes, resource `requests` affect scheduling (where the pod lands) and `limits` affect what the container actually gets. If `limits.memory` is lower than your Maven build needs (`-Xmx2g` but limit is 1Gi), the container gets OOMKilled. Set `limits.memory` >= JVM max heap + 30% overhead. Set requests low enough for efficient bin-packing.
- **`safe-to-evict: "false"` doesn't protect against node termination.** It only prevents the cluster autoscaler from initiating scale-down. If a spot node is preempted by the cloud provider (AWS/GCP taking the node back), the pod dies regardless. For truly critical builds, use on-demand nodes.
- **Workspace cleanup.** Since each build gets a new pod, the workspace is always clean. But this means no incremental builds — Maven downloads all dependencies from scratch every build unless you mount a shared PVC for the Maven cache. The `maven-cache-pvc` volume mount is critical for build performance.
- **Concurrent builds share the cluster namespace.** 100 concurrent builds = 100 pods in `jenkins-builds` namespace. Each pod has its own workspace on the node's ephemeral storage. If nodes have small disks (e.g., 50GB root volume), many large workspaces can fill the disk. Use `nodeSelector` to route builds to nodes with larger ephemeral storage.

---

### 🔗 Connections to Other Topics

| Connected Topic | How It Connects |
|-----------------|-----------------|
| Agent Types (2.4) | Kubernetes pod agents are the most important agent type |
| Pipeline Durability (4.7) | K8s agents → PERFORMANCE_OPTIMIZED (pods die on restart anyway) |
| Parallel Stages (3.7) | Each parallel branch gets its own pod — scales naturally |
| JCasC (5.7) | Global pod templates defined in JCasC YAML |
| Jenkins HA/DR (2.7) | K8s Jenkins deployment on PVC enables faster recovery |
| Stash/Unstash (3.10) | emptyDir volumes replace stash within a pod |

---
---

# Topic 5.6 — Docker Plugin vs Docker Pipeline Plugin

## 🟡 Intermediate | Two Different Docker Integrations

---

### 📌 What It Is — In Simple Terms

Jenkins has TWO distinct Docker-related plugins that serve completely different purposes and are frequently confused:

- **Docker Plugin** — creates Docker containers AS AGENTS (the container replaces the VM agent)
- **Docker Pipeline Plugin** — provides Docker DSL STEPS used WITHIN a pipeline running on any agent

Confusing these two is a common interview error. Understanding the difference reveals whether you've actually worked with Jenkins + Docker.

---

### ⚙️ Docker Plugin — Containers as Agents

```
Plugin ID: docker-plugin
Purpose:   Run each Jenkins build in a Docker container on a Docker-enabled host
           The container IS the agent — it receives the build, runs steps, is deleted

How it works:
  Jenkins Controller → Docker API → Docker host → creates container
  Container: jenkins/inbound-agent + your tools
  Container connects back to Controller as an agent
  Build runs inside container
  Container deleted when build finishes

This is the non-Kubernetes equivalent of the Kubernetes Plugin.
Use when: your build infrastructure is Docker hosts, not Kubernetes.
```

```yaml
# JCasC — Docker Plugin cloud configuration
jenkins:
  clouds:
    - docker:
        name: "docker-cloud"
        dockerApi:
          dockerHost:
            uri: "unix:///var/run/docker.sock"     # Local Docker socket
            # OR for remote Docker host:
            # uri: "tcp://docker-host.example.com:2376"
            # credentialsId: "docker-tls-credentials"

        templates:
          - labelString: "docker-maven"
            dockerTemplateBase:
              image: "maven:3.9-eclipse-temurin-17"
              volumes:
                - "/root/.m2:/root/.m2:rw"   # Maven cache mount
              cpuShares: 1024
              memoryLimit: 4096              # MB
            remoteFs: "/home/jenkins"
            connector:
              jnlp:
                jnlpLauncher:
                  workDirSettings:
                    disabled: false
            instanceCapStr: "10"            # Max 10 concurrent containers of this type
            retentionStrategy:
              once:
                idleMinutes: 5              # Remove after 5 min idle
```

```groovy
// Using Docker Plugin agents in pipeline:
pipeline {
    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-17'
            label 'docker'           // Must run on a Docker-enabled agent
            args  '-v /root/.m2:/root/.m2 --memory=4g'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

---

### ⚙️ Docker Pipeline Plugin — Docker Commands in Pipelines

```
Plugin ID: docker-workflow
Purpose:   Provides Docker-related Groovy DSL steps for use WITHIN pipelines
           The pipeline still runs on a normal agent — this plugin adds
           docker.image(), docker.build(), docker.withRegistry() etc.

Key steps provided:
  docker.image('name')          ← Create a Docker image reference object
  docker.build('name')          ← Run docker build
  docker.withRegistry(url, creds) {} ← docker login, wrap steps, docker logout
  docker.withServer(url, creds)  {} ← Connect to a remote Docker daemon

This plugin is for: running docker commands IN your pipeline steps
Not for: using Docker as the build agent itself (that's Docker Plugin)
```

```groovy
// Docker Pipeline Plugin usage — in a pipeline running on any agent

pipeline {
    agent { label 'docker-host' }    // Any agent with Docker installed

    environment {
        REGISTRY    = 'registry.example.com'
        IMAGE_NAME  = "${REGISTRY}/myapp"
        IMAGE_TAG   = "${BUILD_NUMBER}-${GIT_COMMIT.take(7)}"
    }

    stages {
        // ── BUILD THE IMAGE ────────────────────────────────────────
        stage('Build Image') {
            steps {
                script {
                    // docker.build() runs: docker build -t IMAGE_NAME .
                    def image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    // 'image' is a Docker image object you can use for push, run, etc.
                }
            }
        }

        // ── RUN TESTS INSIDE THE IMAGE ─────────────────────────────
        stage('Test in Container') {
            steps {
                script {
                    def image = docker.image("${IMAGE_NAME}:${IMAGE_TAG}")

                    // Run tests inside the built container:
                    image.inside('-e CI=true -v /tmp/test-results:/app/results') {
                        // 'inside' mounts the workspace into the container
                        sh 'npm test'
                        sh 'npm run integration-test'
                    }
                    // Container exits when the closure ends
                }
            }
        }

        // ── PUSH TO REGISTRY ───────────────────────────────────────
        stage('Push') {
            steps {
                script {
                    // docker.withRegistry handles login/logout automatically
                    docker.withRegistry("https://${REGISTRY}",
                                        'registry-credentials') {
                        def image = docker.image("${IMAGE_NAME}:${IMAGE_TAG}")
                        image.push()           // Push tagged version
                        image.push('latest')   // Also push as 'latest'
                    }
                    // Automatically runs docker logout after block
                }
            }
        }

        // ── PULL AND USE AN EXISTING IMAGE ─────────────────────────
        stage('Integration Test with Service') {
            steps {
                script {
                    // Pull and run a database for integration tests
                    docker.image('postgres:15-alpine').withRun(
                        '-e POSTGRES_PASSWORD=test -e POSTGRES_DB=testdb -p 5432:5432'
                    ) { dbContainer ->
                        // dbContainer is running — run your tests against it
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").inside(
                            "--link ${dbContainer.id}:db -e DB_HOST=db"
                        ) {
                            sh 'mvn verify -Pintegration'
                        }
                    }
                    // Database container automatically stopped and removed
                }
            }
        }

        // ── BUILD WITH DOCKERFILE IN SPECIFIC DIRECTORY ────────────
        stage('Build Service Image') {
            steps {
                script {
                    // docker.build with custom context and dockerfile:
                    def image = docker.build(
                        "${IMAGE_NAME}:${IMAGE_TAG}",
                        "-f services/myservice/Dockerfile services/myservice/"
                    )
                    // Equivalent to:
                    // docker build -t IMAGE:TAG -f services/myservice/Dockerfile services/myservice/
                }
            }
        }
    }
}
```

---

### ⚙️ Docker Plugin vs Docker Pipeline Plugin — Decision Matrix

```
Docker Plugin (docker-plugin):
  ✅ Use when:
     - Build infrastructure is Docker hosts (not Kubernetes)
     - Want container-per-build isolation on Docker hosts
     - Simple setup vs Kubernetes Plugin complexity
  
  Configuration location: Jenkins cloud settings (JCasC or GUI)
  Agent type: Docker container IS the agent
  Scope: Infrastructure-level
  Modern usage: Less common — most teams on Kubernetes use K8s plugin instead

Docker Pipeline Plugin (docker-workflow):
  ✅ Use when:
     - Need to build Docker images in your pipeline
     - Need to run tests inside containers
     - Need to push images to registries
     - Agent is already provisioned (VM, K8s pod) — you just want Docker commands
  
  Configuration location: Jenkinsfile (docker.build(), docker.withRegistry())
  Agent type: Uses whatever agent the pipeline already has
  Scope: Pipeline-step-level
  Modern usage: Common — used alongside Kubernetes plugin

Using both together:
  pipeline {
      agent {
          kubernetes { ... }       ← K8s Plugin: provisions the build pod
      }
      stages {
          stage('Build Image') {
              steps {
                  container('kaniko') {      ← Kaniko for image building (not Docker Pipeline Plugin)
                      sh '/kaniko/executor ...'
                  }
              }
          }
      }
  }
  // In Kubernetes environments, Kaniko is preferred over Docker Pipeline Plugin
  // for image building (no daemon needed, no privileged containers)
```

---

### 💬 Short Crisp Interview Answer

> *"Jenkins has two distinct Docker-related plugins that serve completely different purposes. The Docker Plugin (`docker-plugin`) provisions Docker containers as agents — each build runs inside a fresh Docker container on a Docker host, similar to how the Kubernetes Plugin creates pod agents. It's the Docker-host equivalent of the Kubernetes Plugin. The Docker Pipeline Plugin (`docker-workflow`) provides Docker DSL steps for use within pipelines: `docker.build()` to build images, `docker.withRegistry()` for authenticated registry pushes, and `docker.image().inside()` to run commands inside a specific container. These two plugins are independent — you can use Docker Pipeline Plugin steps on any agent type (VMs, K8s pods, Mac agents). In Kubernetes environments, teams typically use the Kubernetes Plugin for agent provisioning and Kaniko for container image building, which makes the Docker Pipeline Plugin less relevant for image building but it's still used for `withRegistry()` and running existing images."*

---

### ⚠️ Tricky Edge Cases & Gotchas

- **⚠️ Docker Pipeline Plugin requires Docker installed on the agent.** `docker.build()` and `docker.withRegistry()` call the `docker` CLI on the agent's filesystem. If the agent is a plain Maven container (in a Kubernetes pod), it doesn't have Docker installed — these calls fail. Use Kaniko for image building in K8s environments instead.
- **`docker.image().inside()` mounts the Jenkins workspace into the container.** This means the running container has the workspace as a bind mount. Any file the container writes is in the workspace. This is usually desired behavior but can surprise people who expect a clean container environment.
- **`withRun()` cleanup is automatic but not instant.** After the `withRun {}` block closes, Jenkins sends `docker stop` + `docker rm`. The stop has a default 10-second grace period. If you call `withRun` many times, container cleanup may lag behind creation.
- **Docker Pipeline Plugin + `agent none` for multi-platform builds.** You can use `docker.build()` with BuildKit flags for multi-platform builds, but the agent must have `docker buildx` configured. This gets complex quickly — consider a dedicated multi-platform build solution.

---
---

# Topic 5.7 — Configuration as Code (JCasC) Plugin ⚠️

## 🔴 Advanced | The Most Important Operational Plugin

---

### 📌 What It Is — In Simple Terms

**JCasC (Jenkins Configuration as Code)** is a plugin that lets you define the ENTIRE Jenkins system configuration — security realms, authorization strategy, global credentials, cloud providers, LDAP settings, pipeline libraries, agent templates, tool installations, email settings — as human-readable YAML files committed to version control.

Without JCasC: Jenkins configuration lives in XML files in JENKINS_HOME that nobody reviews, aren't version-controlled, and are difficult to audit or reproduce. A new Jenkins instance requires hours of GUI clicking to configure.

With JCasC: New Jenkins instance → apply one YAML file → fully configured in minutes. Configuration changes → PR review → merge → applied automatically.

---

### 🔍 Why JCasC Is Critical

```
Without JCasC:
  Configuration is in JENKINS_HOME/config.xml (XML, not human-readable)
  Change process:
    1. Click through GUI
    2. Save
    3. Change is live immediately (no review)
    4. No record of who changed what when
    5. To reproduce: manually click through GUI again
  
  Disaster scenario:
    Jenkins data corrupted → restore from backup → 
    backup was 3 days ago → 3 days of config changes lost →
    spend hours manually reconfiguring

With JCasC:
  Configuration is in casc.yaml in a Git repo
  Change process:
    1. Submit PR with config change
    2. Peer review
    3. Merge → CI pipeline applies config → Jenkins updated
    4. Full audit trail in Git
    5. Rollback = git revert + apply
  
  Disaster scenario:
    Jenkins data corrupted → new Jenkins instance →
    apply casc.yaml from Git → fully configured in 3 minutes
    All pipelines auto-discovered via Multi-Branch/Org Folder
```

---

### ⚙️ JCasC File Structure — Complete Reference

```yaml
# jenkins-casc.yaml — Complete JCasC reference

# ── JENKINS SYSTEM SETTINGS ──────────────────────────────────────────
jenkins:

  # Basic system info
  systemMessage: |
    <b>Production Jenkins — Managed via Configuration as Code</b>
    Any manual changes will be overwritten on next config reload.
    Modify: https://github.com/myorg/jenkins-config/blob/main/casc.yaml

  numExecutors: 0                # Controller executors (0 = controller is not a build node)
  scmCheckoutRetryCount: 2       # Retry SCM checkouts on failure

  # ── SECURITY REALM (Authentication) ────────────────────────────────
  securityRealm:
    # Option 1: LDAP
    ldap:
      configurations:
        - server: "ldap://ldap.example.com:389"
          rootDN: "dc=example,dc=com"
          userSearchBase: "ou=users"
          userSearch: "uid={0}"
          groupSearchBase: "ou=groups"
          managerDN: "cn=jenkins-sa,ou=service-accounts,dc=example,dc=com"
          managerPasswordSecret: "${LDAP_MANAGER_PASSWORD}"  # From env var
          displayNameAttributeName: "displayName"
          mailAddressAttributeName: "mail"

    # Option 2: GitHub OAuth
    # github:
    #   githubWebUri: "https://github.com"
    #   githubApiUri: "https://api.github.com"
    #   clientID: "${GITHUB_OAUTH_CLIENT_ID}"
    #   clientSecret: "${GITHUB_OAUTH_CLIENT_SECRET}"
    #   oauthScopes: "read:org,user:email"

    # Option 3: SAML (enterprise SSO)
    # saml:
    #   idpMetadataUrl: "https://sso.example.com/saml/metadata"
    #   displayNameAttributeName: "displayName"
    #   groupsAttributeName: "groups"

  # ── AUTHORIZATION STRATEGY ─────────────────────────────────────────
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions:
              - "Overall/Administer"
            assignments:
              - "jenkins-admins"

          - name: "developer"
            permissions:
              - "Overall/Read"
              - "Job/Build"
              - "Job/Cancel"
              - "Job/Read"
              - "View/Read"
              - "Run/Replay"
              - "Agent/Build"
            assignments:
              - "developers"
              - "authenticated"   # All logged-in users get developer role

          - name: "anonymous-readonly"
            permissions:
              - "Overall/Read"
              - "Job/Read"
              - "View/Read"
            assignments:
              - "anonymous"     # Unauthenticated users can only read

        items:
          - name: "ops-deployer"
            pattern: ".*/deploy-production"
            permissions:
              - "Job/Build"
              - "Job/Read"
            assignments:
              - "release-managers"
              - "sre-on-call"

  # ── CRUMB ISSUER (CSRF protection) ─────────────────────────────────
  crumbIssuer:
    standard:
      excludeClientIPFromCrumb: true   # Required when behind a load balancer

  # ── GLOBAL QUIET PERIOD ────────────────────────────────────────────
  quietPeriod: 5   # seconds — debounce rapid pushes

  # ── CLOUD PROVIDERS (Agent Backends) ───────────────────────────────
  clouds:
    - kubernetes:
        name: "kubernetes"
        namespace: "jenkins"
        serverUrl: ""    # Empty = in-cluster config (Jenkins runs in K8s)
        jenkinsUrl: "http://jenkins.jenkins.svc.cluster.local:8080"
        jenkinsTunnel: "jenkins-agent.jenkins.svc.cluster.local:50000"
        containerCapStr: "50"
        connectTimeout: 5
        readTimeout: 15
        podRetentionStr: "never"
        templates:
          - name: "base-pod"
            namespace: "jenkins-builds"
            serviceAccount: "jenkins-build-sa"
            containers:
              - name: jnlp
                image: "jenkins/inbound-agent:3206.vb_15dcf73f6b_9-1-jdk17"
                resourceRequestCpu: "200m"
                resourceRequestMemory: "256Mi"
                resourceLimitCpu: "500m"
                resourceLimitMemory: "512Mi"
            annotations:
              - key: "cluster-autoscaler.kubernetes.io/safe-to-evict"
                value: "false"
          - name: "maven-jdk17"
            inheritFrom: "base-pod"
            containers:
              - name: maven
                image: "maven:3.9-eclipse-temurin-17"
                command: "cat"
                ttyEnabled: true
                resourceRequestCpu: "1"
                resourceRequestMemory: "2Gi"
                resourceLimitCpu: "2"
                resourceLimitMemory: "4Gi"

# ── GLOBAL CREDENTIALS ────────────────────────────────────────────────
credentials:
  system:
    domainCredentials:
      - credentials:
          # GitHub PAT for SCM checkout
          - usernamePassword:
              scope:    GLOBAL
              id:       "github-credentials"
              username: "jenkins-ci-bot"
              password: "${GITHUB_TOKEN}"    # From environment variable — NOT plaintext!
              description: "GitHub CI Bot token"

          # Docker Registry credentials
          - usernamePassword:
              scope:    GLOBAL
              id:       "docker-registry"
              username: "jenkins"
              password: "${DOCKER_REGISTRY_TOKEN}"
              description: "Docker Registry push credentials"

          # Slack webhook token
          - string:
              scope:    GLOBAL
              id:       "slack-token"
              secret:   "${SLACK_BOT_TOKEN}"
              description: "Slack bot token for notifications"

          # Kubeconfig file for staging
          - file:
              scope:    GLOBAL
              id:       "kubeconfig-staging"
              fileName: "config"
              secretBytes: "${readFile:/vault/secrets/kubeconfig-staging}"
              # OR: secretBytes: "${KUBECONFIG_STAGING_BASE64}"
              description: "Kubernetes config for staging cluster"

          # SSH key for Git operations
          - basicSSHUserPrivateKey:
              scope:       GLOBAL
              id:          "github-ssh-key"
              username:    "git"
              description: "SSH key for GitHub access"
              privateKeySource:
                directEntry:
                  privateKey: "${GITHUB_SSH_PRIVATE_KEY}"

# ── GLOBAL PIPELINE LIBRARIES ────────────────────────────────────────
unclassified:
  globalLibraries:
    libraries:
      - name: "jenkins-shared-library"
        defaultVersion: "main"
        implicit: false               # Don't auto-load in every pipeline
        allowVersionOverride: true    # Allow @Library('lib@version')
        includeInChangesets: false
        retriever:
          modernSCM:
            scm:
              git:
                remote: "https://github.com/myorg/jenkins-shared-library.git"
                credentialsId: "github-credentials"

  # ── SLACK CONFIGURATION ───────────────────────────────────────────
  slackNotifier:
    teamDomain: "mycompany"
    tokenCredentialId: "slack-token"
    room: "#build-notifications"
    botUser: true

  # ── EMAIL SETTINGS ────────────────────────────────────────────────
  mailer:
    smtpHost: "smtp.example.com"
    smtpPort: "587"
    useSsl: false
    useTls: true
    smtpAuthUserName: "jenkins@example.com"
    smtpAuthPassword: "${SMTP_PASSWORD}"
    adminAddress: "jenkins@example.com"
    defaultSuffix: "@example.com"   # Appended to usernames for email

  # ── SONARQUBE CONFIGURATION ───────────────────────────────────────
  sonarGlobalConfiguration:
    installations:
      - name: "SonarQube"
        serverUrl: "https://sonarqube.example.com"
        credentialsId: "sonarqube-token"
        mojoVersion: ""
        triggers:
          skipScmCause: false
          skipUpstreamCause: false

  # ── PIPELINE DURABILITY (GLOBAL DEFAULT) ─────────────────────────
  pipelineTriggeredJobsSettings:
    durabilityHintItems:
      - hint: PERFORMANCE_OPTIMIZED
        enabled: true

  # ── LOCATION CONFIGURATION ────────────────────────────────────────
  location:
    url: "https://jenkins.example.com/"
    adminAddress: "admin@example.com"

# ── GLOBAL TOOL INSTALLATIONS ─────────────────────────────────────────
tool:
  git:
    installations:
      - name: "Default"
        home: "git"   # Use system git
  maven:
    installations:
      - name: "Maven-3.9"
        home: ""
        properties:
          - installSource:
              installers:
                - maven:
                    id: "3.9.6"   # Install Maven 3.9.6 automatically
  jdk:
    installations:
      - name: "JDK-17"
        home: ""
        properties:
          - installSource:
              installers:
                - adoptOpenJDKInstaller:
                    id: "jdk-17.0.9+9"
  nodejs:
    installations:
      - name: "NodeJS-18"
        home: ""
        properties:
          - installSource:
              installers:
                - nodeJSInstaller:
                    id: "18.19.0"
                    npmPackages: "yarn@1.22.21"   # Also install yarn

# ── JOBS (bootstrap jobs) ─────────────────────────────────────────────
jobs:
  # Create organization folders that auto-discover repos
  - script: |
      organizationFolder('github-org') {
        displayName('GitHub Organization CI')
        organizations {
          github {
            apiUri('https://api.github.com')
            credentialsId('github-credentials')
            repoOwner('myorg')
            traits {
              gitHubBranchDiscovery { strategyId(1) }
              gitHubPullRequestDiscovery { strategyId(1) }
            }
          }
        }
        orphanedItemStrategy {
          discardOldItems { numToKeep(20) }
        }
      }
```

---

### ⚙️ JCasC Secrets Handling — The Critical Security Concern

```yaml
# NEVER put actual secrets in casc.yaml — use variable substitution

# ── CORRECT: Environment variable substitution ──────────────────────
credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              password: "${GITHUB_TOKEN}"    # Set as env var before Jenkins starts

# How to set the env vars:
# 1. Kubernetes Secret → mounted as environment variables in Jenkins pod:
apiVersion: v1
kind: Secret
metadata:
  name: jenkins-secrets
  namespace: jenkins
type: Opaque
stringData:
  GITHUB_TOKEN: "ghp_xxxxxxxxxxxx"
  SLACK_BOT_TOKEN: "xoxb-xxxxxxxxxxxxx"
  LDAP_MANAGER_PASSWORD: "ldap-password"
---
# In Jenkins deployment:
spec:
  containers:
  - name: jenkins
    envFrom:
    - secretRef:
        name: jenkins-secrets

# 2. Vault Agent Injector — write secrets to files:
#    vault.hashicorp.com/agent-inject: "true"
#    vault.hashicorp.com/agent-inject-secret-github: "secret/jenkins/github"
#    Secrets appear at: /vault/secrets/github
#    In casc.yaml: password: "${readFile:/vault/secrets/github}"

# 3. AWS Secrets Manager (if running on AWS):
#    Use aws-secrets-manager-credentials-provider plugin
#    Credentials auto-populated from AWS Secrets Manager

# ── ANTI-PATTERN: Secrets in casc.yaml ──────────────────────────────
credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              password: "ghp_myactualtoken"  # ❌ NEVER — committed to Git
```

---

### ⚙️ JCasC Reload Without Restart

```bash
# JCasC can reload configuration without Jenkins restart:

# Method 1: Jenkins UI
# Manage Jenkins → Configuration as Code → Reload Existing Configuration

# Method 2: Jenkins API
curl -X POST \
  https://jenkins.example.com/configuration-as-code/reload \
  -u admin:${ADMIN_TOKEN} \
  --data-urlencode ""

# Method 3: Jenkins CLI
java -jar jenkins-cli.jar \
  -s https://jenkins.example.com \
  -auth admin:${ADMIN_TOKEN} \
  reload-jcasc

# Automatic reload: watch casc.yaml for changes
# Set CASC_JENKINS_CONFIG env var to a Git-synced directory
# Use a sidecar container that runs git pull every 60s
# JCasC picks up the file change and reloads automatically

# ── CASC_JENKINS_CONFIG ────────────────────────────────────────────
# Jenkins reads JCasC from paths configured in this env var:
env:
  CASC_JENKINS_CONFIG: "/var/jenkins_casc/jenkins.yaml"
  # OR: Point to a directory with multiple YAML files:
  # CASC_JENKINS_CONFIG: "/var/jenkins_casc/"

# ── SPLITTING LARGE CASC FILES ─────────────────────────────────────
# CASC_JENKINS_CONFIG can point to a directory
# Each .yaml file in the directory is merged:
/var/jenkins_casc/
  security.yaml       ← securityRealm + authorizationStrategy
  credentials.yaml    ← all credentials
  clouds.yaml         ← Kubernetes cloud + pod templates
  tools.yaml          ← JDK, Maven, Node installations
  notifications.yaml  ← Slack, email configuration
  jobs.yaml           ← Seed jobs + org folders
```

---

### ⚙️ Testing JCasC Changes

```bash
# ── VALIDATE CASC YAML BEFORE APPLYING ────────────────────────────
# Method 1: Jenkins API validation endpoint
curl -X POST \
  https://jenkins.example.com/configuration-as-code/checkNewSource \
  -u admin:${ADMIN_TOKEN} \
  --data-urlencode "newSource@jenkins.yaml"
# Returns: OK or error message with line number

# Method 2: Docker-based validation (no running Jenkins needed)
docker run --rm \
  -v $(pwd)/jenkins.yaml:/var/jenkins_casc/jenkins.yaml \
  -e CASC_JENKINS_CONFIG=/var/jenkins_casc/jenkins.yaml \
  -e GITHUB_TOKEN=test-token \
  jenkins/jenkins:lts-jdk17 \
  /bin/bash -c "java -jar /usr/share/jenkins/jenkins.war --httpPort=8080 &
                sleep 30 && curl -s http://localhost:8080/configuration-as-code/schema"

# ── CI PIPELINE FOR CASC CHANGES ─────────────────────────────────
# In the jenkins-config repo:
pipeline {
    agent any
    stages {
        stage('Validate YAML') {
            steps {
                sh 'yamllint jenkins.yaml'   # Check YAML syntax
            }
        }
        stage('Validate JCasC Schema') {
            steps {
                // Apply to a dev Jenkins instance and verify no errors
                sh '''
                    curl -X POST \
                        ${DEV_JENKINS_URL}/configuration-as-code/checkNewSource \
                        -u admin:${DEV_JENKINS_TOKEN} \
                        --data-urlencode "newSource@jenkins.yaml"
                '''
            }
        }
        stage('Apply to Dev Jenkins') {
            steps {
                sh '''
                    # Copy to dev Jenkins's CASC path
                    scp jenkins.yaml jenkins-dev.internal:/var/jenkins_casc/jenkins.yaml
                    # Trigger reload
                    curl -X POST ${DEV_JENKINS_URL}/configuration-as-code/reload \
                        -u admin:${DEV_JENKINS_TOKEN}
                '''
            }
        }
        stage('Apply to Production Jenkins') {
            when { branch 'main' }
            steps {
                input 'Apply to production Jenkins?'
                sh 'kubectl cp jenkins.yaml jenkins/jenkins-0:/var/jenkins_casc/'
                sh 'curl -X POST ${PROD_JENKINS_URL}/configuration-as-code/reload ...'
            }
        }
    }
}
```

---

### 🔑 Key Concepts

| Concept | Meaning |
|---------|---------|
| `CASC_JENKINS_CONFIG` | Env var pointing to JCasC YAML file/directory |
| `${ENV_VAR}` in YAML | Variable substitution — secrets from environment, not hardcoded |
| Hot reload | Apply config changes without Jenkins restart |
| Schema export | Download JCasC schema for IDE autocompletion |
| `jenkins:` block | Jenkins system configuration (security, clouds, numExecutors) |
| `unclassified:` block | Plugin-specific settings that don't fit `jenkins:` block |
| `credentials:` block | Global credential definitions |
| `tool:` block | Global tool installations (JDK, Maven, Node, Git) |
| `jobs:` block | Groovy DSL scripts to create initial jobs |
| Split YAML | Directory mode: multiple YAML files merged together |

---

### 💬 Short Crisp Interview Answer

> *"JCasC is the most important operational plugin for production Jenkins. It lets you define the entire Jenkins system configuration — security realm, authorization strategy, credentials, Kubernetes cloud configuration, shared libraries, SMTP settings, Slack config — as YAML files committed to version control. Without JCasC, configuration lives in JENKINS_HOME XML that no one reviews and is difficult to reproduce. With JCasC: a new Jenkins instance reaches full production config in minutes by applying one YAML file; config changes go through PR review; git history is the audit trail; and rollback is a `git revert`. The critical security discipline: NEVER put secrets in the YAML file itself. Use `${ENV_VAR}` substitution — secrets come from Kubernetes Secrets mounted as environment variables, or from Vault via the agent sidecar injector. JCasC supports hot reload without Jenkins restart, which is critical for zero-downtime config changes in production."*

---

### ⚠️ Tricky Edge Cases & Gotchas

- **⚠️ JCasC `jobs:` block uses Job DSL syntax, NOT Declarative Pipeline.** The `jobs:` section uses the Job DSL plugin's Groovy DSL to CREATE Jenkins job configurations — it doesn't contain Jenkinsfile content. `jobs: - script: | multibranchPipelineJob('myapp') { ... }` creates a Multi-Branch Pipeline job. The Jenkinsfile content lives in your application repositories.
- **`${ENV_VAR}` is JCasC variable syntax, NOT Groovy.** In `password: "${GITHUB_TOKEN}"`, this is JCasC's own `${VAR}` substitution, evaluated when the YAML is loaded. The secret value comes from the process environment. This looks similar to Groovy GStrings but runs at YAML load time, not pipeline time.
- **JCasC does NOT fully manage all Jenkins state.** It manages system configuration but NOT: build history, pipeline run logs, archived artifacts, or installed plugins. Use separate backup strategies for build history and a separate plugin installation mechanism (plugins.txt/jenkins-plugin-cli) for plugins.
- **Schema changes between Jenkins versions.** JCasC schema is generated from the actual plugins installed. Upgrading Jenkins or plugins can change YAML field names. Always validate your CASC YAML against the actual Jenkins version you're deploying to. Use the schema export endpoint to get the current valid schema.
- **Reload creates a transient moment of inconsistency.** During a JCasC reload, there's a brief window where some settings are updated and others aren't. Active builds during reload may see partial configuration. Schedule reloads during low-traffic periods or ensure your settings are backward-compatible.

---
---

# Topic 5.8 — Job DSL Plugin

## 🔴 Advanced | Programmatic Job Management

---

### 📌 What It Is — In Simple Terms

The **Job DSL Plugin** provides a Groovy-based DSL for programmatically creating, updating, and managing Jenkins jobs. You write Groovy scripts that define job configurations, and a "seed job" executes those scripts to create/update the jobs. It's "Infrastructure as Code" for Jenkins job definitions.

**Historical context:** Job DSL emerged before Multi-Branch Pipelines and Organization Folders existed. It was the primary way to manage many Jenkins jobs at scale. Today, its role has narrowed significantly — Organization Folders auto-discover pipelines, making Job DSL less necessary for standard CI pipelines. But it remains valuable for complex job orchestration that can't be expressed with auto-discovery.

---

### 🔍 Job DSL in the Modern Context

```
When Job DSL WAS essential (pre-2016):
  Creating 50 Freestyle jobs → 50 GUI clicks per job
  Job DSL: 50 jobs = 1 seed script
  Updating all 50 → run the seed job once

When Job DSL IS STILL valuable today:
  ✅ Creating non-Pipeline job types: Folders, Views, freestyle jobs
  ✅ Complex job dependency graphs that auto-discovery doesn't model
  ✅ Bootstrapping Jenkins with required folder structure + permissions
  ✅ Migration: converting Freestyle jobs → Pipeline jobs systematically
  ✅ Creating Views that organize jobs by team/type
  ✅ Environments where Org Folders can't be used (Bitbucket Server, older SCM)

When to use Organization Folder INSTEAD of Job DSL:
  ✅ Standard multi-branch CI pipelines
  ✅ GitHub/GitLab/Bitbucket Cloud repos
  ✅ Auto-discovery is sufficient
  → Organization Folders require ZERO Job DSL maintenance

When to use JCasC INSTEAD of Job DSL:
  ✅ System configuration (security, credentials, clouds)
  → JCasC handles system config better than Job DSL
```

---

### ⚙️ Job DSL — Seed Job Pattern

```
Architecture:
  "Seed job" — a special Jenkins Pipeline or Freestyle job
               that runs Job DSL scripts to create/update other jobs

  Seed job lifecycle:
    1. Developer modifies jobs.groovy in the config repo
    2. Pushes to main branch
    3. Config repo's Jenkinsfile triggers the seed job
    4. Seed job runs processJobDsl() step with the jobs.groovy script
    5. Jenkins creates/updates/deletes jobs as defined by the script
    6. All affected jobs now reflect the new configuration
```

```groovy
// ── SEED JOB JENKINSFILE ───────────────────────────────────────────
// lives in: jenkins-config-repo/Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Apply Job DSL') {
            steps {
                // processJobDsl: read all .groovy files and execute them
                jobDsl(
                    targets:                    'jobs/**/*.groovy',  // Glob pattern
                    removedJobAction:           'DELETE',     // Delete jobs removed from DSL
                    removedViewAction:          'DELETE',     // Delete views removed from DSL
                    removedConfigFilesAction:   'DELETE',
                    lookupStrategy:             'JENKINS_ROOT',  // Jobs created at Jenkins root
                    sandbox:                    false,       // Requires Script Approval if true
                    additionalClasspath:        '',
                    failOnMissingPlugin:        false,
                    unstableOnDeprecation:      true
                )
            }
        }
    }
}
```

---

### ⚙️ Job DSL Syntax — Complete Reference

```groovy
// ── jobs/pipelines.groovy ─────────────────────────────────────────
// This script is run by the seed job to create Pipeline jobs

// ── CREATE A MULTI-BRANCH PIPELINE ───────────────────────────────
multibranchPipelineJob('myapp') {
    displayName('My Application')
    description('CI/CD pipeline for the main application')

    branchSources {
        github {
            id('myapp-source')
            repoOwner('myorg')
            repository('myapp')
            credentialsId('github-credentials')
            traits {
                gitHubBranchDiscovery { strategyId(1) }
                gitHubPullRequestDiscovery { strategyId(1) }
                headWildcardFilter {
                    includes('main release/* feature/*')
                    excludes('wip/*')
                }
            }
        }
    }

    orphanedItemStrategy {
        discardOldItems {
            numToKeep(20)
            daysToKeep(7)
        }
    }

    configure { node ->
        // Raw XML configuration for settings not exposed in DSL
        node / 'triggers' / 'com.cloudbees.hudson.plugins.folder.computed.PeriodicFolderTrigger' {
            spec('H/5 * * * *')
            interval('300000')
        }
    }
}

// ── CREATE ORGANIZATION FOLDER ────────────────────────────────────
organizationFolder('github-org') {
    displayName('GitHub Organization CI')
    description('Auto-discovers all repos in myorg')

    organizations {
        github {
            credentialsId('github-credentials')
            repoOwner('myorg')
            traits {
                gitHubBranchDiscovery { strategyId(1) }
                gitHubPullRequestDiscovery { strategyId(1) }
                sourceWildcardFilter {
                    includes('*')
                    excludes('sandbox-* archived-*')
                }
            }
        }
    }

    projectFactories {
        workflowMultiBranchProjectFactory {
            scriptPath('Jenkinsfile')
        }
    }

    orphanedItemStrategy {
        discardOldItems { numToKeep(5) }
    }
}

// ── CREATE A FOLDER WITH PERMISSIONS ─────────────────────────────
folder('team-alpha') {
    displayName('Team Alpha Projects')
    description('CI/CD pipelines for Team Alpha')

    // Configure folder-level properties
    properties {
        // Folder-scoped credentials
        folderCredentialsProperty {
            domainCredentials {
                domainCredentials {
                    domain {
                        description('Team Alpha credentials')
                    }
                    credentials {
                        // Add team-specific credentials to this folder
                        usernamePassword {
                            scope('GLOBAL')
                            id('team-alpha-db-password')
                            username('alpha-db-user')
                            password('${TEAM_ALPHA_DB_PASSWORD}')  // From environment
                        }
                    }
                }
            }
        }
    }

    // Configure authorization strategy for this folder (folder-level RBAC)
    configure { project ->
        project / 'properties' / 'com.cloudbees.hudson.plugins.folder.FolderAllowSiblingScm'
    }
}

// ── CREATE A PARAMETERIZED PIPELINE JOB ───────────────────────────
pipelineJob('manual-deploy') {
    displayName('Manual Production Deploy')
    description('Trigger a production deployment with specific image tag')

    parameters {
        stringParam('IMAGE_TAG',       '',         'Docker image tag to deploy')
        choiceParam('TARGET_ENV',      ['staging', 'production', 'dev'], 'Target environment')
        booleanParam('SKIP_APPROVAL',  false,      'Skip manual approval (use only for hotfixes)')
    }

    triggers {
        // No auto-trigger — manually triggered only
    }

    definition {
        cpsScm {
            scm {
                git {
                    remote {
                        url('https://github.com/myorg/deploy-scripts.git')
                        credentials('github-credentials')
                    }
                    branch('*/main')
                }
            }
            scriptPath('pipelines/manual-deploy.Jenkinsfile')
        }
    }

    logRotator {
        numToKeep(50)
        daysToKeep(90)
        artifactNumToKeep(10)
    }
}

// ── CREATE A LIST VIEW ─────────────────────────────────────────────
listView('Production Deployments') {
    description('All production deployment pipelines')
    filterBuildQueue(true)
    filterExecutors(true)

    jobs {
        // Include jobs matching regex
        regex('.*deploy-production.*')
        // OR include specific jobs:
        // names('myapp-pipeline', 'payments-pipeline', 'auth-pipeline')
    }

    columns {
        status()
        weather()
        name()
        lastSuccess()
        lastFailure()
        lastDuration()
        buildButton()
    }
}

// ── GENERATE JOBS FROM DATA ────────────────────────────────────────
// The real power of Job DSL: create N jobs from a list
def services = [
    [name: 'auth-service',     repo: 'auth-service',     team: 'identity'],
    [name: 'payment-service',  repo: 'payment-service',  team: 'payments'],
    [name: 'order-service',    repo: 'order-service',    team: 'fulfillment'],
    [name: 'inventory-service',repo: 'inventory-service',team: 'supply'],
]

services.each { svc ->
    // Create a folder per team
    folder(svc.team) {
        displayName("${svc.team.capitalize()} Team")
    }

    // Create a Multi-Branch Pipeline in the team's folder
    multibranchPipelineJob("${svc.team}/${svc.name}") {
        displayName(svc.name.replace('-', ' ').capitalize())
        branchSources {
            github {
                id("${svc.name}-source")
                repoOwner('myorg')
                repository(svc.repo)
                credentialsId('github-credentials')
            }
        }
        orphanedItemStrategy {
            discardOldItems { numToKeep(10) }
        }
    }
}
```

---

### ⚙️ Raw XML Configuration — When DSL Doesn't Cover It

```groovy
// Job DSL covers common settings but not everything
// For settings not in the DSL, use the configure{} block to add raw XML

pipelineJob('myapp') {
    // Standard DSL settings:
    logRotator { numToKeep(20) }

    // Raw XML for settings not exposed in DSL API:
    configure { project ->
        // Add a property not in the DSL:
        project / 'properties' << {
            'org.jenkinsci.plugins.workflow.job.properties.DisableConcurrentBuildsJobProperty' {
                abortPrevious(true)  // Abort older builds when new one starts
            }
        }

        // Add a trigger not in the DSL:
        project / 'triggers' << {
            'hudson.triggers.TimerTrigger' {
                spec('H 2 * * 1')  // Weekly cron
            }
        }
    }
}
```

---

### 🔑 Key Concepts

| Concept | Meaning |
|---------|---------|
| **Seed job** | The Jenkins job that runs Job DSL scripts to create/update other jobs |
| **`jobDsl()`** | Pipeline step that processes Job DSL Groovy scripts |
| **`removedJobAction: 'DELETE'`** | Delete jobs no longer defined in DSL scripts |
| **`multibranchPipelineJob()`** | DSL for creating Multi-Branch Pipeline jobs |
| **`organizationFolder()`** | DSL for creating Org Folder jobs |
| **`pipelineJob()`** | DSL for creating regular Pipeline jobs |
| **`folder()`** | DSL for creating Jenkins folders |
| **`configure {}`** | Escape hatch for raw XML configuration |
| **Dynamic job generation** | Loop over a list to create N jobs from N data points |
| **Script sandbox** | Security restriction on Job DSL scripts — usually disable for trusted config repos |

---

### 💬 Short Crisp Interview Answer

> *"Job DSL provides a Groovy-based DSL for programmatically creating and managing Jenkins jobs. A 'seed job' runs Job DSL scripts that define job configurations — Multi-Branch Pipelines, Organization Folders, regular Pipeline jobs, Views, Folders — and Jenkins creates or updates those jobs accordingly. With `removedJobAction: 'DELETE'`, jobs removed from the DSL script are automatically deleted. The main use case today: bootstrapping Jenkins with the initial folder structure and Org Folder/Multi-Branch Pipeline configurations that auto-discovery then takes over from. The dynamic generation pattern is powerful — loop over a list of services and create a folder + Multi-Branch Pipeline job for each with one script. That said, if Organization Folders already auto-discover your repos, you don't need Job DSL for those pipelines. JCasC handles system configuration better than Job DSL. So today, Job DSL fills a specific niche: creating non-auto-discoverable job types and complex job structures that JCasC's `jobs:` block initiates."*

---

### ⚠️ Tricky Edge Cases & Gotchas

- **⚠️ Script sandbox must be disabled for trusted seed jobs.** With `sandbox: true`, Job DSL scripts run in a restricted Groovy sandbox and many methods are blocked (file I/O, network calls, certain Groovy APIs). For trusted config repos, disable the sandbox (`sandbox: false`). For untrusted sources, enable sandbox but be aware of limitations.
- **`removedJobAction: 'DELETE'` deletes build history.** When a job is removed from the DSL script and the seed job runs with `removedJobAction: 'DELETE'`, the job AND all its build history, logs, and archived artifacts are deleted. This is usually correct but verify you have what you need before removing jobs from DSL.
- **Job DSL and JCasC `jobs:` block serve the same purpose.** JCasC's `jobs:` block also accepts Job DSL Groovy scripts. Using JCasC `jobs:` for simple org folder bootstrapping and a full seed job for complex dynamic generation is a common hybrid approach.
- **Job DSL API version drift.** The Job DSL plugin generates its API from the currently installed plugins. Method signatures change when plugins update. A DSL script that worked with Jenkins 2.x + plugin version A may fail with newer plugin versions. Always test DSL changes against your actual Jenkins version.

---

### 🔗 Connections to Other Topics

| Connected Topic | How It Connects |
|-----------------|-----------------|
| JCasC (5.7) | JCasC `jobs:` block uses Job DSL syntax for bootstrapping |
| Org Folders (4.2) | Org Folder replaces most Job DSL use cases for CI pipelines |
| Multi-Branch Pipeline (4.1) | Job DSL creates Multi-Branch Pipeline configurations |
| Shared Libraries (3.13) | Job DSL can configure which libraries are available to jobs |
| RBAC (5.4) | Job DSL can configure folder-level permissions |

---
---

# 📊 Category 5 Summary — Quick Reference

| Topic | Core Concept | Interview Priority |
|-------|-------------|-------------------|
| 5.1 Essential Plugins | Git, Pipeline suite, Credentials, ws-cleanup, Timestamper — pin versions | ⭐⭐⭐ |
| 5.2 Trigger Plugins | Generic Webhook > polling; GitHub/GitLab for native integration | ⭐⭐⭐⭐ |
| 5.3 Notification Plugins | Slack threading, `culprits()`, PagerDuty resolve — avoid fatigue | ⭐⭐⭐ |
| 5.4 Security Plugins | RBAC roles, item patterns, folder credentials, audit trail | ⭐⭐⭐⭐ |
| 5.5 Kubernetes Plugin ⚠️ | Pod templates, jnlp sidecar, Kaniko, spot instances, resource limits | ⭐⭐⭐⭐⭐ |
| 5.6 Docker Plugin vs Docker Pipeline | Docker Plugin = agents; Docker Pipeline Plugin = steps | ⭐⭐⭐ |
| 5.7 JCasC ⚠️ | Full system config as YAML, `${ENV_VAR}` secrets, hot reload | ⭐⭐⭐⭐⭐ |
| 5.8 Job DSL | Seed job pattern, dynamic job generation, modern role is narrower | ⭐⭐⭐ |

---

## 🔑 The Mental Model for Category 5

```
PLUGINS are Jenkins's "organs" — the core is a skeleton,
  plugins provide all the actual functionality.
  Rule: Pin versions. Test updates on dev Jenkins first.

TRIGGER PLUGINS: Webhooks > Polling
  GitHub/GitLab plugin for native integration
  Generic Webhook for anything else (JSONPath extracts any field)
  No trigger plugin needed for cron and upstream

SECURITY: Two layers
  Authentication: Who are you? (LDAP, GitHub OAuth, SAML)
  Authorization: What can you do? (Role Strategy Plugin)
  Team isolation: Folder-scoped credentials + item role patterns
  Compliance: Audit Trail plugin for every action log

KUBERNETES PLUGIN: The modern agent model
  Pod = ephemeral agent (fresh per build, deleted on completion)
  jnlp sidecar = required (DO NOT override its command)
  Kaniko > DinD (no privileged containers needed)
  PERFORMANCE_OPTIMIZED durability (pods die on Controller restart anyway)

JCASC: The operational foundation
  ALL system config as YAML in Git
  Secrets via ${ENV_VAR} substitution (never in the file)
  Hot reload without restart
  Disaster recovery: new Jenkins + CASC file = fully configured in minutes

JOB DSL: Narrower role today
  Org Folders handle auto-discovery (no DSL needed)
  JCasC handles system config (no DSL needed)
  Job DSL niche: bootstrapping complex folder structures, dynamic job generation
```

---

## 🧪 Self-Quiz — 10 Interview Questions

1. Why is `command: ["cat"]` + `tty: true` required on every non-jnlp container in a Kubernetes pod template?

2. What's the difference between the Docker Plugin and Docker Pipeline Plugin? Give a concrete usage example of each.

3. A team wants to run Docker image builds inside their Kubernetes pod agents. Should they use DinD, DooD, or Kaniko? Explain the tradeoffs.

4. You need to restrict Team Alpha so they can ONLY see and build their own jobs, and can ONLY use their own production credentials. What combination of plugins and configurations achieves this?

5. How do you store secrets in JCasC without committing them to Git? Show the YAML and explain the mechanism.

6. What does `culprits()` do in email-ext and why is it more useful than emailing the whole team on failure?

7. A developer adds `triggers { githubPush() }` to the Jenkinsfile of a Multi-Branch Pipeline. Does this have any effect? Why or why not?

8. Explain JCasC hot reload. How does it work and what's the risk during a reload?

9. When would you use Job DSL in 2024+ alongside JCasC and Organization Folders?

10. A new engineer joins and installs 10 plugins on the production Jenkins by clicking "Install without restart." What's the risk and what process should they have followed?

---

*Next: Category 6 — Jenkins Integrations (GitHub PRs, Kubernetes deployments, Artifactory, SonarQube, Vault)*
*Or: "quiz me" to test yourself on Category 5*

---
> **Document:** Category 5 Complete | Jenkins Plugins Ecosystem
> **Coverage:** 8 topics | Beginner → Advanced | From essential plugins to JCasC architecture
> **Key insight:** JCasC + Kubernetes Plugin are the two skills that separate production Jenkins operators from hobbyists
