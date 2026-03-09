**You have a Jenkins pipeline that runs for 45 minutes. Your team complains it's too slow. How do you optimize and speed it up?**
- First step is always to identify the bottleneck — check which stage takes longest, then optimize that first. Don't optimize what's already fast!
- Run stages in Parallel:
```groovy
pipeline {
  agent any
  stages {
    stage('Parallel Tests') {
      parallel {
        stage('Unit Tests') {
          steps { sh 'mvn test' }
        }
        stage('Code Analysis') {
          steps { sh 'sonar-scanner' }
        }
        stage('Security Scan') {
          steps { sh 'trivy image myapp' }
        }
      }
    }
  }
}
```
- Cache Dependencies:
```groovy
// Without cache — downloads dependencies every time
// With cache — reuse from previous build

stage('Build') {
  steps {
    // Mount maven cache from agent
    sh 'mvn clean package -Dmaven.repo.local=/cache/.m2'
  }
}
```
- Use Docker Layer Caching:
```groovy
# Put dependencies first — they change less often
COPY pom.xml .
RUN mvn dependency:resolve  # cached if pom.xml unchanged

COPY src/ .
RUN mvn package
```
- Only run expensive stages when needed:
```groovy
stage('Integration Tests') {
  // Only run on main branch — not every feature branch
  when {
    branch 'main'
  }
  steps {
    sh 'mvn integration-test'
  }
}
```
- Use faster agents:
  - Use agents with more CPU/RAM
  - Use spot instances for build agents
  - Use dedicated build agents — not shared ones

<img width="561" height="242" alt="image" src="https://github.com/user-attachments/assets/c20baec7-af53-478b-a3c0-b8ba4e3c4252" />
