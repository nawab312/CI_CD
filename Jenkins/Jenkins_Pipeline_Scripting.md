### Pipeline Execution Optimization ###

**Throttle Concurrent Builds and Cancel Previous Running Jobs**

In Jenkins CI/CD pipelines, managing concurrent builds efficiently is crucial to avoid unnecessary resource consumption and ensure faster feedback loops. Two important strategies for handling this are:
- Throttling concurrent builds – Limits the number of builds running simultaneously.
- Canceling previous running jobs – Ensures that when a new build is triggered, any currently running build of the same job is automatically aborted to prevent redundant executions.

*Throttling Concurrent Builds in Jenkins*

Throttling concurrent builds allows you to control the number of builds that can run simultaneously. This is useful when:
- You have limited system resources (CPU, memory, etc.).
- Your pipeline involves shared resources (e.g., databases, APIs, environments).
- You want to ensure that only a fixed number of builds execute at the same time.

How to Enable Throttling in Jenkins?
- Using the `Throttle Concurrent Builds` Plugin
  - Install Plugin:
    - Go to Manage Jenkins > Plugin Manager > Available Plugins.
    - Search for "Throttle Concurrent Builds" and install it.
  - Configure Throttling:
    - Go to the Jenkins job configuration page.
    - Find the "Throttle Concurrent Builds" section.
    - Enable "Throttle this project as part of one or more categories".
    - Set the maximum number of concurrent builds allowed.
- Using Declarative Pipeline. This ensures that only one build runs at a time for this job.
```groovy
pipeline {
    options {
        disableConcurrentBuilds()
    }
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Building the application..."
                sleep 30
            }
        }
    }
}
```

**Canceling Previous Running Jobs in Jenkins**

If a new commit is pushed while a build is running, canceling the previous job ensures that only the latest commit is built, which helps:
- Avoid wasting time and resources on outdated commits.
- Speed up feedback loops in a CI/CD workflow.
- Ensure that only the latest and most relevant code is tested and deployed.

How to Cancel Previous Running Jobs?
- Using Declarative Pipeline (`abortPrevious: true`). If a new build starts, the currently running build is automatically aborted.
```groovy
pipeline {
    options {
        disableConcurrentBuilds(abortPrevious: true)
    }
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Building the project..."
                sleep 30  // Simulating a long build process
            }
        }
    }
}
```
- Using Jenkins Job Configuration (UI)
  - Open the Jenkins job configuration page.
  - Scroll to "Throttle Concurrent Builds" section.
  - Enable "Abort Previous Running Builds".
 
**Using the quietPeriod directive**

The `quietPeriod` directive in Jenkins introduces a delay (in seconds) before a job starts execution after being triggered.
- To avoid triggering multiple builds unnecessarily when multiple commits or SCM changes happen in quick succession.
- Helps in batching changes together for a single build execution.
- Prevents unnecessary resource utilization in high-commit-frequency environments.

How quietPeriod Works. When a job is triggered (e.g., by a Git commit), Jenkins:
- Waits for the specified quietPeriod (e.g., 10 seconds).
- Checks if another trigger occurs during this time (e.g., another commit).
- If a new trigger occurs within the quiet period, Jenkins resets the timer.
- The job executes only once after the final quiet period expires.

```groovy
pipeline {
    options {
        quietPeriod(10) // Waits 10 seconds before execution
    }
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Executing the build after quiet period..."
            }
        }
    }
}
```
