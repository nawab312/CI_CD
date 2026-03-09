```bash
You write source code (Java, Python, Node.js etc.)
        ↓
Jenkins builds it
        ↓
Output is an ARTIFACT — the thing that gets deployed

Examples:
  Java app    → myapp.jar or myapp.war
  Node app    → build/ folder (minified JS, HTML, CSS)
  Python app  → .whl file or Docker image
  Any app     → Docker image (most common today)
```

### Archiving Artifacts in Jenkins ###
Jenkins can save build outputs so you can download them later directly from the Jenkins UI. This is called archiving.

**Basic Artifact Archiving**
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
                // produces target/myapp-1.0.jar
            }
        }
    }

    post {
        success {
            // save the jar file in Jenkins
            archiveArtifacts artifacts: 'target/*.jar',
                             fingerprint: true
        }
    }
}
```
- After the build, Jenkins UI shows:
```bash
Build #42
  Build Successful
  Artifacts:
    myapp-1.0.jar    (download link)
```

**Archiving Multiple File Types**
```groovy
post {
    always {
        archiveArtifacts artifacts: [
            'target/*.jar',           // Java jar
            'target/*.war',           // Java war
            'reports/**/*.html',      // test reports
            'logs/*.log'              // build logs
        ].join(', '),
        fingerprint: true,
        allowEmptyArchive: true       // don't fail if no files found
    }
}
```

**What fingerprint does**
- Jenkins calculates MD5 hash of the artifact
- Tracks which build produced which artifact
- Tracks which jobs used which artifact
- Useful for tracing "which build is deployed right now?"

**Limitations of Jenkins Artifact Archiving**
```bash
Jenkins stores artifacts on its own disk
        ↓
Jenkins disk fills up over time 
Artifacts deleted when build is deleted 
Cannot share artifacts between Jenkins instances 
Not suitable for large artifacts 

This is why real teams use external artifact managers: Nexus, Artifactory, or S3
```

---
---

### Integration with Nexus, Artifactory, S3 ###
These are dedicated artifact storage systems built for storing and managing build outputs properly.

**Nexus Repository Manager**
- Nexus is the most common artifact manager in Java/Maven shops. What Nexus stores: Maven JARs and WARs, npm packages, Docker images, Python packages (PyPI), Raw files (zip, tar etc.)
```groovy
pipeline {
    agent any

    environment {
        NEXUS_URL     = 'https://nexus.mycompany.com'
        NEXUS_REPO    = 'maven-releases'
        GROUP_ID      = 'com.mycompany'
        ARTIFACT_ID   = 'myapp'
        VERSION       = '1.0.${BUILD_NUMBER}'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Push to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    // Maven deploy directly to Nexus
                    sh '''
                        mvn deploy \
                            -DaltDeploymentRepository=${NEXUS_REPO}::default::${NEXUS_URL}/repository/${NEXUS_REPO} \
                            -Dusername=$NEXUS_USER \
                            -Dpassword=$NEXUS_PASS
                    '''
                }
            }
        }
    }
}
```
- What happens in Nexus:
```bash
Nexus stores:
  com/mycompany/myapp/1.0.42/myapp-1.0.42.jar
  com/mycompany/myapp/1.0.42/myapp-1.0.42.pom

Other teams/pipelines can now pull this artifact:
  mvn dependency:get -Dartifact=com.mycompany:myapp:1.0.42
```

**JFrog Artifactory**
- Artifactory is similar to Nexus but more feature-rich. Very common in enterprise setups.
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Push to Artifactory') {
            steps {
                // using JFrog Jenkins plugin
                rtMavenDeployer(
                    id: 'maven-deployer',
                    serverId: 'artifactory-server',    // configured in Jenkins
                    releaseRepo: 'libs-release-local',
                    snapshotRepo: 'libs-snapshot-local'
                )

                rtMavenRun(
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: 'maven-deployer'
                )

                rtPublishBuildInfo(
                    serverId: 'artifactory-server'
                )
            }
        }
    }
}
```
- What Artifactory gives you over Nexus:
  - Build info tracking → Which source commit produced which artifact
  - Artifact promotion → Move artifact from dev repo → staging → prod repo
  - Xray scanning → Security vulnerability scanning of artifacts
 
**Artifact Promotion in Artifactory**
- Artifact promotion is the practice of building a deployable artifact exactly once, storing it in a versioned repository, and then moving (promoting) that same artifact through environments — Dev → Staging → Production — rather than rebuilding at each stage. *Only the config changes at each environment — never the artifact itself*

How It Works Technically — JAR / WAR
- Developer pushes code
  - git push origin main → CI pipeline (Jenkins / GitHub Actions) wakes up
  - The pipeline is triggered automatically on every push.
- CI builds artifact ONCE
  - mvn build → payment-service-v1.2.3.jar
  - Metadata is stamped: Git commit hash, build time, who triggered, test results.
- Artifact pushed to Repository
  - artifactory.company.com/libs-dev/payment-service/v1.2.3/payment-service-v1.2.3.jar
- Deploy to DEV + automated tests
  - Pipeline pulls v1.2.3 from repo and deploys to DEV server. Unit tests, API tests, smoke tests run.
- Quality Gate check
  - Unit tests ■ | Security scan ■ | Coverage >80% ■ | QA approval ■
  - All gates must pass. Only then does promotion happen.
- Promote to STAGING
  - Same v1.2.3.jar deployed to staging. Config changes (DB URL = staging DB). Binary is identical.
  - In JFrog, this means moving the artifact from libs-dev-local → libs-staging-local.
- Promote to PRODUCTION
  - Same v1.2.3.jar. Config now points to production DB & services.
```bash
libs-dev-local ← artifact lands here first (after build)
libs-staging-local ← promoted here after dev tests pass
libs-release-local ← promoted here for production use
BEFORE: payment-service-v1.2.3.jar status: 'dev-tested'
AFTER: payment-service-v1.2.3.jar status: 'release-ready' (same file!)
```

Docker Image Promotion — Modern Microservices
```bash
registry.company.com / payment-service : v1.2.3
        ↑                    ↑              ↑
    Where stored         App name     Version tag
# AWS: 123456.dkr.ecr.ap-south-1.amazonaws.com/payment-service:v1.2.3
# GCP: gcr.io/mycompany/payment-service:v1.2.3
# JFrog: mycompany.jfrog.io/docker/payment-service:v1.2.3
```
- SHA Digest — The True Identity
  - Every Docker image gets a SHA256 digest automatically. Even if someone changes the tag (e.g. v1.2.3 → latest), the SHA never changes. This is how you prove it's the exact same image throughout the entire promotion chain.
- Build image once: `docker build -t payment-service:v1.2.3 .`
- Push to dev registry: `docker push mycompany.jfrog.io/docker-dev/payment-service:v1.2.3`
- Security scan: Trivy / Snyk scans image layers for CVEs. CRITICAL/HIGH vulnerabilities block promotion.
- Deploy to DEV + test: Kubernetes pulls image. Unit, API, integration tests run automatically.
- Retag & push to staging: docker tag ...docker-dev/... ...docker-staging/... docker push ...docker-staging/...
- Deploy to STAGING: Same image. Config changes via env vars / ConfigMaps. QA tests & sign-off.
- Retag & push to prod: docker tag ...docker-staging/... ...docker-prod/... docker push ...docker-prod/...
- Deploy to PRODUCTION

Promotion is almost always automated — the CI/CD pipeline calls the JFrog API or Docker registry API to move the artifact. Humans are only involved as an approval gate before production, clicking approve in a tool like Spinnaker or Jenkins. Nobody manually copies files in JFrog day-to-day

**AWS S3 for Artifact Storage**
- S3 is the simplest option — just store build outputs as files in an S3 bucket.
- Good for:
  - Non-Java projects (Python, Node, shell scripts)
  - Storing raw zip/tar files
  - Terraform state files
  - Static website builds
```groovy
pipeline {
    agent any

    environment {
        S3_BUCKET = 'mycompany-artifacts'
        APP_NAME  = 'myapp'
    }

    stages {
        stage('Build') {
            steps {
                sh 'npm run build'
                // produces build/ folder
                sh "zip -r ${APP_NAME}-${BUILD_NUMBER}.zip build/"
            }
        }

        stage('Push to S3') {
            steps {
                // Jenkins runs on EC2 with IAM role — no keys needed
                sh '''
                    aws s3 cp ${APP_NAME}-${BUILD_NUMBER}.zip \
                        s3://${S3_BUCKET}/builds/${APP_NAME}/${BUILD_NUMBER}/ \
                        --region us-east-1
                '''

                // also update a "latest" pointer
                sh '''
                    aws s3 cp ${APP_NAME}-${BUILD_NUMBER}.zip \
                        s3://${S3_BUCKET}/builds/${APP_NAME}/latest/${APP_NAME}-latest.zip
                '''
            }
        }
    }
}
```
- Downloading artifact from S3 in deploy pipeline
```groovy
stage('Download from S3') {
    steps {
        sh '''
            aws s3 cp \
                s3://mycompany-artifacts/builds/myapp/${BUILD_NUMBER}/myapp-${BUILD_NUMBER}.zip \
                .
            unzip myapp-${BUILD_NUMBER}.zip
        '''
    }
}
```

---
---

### Docker Image Building and Pushing ###

**Building and Pushing to AWS ECR**
```groovy
pipeline {
    agent any

    environment {
        AWS_REGION  = 'us-east-1'
        AWS_ACCOUNT = '123456789012'
        ECR_REPO    = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_NAME  = "${ECR_REPO}/myapp"
        IMAGE_TAG   = "${BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to ECR') {
            steps {
                // Jenkins EC2 has IAM role — no keys needed
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REPO}

                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }
}
```

---
---

### Stashing and Unstashing Between Stages ###
By default, each stage in Jenkins runs on the same workspace. But when you use parallel stages or different agents, files don't automatically carry over. Stash solves this.

**The Problem**
```bash
Stage 1 (Build)    → runs on Agent A → produces target/myapp.jar
Stage 2 (Test)     → runs on Agent B → needs target/myapp.jar
// But Agent B has no idea what Agent A did
```

**Stash — Save files to Jenkins master**
```groovy
pipeline {
    agent none  // different agents per stage

    stages {
        stage('Build') {
            agent { label 'build-agent' }
            steps {
                sh 'mvn clean package -DskipTests'

                // save the jar file with a name
                stash name: 'app-jar',
                      includes: 'target/*.jar'

                // stash multiple things
                stash name: 'test-configs',
                      includes: 'src/test/resources/**'
            }
        }

        stage('Test') {
            agent { label 'test-agent' }  // different agent
            steps {
                // retrieve the jar from build stage
                unstash 'app-jar'
                unstash 'test-configs'

                // now target/myapp.jar is available here
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            agent { label 'docker-agent' }
            steps {
                // retrieve jar again on docker agent
                unstash 'app-jar'

                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
    }
}
```

**Stash Limitations — Important for Interviews**
- Stash size limit:
  - Default max stash size is 100MB
  - Not suitable for large artifacts
  - Use S3 or Artifactory for large files
- Stash is temporary:
  - Only available during the current build
  - Deleted when build finishes
- Stash goes through Jenkins master:
  - Build agent → uploads to Jenkins master → test agent downloads
  - Can be slow for large files
  - Can put load on Jenkins master
 
**Stash vs Archiving vs External Storage**
- Stash:
  - Temporary, within one build
  - Sharing files between stages/agents
  - Max ~100MB
  - Deleted after build
- Archive Artifacts:
  - Saved in Jenkins permanently (until build deleted)
  - Download from Jenkins UI
  - Small files only
  - Jenkins disk fills up
- Nexus / Artifactory / S3:
  - Permanent external storage
  - Large files fine
  - Share between teams and pipelines
  - Production-grade artifact management
