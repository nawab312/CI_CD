Your team is using Jenkins for CI/CD, and you notice that a specific job sometimes fails due to a dependency conflict, but rerunning the job without any code changes sometimes makes it pass.
- What could be the possible reasons for this intermittent failure?
- How would you go about debugging and permanently fixing it?

### Solution ###
- The issue described relates to **CI/CD dependency resolution and build consistency**.
- In continuous integration pipelines, dependencies are often fetched dynamically from *package repositories, caches, or artifact stores*.
- If dependency versions are not locked properly or there are issues with caching, builds can become non-deterministic, leading to intermittent failures.

Possible Reasons for Intermittent Failure:

**Floating Dependencies:**
- If your build process fetches the latest available version of dependencies instead of a fixed version, a new (possibly incompatible) version could be introduced between runs.
```groovy
stage('Install Dependencies') {
  steps {
    sh 'npm install'  // This will install floating versions if ^ or ~ is used in package.json
  }
}
```
```json
//package.json
"dependencies": {
    "express": "^4.17.1"
}
```
- *Use `npm ci` instead of `npm install` to enforce fixed versions from `package-lock.json`*
- When you run `npm ci` (short for Clean Install), it does the following:
  - Deletes `node_modules/` and ensures a clean slate.
  - Strictly installs dependencies from `package-lock.json`:
    - Ignores `package.json` for resolving versions.
    - Fails if `package-lock.json` is missing or out of sync with `package.json`.
  - Does NOT modify `package-lock.json`:
    - Unlike `npm install`, which updates the lock file if necessary.
```json
//package-lock.json
{
  "dependencies": {
    "express": {
      "version": "4.17.1"
    }
  }
}
```

**Dependency Conflict in Transitive Dependencies:**
- Dependency conflicts occur when different libraries require different versions of the same dependency, and the build system must decide which version to use
- Example: Dependency Conflict in a Node.js (Jenkins Pipeline)
  - Library A requires `lodash@4.17.10`
  - Library B requires `lodash@4.17.21`
  - NPM might pick an arbitrary version, causing inconsistencies.
```groovy
stage('Install Dependencies') {
    steps {
        sh 'npm ci'  // Ensures locked versions
    }
}
stage('Check Dependency Tree') {
    steps {
        sh 'npm list lodash'
    }
}
stage('Build & Test') {
    steps {
        sh 'npm run build'
        sh 'npm test'
    }
}
```
- Example package.json with Dependency Conflict
```json
{
  "dependencies": {
    "library-a": "1.0.0",   // Requires lodash@4.17.10
    "library-b": "2.0.0"    // Requires lodash@4.17.21
  }
}
```
- NPM may install two versions (`node_modules/library-a/node_modules/lodash` and `node_modules/library-b/node_modules/lodash`). This can cause unexpected behavior if both are used.

- Force a Specific Version
```json
"dependencies": {
  "library-a": "1.0.0",
  "library-b": "2.0.0",
  "lodash": "4.17.21"
}
```

**Race Conditions in Dependency Fetching**
- A *race condition* occurs when multiple Jenkins builds run *in parallel* and try to *fetch, modify, or upload dependencies* in a shared artifact repository (like Nexus, Artifactory, or a shared NPM registry). This can cause:
  - *Corrupt or inconsistent dependencies:* If one build uploads a new version while another is downloading the previous one.
- Example: Race Condition in a Jenkins Pipeline (Maven Build)
  - Jenkins has two parallel jobs:
    - *Job A* is building and deploying `my-library-1.0.0.jar` to Artifactory.
    - *Job B* is running tests and fetching `my-library-1.0.0.jar` from the same repository.
  - Race Condition:
    - *Job A* uploads a new version while *Job B* is still fetching it.
    - *Job B* may get a corrupt or incomplete artifact, leading to build failures.
```groovy
pipeline {
    agent any
    stages {
        stage('Parallel Build & Fetch') {
            parallel {
                stage('Build and Deploy') {
                    steps {
                        sh './mvnw clean package'
                        sh './mvnw deploy'  // Uploads artifact to Nexus/Artifactory
                    }
                }
                stage('Fetch Dependency') {
                    steps {
                        sh './mvnw dependency:resolve'  // Fetches dependencies
                        sh './mvnw test'
                    }
                }
            }
        }
    }
}
```
- What Can Go Wrong?
  - Job A uploads `my-library-1.0.0.jar` while Job B is still fetching it.
  - Job B might:
    - Fetch a partially uploaded/corrupt JAR.
   
**Unstable Remote Repositories in Jenkins Pipeline**
- When your Jenkins build fetches dependencies from external repositories like Maven Central, PyPI, or npm, there is always a risk that:
  - The remote repository is down (e.g., Maven Central is temporarily unavailable).
  - Packages are removed or changed (dependency version no longer exists).

Jenkins Pipeline (Maven Build with External Dependencies)
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }
    }
}
```
If Maven Central is down or has connectivity issues, you might see an error like:
```bash
[ERROR] Failed to resolve dependencies for project com.example:myapp:jar:1.0
[ERROR] Could not transfer artifact org.springframework.boot:spring-boot-starter:jar:2.7.0
[ERROR] Connection refused: connect
```

- *How to Prevent Failures Due to Unstable Remote Repositories*
- Use a Local Cache (`.m2` for Maven, `npm ci`, `pip cache`): Instead of fetching dependencies from remote repositories every time, cache them locally.
  - Below Jenkinsfile  forces Maven to *use locally cached dependencies* instead of fetching new ones.
```groovy
pipeline {
    agent any
    environment {
        MAVEN_OPTS = "-Dmaven.repo.local=/home/jenkins/.m2/repository"
    }
    stages {
        stage('Build') {
            steps {
                sh './mvnw clean package --offline' #The dependencies must have been downloaded at least once before using --offline.
            }
        }
    }
}
```

- Set Up an Internal Proxy Repository (Nexus/Artifactory)
  - Instead of relying on Maven Central, PyPI, or npm directly, set up a proxy repository (e.g., Nexus or JFrog Artifactory)
  - If Maven Central goes down, your builds still work because the proxy has cached copies of dependencies. Faster builds as dependencies are fetched from a local proxy instead of the internet.
  - Change `pom.xml` to Use a Nexus Proxy
```xml
<repositories>
    <repository>
        <id>nexus-proxy</id>
        <url>http://nexus.example.com/repository/maven-central/</url>
    </repository>
</repositories>
```

