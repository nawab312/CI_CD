```jenkinsfile
// ============================================================================
//  INDUSTRY-GRADE CI/CD PIPELINE — payment-service
//  Covers: Build → Test → Security Scan → Artifact Promotion →
//          Binary Signing → Deploy Dev → Deploy Staging → Deploy Prod
//
//  Tools used:
//    - Jenkins (pipeline engine)
//    - Maven   (build tool — Java)
//    - Docker  (containerization)
//    - JFrog Artifactory (artifact repository + promotion)
//    - Trivy   (Docker image vulnerability scan)
//    - Cosign  (binary signing)
//    - SonarQube (code quality)
//    - Kubernetes (deployment target)
//    - Slack   (notifications)
// ============================================================================

pipeline {

    // ── Where to run ─────────────────────────────────────────────────────────
    // Runs on any Jenkins agent that has Docker + Maven + kubectl installed
    agent {
        kubernetes {
            label 'jenkins-agent'
            yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: maven
                    image: maven:3.9-eclipse-temurin-17
                    command: ['sleep', '99d']
                  - name: docker
                    image: docker:24-dind
                    securityContext:
                      privileged: true
                  - name: tools
                    image: alpine/curl:latest
                    command: ['sleep', '99d']
            """
        }
    }

    // ── Environment Variables ─────────────────────────────────────────────────
    // These are global across all stages
    environment {

        // ── App Info ──────────────────────────────────────────────────────────
        APP_NAME        = "payment-service"
        APP_VERSION     = "1.2.${BUILD_NUMBER}"   // e.g. 1.2.42

        // ── Docker / Registry ─────────────────────────────────────────────────
        DOCKER_REGISTRY = "mycompany.jfrog.io"
        IMAGE_DEV       = "${DOCKER_REGISTRY}/docker-dev/${APP_NAME}:${APP_VERSION}"
        IMAGE_STAGING   = "${DOCKER_REGISTRY}/docker-staging/${APP_NAME}:${APP_VERSION}"
        IMAGE_PROD      = "${DOCKER_REGISTRY}/docker-prod/${APP_NAME}:${APP_VERSION}"

        // ── JFrog Artifactory Repos ───────────────────────────────────────────
        JFROG_URL       = "https://mycompany.jfrog.io/artifactory"
        REPO_DEV        = "libs-dev-local"
        REPO_STAGING    = "libs-staging-local"
        REPO_PROD       = "libs-release-local"

        // ── Kubernetes Namespaces ─────────────────────────────────────────────
        K8S_NS_DEV      = "dev"
        K8S_NS_STAGING  = "staging"
        K8S_NS_PROD     = "production"

        // ── SonarQube ─────────────────────────────────────────────────────────
        SONAR_URL       = "http://sonarqube.company.internal:9000"
        SONAR_PROJECT   = "payment-service"

        // ── Credentials (stored securely in Jenkins Credential Store) ─────────
        // NEVER hardcode passwords — always use Jenkins credentials binding
        JFROG_CREDS     = credentials('jfrog-credentials')       // username/password
        SONAR_TOKEN     = credentials('sonarqube-token')         // secret text
        K8S_KUBECONFIG  = credentials('kubeconfig-all-envs')     // secret file
        COSIGN_KEY      = credentials('cosign-private-key')      // secret file
        SLACK_WEBHOOK   = credentials('slack-webhook-url')       // secret text
        DOCKER_CREDS    = credentials('docker-registry-creds')   // username/password

        // ── Thresholds ────────────────────────────────────────────────────────
        COVERAGE_THRESHOLD = "80"    // Minimum code coverage % required
        CRITICAL_CVE_LIMIT = "0"     // Zero critical CVEs allowed
    }

    // ── Pipeline Options ──────────────────────────────────────────────────────
    options {
        timeout(time: 60, unit: 'MINUTES')       // Kill pipeline if it runs > 60 min
        disableConcurrentBuilds()                 // Don't run 2 builds of same branch at once
        buildDiscarder(logRotator(numToKeepStr: '10'))  // Keep only last 10 builds
        timestamps()                              // Add timestamps to all log lines
    }

    // ── Triggers ──────────────────────────────────────────────────────────────
    triggers {
        // Auto-trigger on Git push (webhook from GitHub/GitLab calls this)
        githubPush()

        // Also run a full pipeline every night at 2am (nightly build)
        cron('H 2 * * *')
    }

    // ══════════════════════════════════════════════════════════════════════════
    //  STAGES
    // ══════════════════════════════════════════════════════════════════════════
    stages {

        // ── STAGE 1: Checkout ─────────────────────────────────────────────────
        // Pull source code from Git
        stage('Checkout') {
            steps {
                echo "━━━ STAGE 1: Checking out source code ━━━"
                checkout scm   // Checks out the branch that triggered this build

                // Save the Git commit hash — used in artifact metadata later
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                    env.GIT_AUTHOR = sh(
                        script: "git log -1 --pretty=format:'%an'",
                        returnStdout: true
                    ).trim()
                    echo "Branch:  ${env.BRANCH_NAME}"
                    echo "Commit:  ${env.GIT_COMMIT_SHORT}"
                    echo "Author:  ${env.GIT_AUTHOR}"
                }
            }
        }

        // ── STAGE 2: Build ────────────────────────────────────────────────────
        // Compile the code and produce a JAR artifact
        stage('Build') {
            steps {
                echo "━━━ STAGE 2: Building application ━━━"
                container('maven') {
                    sh '''
                        # Clean previous build output, then compile + package
                        # -DskipTests = skip tests here (we run them separately next)
                        mvn clean package -DskipTests \
                            -Dapp.version=${APP_VERSION} \
                            -Dgit.commit=${GIT_COMMIT_SHORT}

                        echo "Build complete. Artifact:"
                        ls -lh target/*.jar
                    '''
                }
            }
            // Save the JAR so later stages can use it
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        // ── STAGE 3: Unit Tests ───────────────────────────────────────────────
        // Run all unit tests and check code coverage
        stage('Unit Tests') {
            steps {
                echo "━━━ STAGE 3: Running unit tests ━━━"
                container('maven') {
                    sh '''
                        # Run tests + generate coverage report (JaCoCo)
                        mvn test jacoco:report

                        echo "Test results:"
                        cat target/surefire-reports/TEST-*.xml | grep -E "tests=|failures=|errors="
                    '''
                }
            }
            post {
                always {
                    // Publish test results in Jenkins UI (pass or fail)
                    junit 'target/surefire-reports/*.xml'

                    // Publish coverage report in Jenkins UI
                    jacoco(
                        execPattern: 'target/jacoco.exec',
                        classPattern: 'target/classes',
                        sourcePattern: 'src/main/java'
                    )
                }
                failure {
                    // Unit tests failed — notify team immediately
                    slackSend(
                        channel: '#ci-alerts',
                        color: 'danger',
                        message: "❌ Unit tests FAILED — ${APP_NAME} v${APP_VERSION}\nBranch: ${BRANCH_NAME}\nAuthor: ${GIT_AUTHOR}\nBuild: ${BUILD_URL}"
                    )
                }
            }
        }

        // ── STAGE 4: Code Quality Gate (SonarQube) ────────────────────────────
        // Static code analysis — catches bugs, code smells, security hotspots
        stage('Code Quality — SonarQube') {
            steps {
                echo "━━━ STAGE 4: SonarQube code analysis ━━━"
                container('maven') {
                    sh '''
                        mvn sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT} \
                            -Dsonar.host.url=${SONAR_URL} \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
            post {
                always {
                    // Check if SonarQube Quality Gate passed
                    // This waits up to 5 minutes for SonarQube to process results
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                        // abortPipeline: true = if quality gate fails, stop the whole pipeline
                    }
                }
            }
        }

        // ── STAGE 5: Code Coverage Gate ───────────────────────────────────────
        // Enforce minimum coverage threshold — blocks promotion if too low
        stage('Coverage Gate') {
            steps {
                echo "━━━ STAGE 5: Checking code coverage threshold ━━━"
                container('maven') {
                    script {
                        // Read coverage % from JaCoCo XML report
                        def coverage = sh(
                            script: """
                                python3 -c "
                                import xml.etree.ElementTree as ET
                                tree = ET.parse('target/site/jacoco/jacoco.xml')
                                root = tree.getroot()
                                for counter in root.findall('counter'):
                                    if counter.get('type') == 'LINE':
                                        covered = int(counter.get('covered'))
                                        missed  = int(counter.get('missed'))
                                        pct = (covered / (covered + missed)) * 100
                                        print(f'{pct:.1f}')
                                "
                            """,
                            returnStdout: true
                        ).trim().toFloat()

                        echo "Code coverage: ${coverage}%"
                        echo "Required minimum: ${COVERAGE_THRESHOLD}%"

                        // QUALITY GATE — block if coverage is below threshold
                        if (coverage < COVERAGE_THRESHOLD.toFloat()) {
                            error("❌ Coverage gate FAILED: ${coverage}% < ${COVERAGE_THRESHOLD}% required. Add more tests!")
                        }
                        echo "✅ Coverage gate PASSED: ${coverage}%"
                    }
                }
            }
        }

        // ── STAGE 6: Build Docker Image ───────────────────────────────────────
        // Package the JAR into a Docker image — the artifact we will promote
        stage('Build Docker Image') {
            steps {
                echo "━━━ STAGE 6: Building Docker image ━━━"
                container('docker') {
                    sh '''
                        # Login to JFrog Docker registry
                        echo ${DOCKER_CREDS_PSW} | docker login ${DOCKER_REGISTRY} \
                            -u ${DOCKER_CREDS_USR} --password-stdin

                        # Build the image — tag it for the DEV registry first
                        docker build \
                            --build-arg APP_VERSION=${APP_VERSION} \
                            --build-arg GIT_COMMIT=${GIT_COMMIT_SHORT} \
                            --build-arg BUILD_DATE=$(date -u +%Y-%m-%dT%H:%M:%SZ) \
                            --label "org.opencontainers.image.version=${APP_VERSION}" \
                            --label "org.opencontainers.image.revision=${GIT_COMMIT_SHORT}" \
                            --label "org.opencontainers.image.created=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
                            -t ${IMAGE_DEV} \
                            .

                        echo "Image built: ${IMAGE_DEV}"
                        docker images | grep ${APP_NAME}
                    '''
                }
            }
        }

        // ── STAGE 7: Security Scan (Trivy) ────────────────────────────────────
        // Scan Docker image for known vulnerabilities (CVEs)
        // CRITICAL CVEs = hard block. HIGH CVEs = warning.
        stage('Security Scan — Trivy') {
            steps {
                echo "━━━ STAGE 7: Trivy vulnerability scan ━━━"
                container('docker') {
                    script {
                        // Run Trivy scan and output results as JSON
                        sh '''
                            # Install Trivy (or use pre-installed on agent)
                            curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

                            # Scan the image
                            # --exit-code 1 = exit with error if CRITICAL found
                            # --severity CRITICAL,HIGH = only report these levels
                            trivy image \
                                --exit-code 0 \
                                --severity CRITICAL,HIGH \
                                --format json \
                                --output trivy-report.json \
                                ${IMAGE_DEV}

                            # Also print human-readable table to console
                            trivy image \
                                --severity CRITICAL,HIGH \
                                --format table \
                                ${IMAGE_DEV}
                        '''

                        // Parse results and enforce zero CRITICAL CVE policy
                        def trivyReport = readJSON file: 'trivy-report.json'
                        def criticalCount = 0
                        def highCount = 0

                        trivyReport.Results?.each { result ->
                            result.Vulnerabilities?.each { vuln ->
                                if (vuln.Severity == 'CRITICAL') criticalCount++
                                if (vuln.Severity == 'HIGH')     highCount++
                            }
                        }

                        echo "CRITICAL CVEs found: ${criticalCount}"
                        echo "HIGH CVEs found:     ${highCount}"

                        // QUALITY GATE — zero critical CVEs allowed
                        if (criticalCount > CRITICAL_CVE_LIMIT.toInteger()) {
                            error("❌ Security gate FAILED: ${criticalCount} CRITICAL vulnerabilities found. Fix them before promoting!")
                        }
                        echo "✅ Security gate PASSED"
                    }
                }
            }
            post {
                always {
                    // Archive Trivy report for audit purposes
                    archiveArtifacts artifacts: 'trivy-report.json', allowEmptyArchive: true
                }
            }
        }

        // ── STAGE 8: Push to DEV Registry ─────────────────────────────────────
        // All gates passed — push image to the DEV artifact repository
        stage('Push to DEV Registry') {
            steps {
                echo "━━━ STAGE 8: Pushing image to DEV registry ━━━"
                container('docker') {
                    sh '''
                        docker push ${IMAGE_DEV}
                        echo "✅ Pushed: ${IMAGE_DEV}"

                        # Capture the SHA digest — used for signing
                        docker inspect --format='{{index .RepoDigests 0}}' ${IMAGE_DEV}
                    '''

                    script {
                        // Save SHA digest to env — this is the TRUE identity of our image
                        env.IMAGE_SHA = sh(
                            script: "docker inspect --format='{{index .Id}}' ${IMAGE_DEV}",
                            returnStdout: true
                        ).trim()
                        echo "Image SHA: ${env.IMAGE_SHA}"
                    }
                }
            }
        }

        // ── STAGE 9: Binary Signing (Cosign) ──────────────────────────────────
        // Sign the Docker image with the company private key
        // This proves: "our official CI pipeline built this — it's genuine"
        stage('Binary Signing — Cosign') {
            steps {
                echo "━━━ STAGE 9: Signing image with Cosign ━━━"
                container('tools') {
                    sh '''
                        # Install Cosign
                        curl -Lo /usr/local/bin/cosign \
                            https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64
                        chmod +x /usr/local/bin/cosign

                        # Sign the image using the company private key
                        # The signature is stored IN the registry alongside the image
                        cosign sign \
                            --key ${COSIGN_KEY} \
                            --annotations "pipeline=jenkins" \
                            --annotations "build=${BUILD_NUMBER}" \
                            --annotations "git-commit=${GIT_COMMIT_SHORT}" \
                            --annotations "git-author=${GIT_AUTHOR}" \
                            ${IMAGE_DEV}

                        echo "✅ Image signed successfully"

                        # Verify the signature immediately to confirm it worked
                        cosign verify \
                            --key ${COSIGN_KEY}.pub \
                            ${IMAGE_DEV}

                        echo "✅ Signature verified"
                    '''
                }
            }
        }

        // ── STAGE 10: Promote JAR to JFrog Artifactory ────────────────────────
        // This promotes the JAR artifact from libs-dev-local to libs-staging-local
        // The SAME file moves — it is NOT rebuilt
        stage('Promote JAR — DEV to STAGING') {
            steps {
                echo "━━━ STAGE 10: Promoting JAR artifact in JFrog ━━━"
                container('tools') {
                    sh '''
                        # Install JFrog CLI
                        curl -fL https://install-cli.jfrog.io | sh
                        mv jf /usr/local/bin/jf

                        # Configure JFrog CLI with server credentials
                        jf config add myserver \
                            --url=${JFROG_URL} \
                            --user=${JFROG_CREDS_USR} \
                            --password=${JFROG_CREDS_PSW} \
                            --interactive=false

                        # PROMOTE the build from DEV repo → STAGING repo
                        # This is an API call — the binary file does NOT move physically
                        # Only the metadata pointer changes in JFrog
                        jf rt build-promote ${APP_NAME} ${APP_VERSION} \
                            ${REPO_STAGING} \
                            --source-repo=${REPO_DEV} \
                            --status="staging-candidate" \
                            --comment="Promoted by Jenkins build #${BUILD_NUMBER} — all quality gates passed" \
                            --copy=true

                        echo "✅ JAR promoted: ${REPO_DEV} → ${REPO_STAGING}"
                    '''
                }
            }
        }

        // ── STAGE 11: Promote Docker Image — DEV to STAGING ──────────────────
        // Retag the same Docker image for the staging registry
        stage('Promote Docker — DEV to STAGING') {
            steps {
                echo "━━━ STAGE 11: Promoting Docker image to STAGING ━━━"
                container('docker') {
                    sh '''
                        # Pull from dev registry
                        docker pull ${IMAGE_DEV}

                        # Retag for staging registry (same image, different registry path)
                        docker tag ${IMAGE_DEV} ${IMAGE_STAGING}

                        # Push to staging registry
                        docker push ${IMAGE_STAGING}

                        echo "✅ Docker promoted: docker-dev → docker-staging"
                        echo "   DEV:     ${IMAGE_DEV}"
                        echo "   STAGING: ${IMAGE_STAGING}"

                        # Prove it's the same image (SHA should be identical)
                        echo "DEV SHA:"
                        docker inspect --format='{{.Id}}' ${IMAGE_DEV}
                        echo "STAGING SHA:"
                        docker inspect --format='{{.Id}}' ${IMAGE_STAGING}
                    '''
                }
            }
        }

        // ── STAGE 12: Deploy to DEV Environment ──────────────────────────────
        // Deploy to Kubernetes DEV namespace
        stage('Deploy — DEV') {
            steps {
                echo "━━━ STAGE 12: Deploying to DEV environment ━━━"
                container('tools') {
                    sh '''
                        # Install kubectl
                        curl -LO "https://dl.k8s.io/release/stable.txt"
                        curl -LO "https://dl.k8s.io/release/$(cat stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl && mv kubectl /usr/local/bin/

                        export KUBECONFIG=${K8S_KUBECONFIG}

                        # Verify signature BEFORE deploying (security check)
                        cosign verify --key ${COSIGN_KEY}.pub ${IMAGE_DEV}
                        echo "✅ Signature verified before DEV deploy"

                        # Update the Kubernetes deployment to use new image
                        kubectl set image deployment/${APP_NAME} \
                            ${APP_NAME}=${IMAGE_DEV} \
                            -n ${K8S_NS_DEV}

                        # Wait for rollout to complete (max 5 minutes)
                        kubectl rollout status deployment/${APP_NAME} \
                            -n ${K8S_NS_DEV} \
                            --timeout=5m

                        echo "✅ Deployed to DEV: ${IMAGE_DEV}"
                    '''
                }
            }
        }

        // ── STAGE 13: Integration Tests (against DEV) ─────────────────────────
        // Run full integration test suite against the deployed DEV environment
        stage('Integration Tests — DEV') {
            steps {
                echo "━━━ STAGE 13: Running integration tests on DEV ━━━"
                container('maven') {
                    sh '''
                        # Run integration tests pointing to DEV environment URL
                        mvn verify \
                            -Pintegration-tests \
                            -Dbase.url=https://payment-service.dev.company.internal \
                            -Dtest.timeout=30

                        echo "Integration tests complete"
                    '''
                }
            }
            post {
                always {
                    junit 'target/failsafe-reports/*.xml'
                }
                failure {
                    slackSend(
                        channel: '#ci-alerts',
                        color: 'danger',
                        message: "❌ Integration tests FAILED on DEV\n${APP_NAME} v${APP_VERSION}\nBuild: ${BUILD_URL}"
                    )
                }
            }
        }

        // ── STAGE 14: Deploy to STAGING ───────────────────────────────────────
        // Deploy the staging-promoted image to Kubernetes STAGING namespace
        stage('Deploy — STAGING') {
            steps {
                echo "━━━ STAGE 14: Deploying to STAGING environment ━━━"
                container('tools') {
                    sh '''
                        export KUBECONFIG=${K8S_KUBECONFIG}

                        # Verify signature before staging deploy
                        cosign verify --key ${COSIGN_KEY}.pub ${IMAGE_STAGING}
                        echo "✅ Signature verified before STAGING deploy"

                        kubectl set image deployment/${APP_NAME} \
                            ${APP_NAME}=${IMAGE_STAGING} \
                            -n ${K8S_NS_STAGING}

                        kubectl rollout status deployment/${APP_NAME} \
                            -n ${K8S_NS_STAGING} \
                            --timeout=5m

                        echo "✅ Deployed to STAGING: ${IMAGE_STAGING}"
                    '''
                }
            }
            post {
                success {
                    // Notify QA team that staging is ready for manual testing
                    slackSend(
                        channel: '#qa-team',
                        color: 'good',
                        message: "✅ ${APP_NAME} v${APP_VERSION} deployed to STAGING\nReady for QA testing\nStaging URL: https://payment-service.staging.company.internal\nApprove production deploy here: ${BUILD_URL}input"
                    )
                }
            }
        }

        // ── STAGE 15: Manual Approval Gate ────────────────────────────────────
        // HUMAN decision point before production
        // Pipeline pauses here — waits for QA Lead / Release Manager to approve
        stage('APPROVAL — Production Deploy') {
            steps {
                echo "━━━ STAGE 15: Waiting for production approval ━━━"
                script {
                    // Pipeline pauses here and waits for human input
                    // Timeout: 24 hours — if nobody approves, pipeline aborts
                    timeout(time: 24, unit: 'HOURS') {
                        def approver = input(
                            id: 'ProductionApproval',
                            message: "Deploy ${APP_NAME} v${APP_VERSION} to PRODUCTION?",
                            submitter: 'qa-leads,release-managers',  // Only these roles can approve
                            parameters: [
                                choice(
                                    name: 'ACTION',
                                    choices: ['APPROVE', 'REJECT'],
                                    description: 'Approve or reject this production deployment'
                                ),
                                string(
                                    name: 'APPROVER_NAME',
                                    description: 'Your name (for audit trail)'
                                ),
                                string(
                                    name: 'JIRA_TICKET',
                                    description: 'Change request ticket number (e.g. CHG-1234)'
                                )
                            ]
                        )

                        if (approver['ACTION'] == 'REJECT') {
                            error("🚫 Production deployment REJECTED by ${approver['APPROVER_NAME']}")
                        }

                        // Store approver info for audit trail
                        env.APPROVED_BY    = approver['APPROVER_NAME']
                        env.CHANGE_TICKET  = approver['JIRA_TICKET']
                        echo "✅ Approved by: ${env.APPROVED_BY} | Ticket: ${env.CHANGE_TICKET}"
                    }
                }
            }
        }

        // ── STAGE 16: Promote JAR to PRODUCTION in JFrog ─────────────────────
        stage('Promote JAR — STAGING to PRODUCTION') {
            steps {
                echo "━━━ STAGE 16: Promoting JAR to PRODUCTION in JFrog ━━━"
                container('tools') {
                    sh '''
                        jf rt build-promote ${APP_NAME} ${APP_VERSION} \
                            ${REPO_PROD} \
                            --source-repo=${REPO_STAGING} \
                            --status="release" \
                            --comment="PRODUCTION release. Approved by: ${APPROVED_BY}. Ticket: ${CHANGE_TICKET}" \
                            --copy=true

                        echo "✅ JAR promoted: ${REPO_STAGING} → ${REPO_PROD}"
                    '''
                }
            }
        }

        // ── STAGE 17: Promote Docker Image — STAGING to PRODUCTION ───────────
        stage('Promote Docker — STAGING to PRODUCTION') {
            steps {
                echo "━━━ STAGE 17: Promoting Docker image to PRODUCTION registry ━━━"
                container('docker') {
                    sh '''
                        docker pull ${IMAGE_STAGING}
                        docker tag  ${IMAGE_STAGING} ${IMAGE_PROD}
                        docker push ${IMAGE_PROD}

                        echo "✅ Docker promoted: docker-staging → docker-prod"
                        echo "   PROD: ${IMAGE_PROD}"
                    '''
                }
            }
        }

        // ── STAGE 18: Deploy to PRODUCTION (Canary first) ────────────────────
        // In real industry: deploy to 10% of traffic first (canary)
        // Monitor for 10 minutes, then roll out to 100%
        stage('Deploy — PRODUCTION') {
            steps {
                echo "━━━ STAGE 18: Deploying to PRODUCTION ━━━"
                container('tools') {
                    sh '''
                        export KUBECONFIG=${K8S_KUBECONFIG}

                        # CRITICAL: Verify signature one final time before prod deploy
                        cosign verify --key ${COSIGN_KEY}.pub ${IMAGE_PROD}
                        echo "✅ Signature verified before PRODUCTION deploy"

                        # ── Step 1: Canary deploy (10% traffic) ──────────────
                        # Deploy new version to a small subset of pods first
                        kubectl set image deployment/${APP_NAME}-canary \
                            ${APP_NAME}=${IMAGE_PROD} \
                            -n ${K8S_NS_PROD}

                        kubectl rollout status deployment/${APP_NAME}-canary \
                            -n ${K8S_NS_PROD} \
                            --timeout=5m

                        echo "Canary deployed. Monitoring for 10 minutes..."
                        sleep 600   # Wait 10 minutes — monitor error rates in Datadog/Grafana

                        # ── Step 2: Check canary health ───────────────────────
                        # Query metrics API — if error rate > 1%, abort and rollback
                        ERROR_RATE=$(curl -s "http://metrics.company.internal/api/error-rate?service=${APP_NAME}&env=canary" | jq '.rate')
                        echo "Canary error rate: ${ERROR_RATE}%"

                        if (( $(echo "${ERROR_RATE} > 1.0" | bc -l) )); then
                            echo "❌ Canary error rate too high! Rolling back..."
                            kubectl rollout undo deployment/${APP_NAME}-canary -n ${K8S_NS_PROD}
                            exit 1
                        fi

                        echo "✅ Canary healthy. Proceeding with full rollout..."

                        # ── Step 3: Full production rollout (100% traffic) ────
                        kubectl set image deployment/${APP_NAME} \
                            ${APP_NAME}=${IMAGE_PROD} \
                            -n ${K8S_NS_PROD}

                        kubectl rollout status deployment/${APP_NAME} \
                            -n ${K8S_NS_PROD} \
                            --timeout=10m

                        echo "✅ PRODUCTION deployment complete!"
                        echo "   Image: ${IMAGE_PROD}"
                        echo "   Approved by: ${APPROVED_BY}"
                        echo "   Change ticket: ${CHANGE_TICKET}"
                    '''
                }
            }
        }

    }
    // ══════════════════════════════════════════════════════════════════════════
    // END OF STAGES
    // ══════════════════════════════════════════════════════════════════════════


    // ── Post Actions — run after ALL stages complete ──────────────────────────
    post {

        success {
            echo "🎉 Pipeline completed successfully!"
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: """
                    ✅ *PRODUCTION DEPLOY SUCCESS*
                    App:     ${APP_NAME} v${APP_VERSION}
                    Commit:  ${GIT_COMMIT_SHORT} by ${GIT_AUTHOR}
                    Approved by: ${env.APPROVED_BY ?: 'N/A'}
                    Ticket:  ${env.CHANGE_TICKET ?: 'N/A'}
                    Build:   ${BUILD_URL}
                """
            )
        }

        failure {
            echo "💥 Pipeline FAILED"
            slackSend(
                channel: '#ci-alerts',
                color: 'danger',
                message: """
                    ❌ *PIPELINE FAILED*
                    App:    ${APP_NAME} v${APP_VERSION}
                    Branch: ${BRANCH_NAME}
                    Author: ${GIT_AUTHOR}
                    Stage:  ${env.STAGE_NAME}
                    Build:  ${BUILD_URL}
                """
            )
        }

        aborted {
            echo "⏹️ Pipeline was aborted (likely approval timeout or manual cancel)"
            slackSend(
                channel: '#deployments',
                color: 'warning',
                message: "⏹️ Pipeline ABORTED — ${APP_NAME} v${APP_VERSION} | ${BUILD_URL}"
            )
        }

        always {
            // Always clean up workspace to free disk space on Jenkins agent
            cleanWs()
        }
    }

}

// ============================================================================
// DOCKERFILE (referenced in Stage 6 — BUILD DOCKER IMAGE)
// Place this in the root of your repo alongside the Jenkinsfile
// ============================================================================
//
// FROM eclipse-temurin:17-jre-alpine
//
// # Non-root user for security — never run as root in production
// RUN addgroup -S appgroup && adduser -S appuser -G appgroup
//
// WORKDIR /app
//
// # Copy the JAR built by Maven
// COPY target/payment-service-*.jar app.jar
//
// # Switch to non-root user
// USER appuser
//
// # Accept build args and embed as image labels
// ARG APP_VERSION
// ARG GIT_COMMIT
// ARG BUILD_DATE
//
// LABEL org.opencontainers.image.version=${APP_VERSION}
// LABEL org.opencontainers.image.revision=${GIT_COMMIT}
// LABEL org.opencontainers.image.created=${BUILD_DATE}
//
// EXPOSE 8080
//
// ENTRYPOINT ["java", \
//     "-XX:+UseContainerSupport", \
//     "-XX:MaxRAMPercentage=75.0", \
//     "-jar", "app.jar"]
//
// ============================================================================
```
