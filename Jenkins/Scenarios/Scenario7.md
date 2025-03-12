Your team is using Jenkins for CI/CD, and you notice that a specific job sometimes fails due to a dependency conflict, but rerunning the job without any code changes sometimes makes it pass.
- What could be the possible reasons for this intermittent failure?
- How would you go about debugging and permanently fixing it?

### Solution ###
- The issue described relates to CI/CD dependency resolution and build consistency.
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
