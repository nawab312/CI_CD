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

### What happens at runtime:
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
