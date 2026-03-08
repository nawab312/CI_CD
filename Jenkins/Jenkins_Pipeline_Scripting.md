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

**quietPeriod**
- Adds a deliberate delay (in seconds) before a build actually starts after being triggered.
