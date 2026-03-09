**Why Monitor Jenkins?**
- Without monitoring:
  - Jenkins slows down -> You find out when developers complain
  - Build queue is full -> Jobs waiting 30 mins, nobody knows why
  - Agent disk is full -> Builds failing randomly
  - 50% of builds failing -> Nobody noticed for 3 days
  - Jenkins master is down -> You find out from angry Slack messages
- With monitoring:
  - Jenkins slows down -> Grafana alert fires immediately
  - Build queue growing -> alert: "queue length > 10"
  - Agent disk filling up -> alert before it causes failures
  - Build failure rate spike -> alert: "failure rate > 20%"
  - Jenkins master down -> PagerDuty wakes you up
 
### Jenkins Prometheus Plugin ###
Prometheus is a monitoring tool that scrapes metrics from your applications. The Jenkins Prometheus plugin exposes Jenkins metrics in a format Prometheus understands.

**How it works**
```bash
Jenkins Prometheus Plugin
        ↓
Exposes metrics at:
  http://your-jenkins.com/prometheus/
        ↓
Prometheus scrapes this URL every 15 seconds
        ↓
Stores metrics in Prometheus time-series database
        ↓
Grafana reads from Prometheus
        ↓
Displays dashboards and fires alerts
```

**Installing the Plugin**
```bash
Jenkins → Manage Jenkins → Plugins → Available
Search: "Prometheus metrics"
Install and restart Jenkins
```
- After installing, metrics are available at:
```bash
http://your-jenkins.com/prometheus/

Example output:
  jenkins_builds_duration_milliseconds_summary{...} 45230
  jenkins_queue_size_value 3
  jenkins_executor_count_value{...} 10
  jenkins_executor_in_use_value{...} 7
```

**Configuring Prometheus to Scrape Jenkins**
- In your `prometheus.yml`:
```yaml
scrape_configs:

  - job_name: 'jenkins'
    scrape_interval: 15s
    metrics_path: /prometheus
    static_configs:
      - targets:
          - 'your-jenkins.com:8080'

    # if Jenkins requires authentication
    basic_auth:
      username: 'prometheus-user'
      password: 'prometheus-password'
```

---
---

### Key Metrics to Monitor ###

**Metric 1 — Build Duration**
- What it tells you: How long builds are taking. Sudden increases mean something is wrong.
- Prometheus metric: `jenkins_builds_duration_milliseconds_summary`
- What to watch:
  - Normal: Build takes 5 minutes
  - Warning: Build takes 10 minutes -> Something is slowing it down
  - Critical: Build takes 30 minutes -> Something is very wrong
- Common causes of slow builds:
  - Tests are hanging
  - Agent is overloaded
  - Network issues pulling dependencies
  - Docker layer cache missing
- Grafana query to show average build duration:
  ```bash
  rate(jenkins_builds_duration_milliseconds_summary_sum[5m])
  /
  rate(jenkins_builds_duration_milliseconds_summary_count[5m])
  ```

**Metric 2 — Queue Length**
- What it tells you: How many builds are waiting for an agent. High queue means not enough agents.
- Prometheus metric: `jenkins_queue_size_value`
- What to watch:
  - Normal: Queue = 0-2 -> Builds start immediately
  - Warning: Queue = 5-10 -> Developers waiting for builds
  - Critical: Queue > 10 -> Serious agent shortage
- Common causes of high queue:
  - Not enough agents
  - Agents are offline
  - All agents busy with long-running builds
  - A stuck build is occupying an agent
- Grafana query: `jenkins_queue_size_value`

**Metric 3 — Agent Utilization**
- What it tells you: How busy your agents are. Helps you decide if you need more agents or fewer.
- Prometheus metrics:
  - `jenkins_executor_count_value` -> Total executors available
  - `jenkins_executor_in_use_value` -> Executors currently running builds
  - Utilization % = (in_use / total) × 100
- What to watch:
  - Utilization 0-50% -> Agents are underused, maybe scale down
  - Utilization 50-80% -> Healthy range
  - Utilization > 90% -> Agents are overloaded, add more agents
- Grafana query for utilization percentage: `(jenkins_executor_in_use_value / jenkins_executor_count_value) * 100`

**Metric 4 — Build Success vs Failure Rate**
- What it tells you: What percentage of builds are passing or failing.
- Prometheus metrics:
  - jenkins_builds_success_build_count -> Successful builds
  - jenkins_builds_failed_build_count -> Failed builds
  - jenkins_builds_unstable_build_count -> Unstable builds (test failures)
- What to watch:
  - Failure rate < 10% -> Healthy
  - Failure rate > 20% -> Something is wrong, investigate
  - Failure rate > 50% -> Critical, builds are broken
- Grafana query for failure rate:
  ```bash
  rate(jenkins_builds_failed_build_count[1h])
  /
  (rate(jenkins_builds_success_build_count[1h]) + rate(jenkins_builds_failed_build_count[1h]))
  * 100
  ```

**Metric 5 — Jenkins System Health**
- What it tells you: CPU, memory, disk usage of Jenkins master itself.
- Prometheus metrics (from Node Exporter on Jenkins server):
  - CPU: `rate(node_cpu_seconds_total{mode!="idle"}[5m])`
  - Memory: `node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes`
  - Disk: `node_filesystem_avail_bytes / node_filesystem_size_bytes`
- What to watch:
  - CPU    > 80% -> Jenkins master overloaded
  - Memory > 85% -> Risk of OOM kills
  - Disk   < 20% -> Artifacts filling up disk, cleanup needed
 
---
---

### Grafana Dashboards for Jenkins ###

**How Grafana connects to Prometheus**
```bash
Grafana → Add Data Source → Prometheus
  URL: http://prometheus-server:9090
        ↓
Grafana can now query all Jenkins metrics
and display them as graphs, gauges, tables
```

**Using Pre-built Grafana Dashboard**
```bash
Grafana → Dashboards → Import
  Dashboard ID: 9964
  (Jenkins: Performance and Health Overview — community dashboard)
        ↓
Select your Prometheus data source
        ↓
Dashboard loads with all Jenkins metrics pre-configured
```

---
---

### Build Failure Alerting ###
Knowing about failures through Grafana dashboards is good. But you also need active alerts that notify your team immediately.

**Method 1 — Email Notifications (Built into Jenkins)**
- The simplest alerting — Jenkins sends email when builds fail.
```groovy
pipeline {
    agent any

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
    }

    post {
        failure {
            mail(
                to: 'team@mycompany.com',
                subject: "Build Failed: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                    Build failed!

                    Job:    ${JOB_NAME}
                    Build:  #${BUILD_NUMBER}
                    Branch: ${BRANCH_NAME}

                    Check the build here:
                    ${BUILD_URL}

                    Console output:
                    ${BUILD_URL}console
                """
            )
        }

        success {
            mail(
                to: 'team@mycompany.com',
                subject: "Build Passed: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "Build succeeded. See: ${BUILD_URL}"
            )
        }
    }
}
```
- Configure SMTP in Jenkins:
```bash
Jenkins → Manage Jenkins → Configure System → Email Notification

  SMTP server:  smtp.gmail.com
  Port:         465
  Use SSL:      ✅
  Username:     jenkins@mycompany.com
  Password:     (app password from Gmail)
```

**Method 2 — Slack Notifications (Most Common in Modern Teams)**
- More visible than email. The whole team sees failures in the channel immediately.
- Install plugin: `Jenkins → Plugins → Install "Slack Notification Plugin"`
- Configure Slack in Jenkins
```bash
Jenkins → Manage Jenkins → Configure System → Slack

  Workspace:    mycompany
  Credential:   (Slack Bot Token stored in Jenkins credentials)
  Channel:      #jenkins-alerts
```
- In Jenkinsfile
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#deployments',
                color: 'good',            // green
                message: """
                    *Build Passed*
                    Job: ${JOB_NAME}
                    Build: #${BUILD_NUMBER}
                    Branch: ${BRANCH_NAME}
                    Duration: ${currentBuild.durationString}
                    <${BUILD_URL}|View Build>
                """
            )
        }

        failure {
            slackSend(
                channel: '#alerts',
                color: 'danger',          // red
                message: """
                    *Build Failed*
                    Job: ${JOB_NAME}
                    Build: #${BUILD_NUMBER}
                    Branch: ${BRANCH_NAME}
                    Author: ${env.GIT_AUTHOR_NAME}
                    <${BUILD_URL}|View Build> | <${BUILD_URL}console|Console Log>
                """
            )
        }

        unstable {
            slackSend(
                channel: '#alerts',
                color: 'warning',         // yellow
                message: """
                    *Build Unstable — Tests Failing*
                    Job: ${JOB_NAME}
                    Build: #${BUILD_NUMBER}
                    <${BUILD_URL}|View Build>
                """
            )
        }
    }
}
```
- What the Slack message looks like:
```bash
#alerts channel:

  Build Failed
  Job:      myapp/main
  Build:    #142
  Branch:   main
  Author:   john.doe
  View Build | Console Log
```

**Method 3 — Grafana Alerts (Infrastructure-level Alerting)**
- Grafana alerts fire based on metric thresholds — not just build failures but system-level issues.
```bash
Alert 1 — Queue too long:
  Condition: jenkins_queue_size_value > 10
  For:       5 minutes
  Notify:    Slack #alerts
  Message:   "Jenkins queue has 10+ builds waiting. Add more agents."

Alert 2 — High failure rate:
  Condition: build_failure_rate > 20%
  For:       15 minutes
  Notify:    Slack #alerts + Email team lead
  Message:   "Jenkins failure rate above 20% for 15 minutes"

Alert 3 — Disk filling up:
  Condition: disk_free_percent < 15%
  For:       5 minutes
  Notify:    PagerDuty (wakes someone up)
  Message:   "Jenkins master disk below 15% free"

Alert 4 — Jenkins master down:
  Condition: up{job="jenkins"} == 0
  For:       2 minutes
  Notify:    PagerDuty immediately
  Message:   "Jenkins master is unreachable"
```

---
---

### Log Management for Jenkins ###
Jenkins generates a lot of logs — build logs, system logs, agent logs. Managing them properly is important.

**Types of Jenkins Logs**
- Build Console Logs
  - Output of each build (what you see when you click a build)
  - stored in `JENKINS_HOME/jobs/jobname/builds/`
- Jenkins System Log
  - Jenkins application logs (startup, plugin errors, etc.)
  - `Jenkins → Manage Jenkins → System Log`
- Agent Logs
  - Logs from each agent connection
  - `Jenkins → Manage Nodes → Click Agent → Log`
 
**Problem with Default Log Storage**
```bash
Jenkins stores all logs on its own disk
        ↓
100 jobs × 50 builds each × 10MB per log
= 50GB of logs on Jenkins disk 
```
- Old logs never cleaned up automatically 
- Cannot search across all logs 
- Agent logs lost when agent restarts 

**Solution 1 — Build Discarder (Most Important)**
- Always configure how long to keep builds and logs:
```groovy
pipeline {
    agent any

    options {
        // keep only last 30 builds
        // keeps disk usage under control
        buildDiscarder(logRotator(
            numToKeepStr: '30',         // keep 30 builds
            daysToKeepStr: '14',        // or 14 days — whichever is less
            artifactNumToKeepStr: '5',  // keep artifacts for only 5 builds
            artifactDaysToKeepStr: '7'
        ))
    }

    stages { ... }
}
```

**Solution 2 — Sending Logs to ELK Stack**
- ELK = Elasticsearch + Logstash + Kibana. Centralizes all logs in one searchable place.
```bash
Jenkins build runs
        ↓
Build logs sent to Logstash
(using Logstash Jenkins plugin)
        ↓
Logstash processes and ships to Elasticsearch
        ↓
Kibana lets you search all logs across all builds
```
```groovy
// Jenkinsfile with Logstash plugin
pipeline {
    agent any

    // wrap entire pipeline — all output goes to Logstash
    options {
        logstashSend(failBuild: false)
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

**Solution 3 — Sending Logs to AWS CloudWatch**
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package 2>&1 | tee build.log'
            }
        }
    }

    post {
        always {
            // ship log to CloudWatch
            sh '''
                aws logs put-log-events \
                    --log-group-name /jenkins/builds \
                    --log-stream-name ${JOB_NAME}-${BUILD_NUMBER} \
                    --log-events file://build.log \
                    --region us-east-1
            '''
        }
    }
}
```
