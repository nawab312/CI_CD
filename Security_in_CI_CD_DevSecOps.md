**Trivy**

Trivy is an Open-Source Container Image scanner that detects security vulnerabilities in Docker images before they are deployed. Trivy scans container images for:
- OS package vulnerabilities (Debian, Ubuntu, RHEL, etc.)
- Programming language dependencies (Python, Node.js, Java, Go, etc.)
- Exploitable Common Vulnerabilities and Exposures (CVEs)
- Misconfigurations (for Kubernetes and Docker files)
It fetches the latest Vulnerability Database and compares the software versions inside the container image against known vulnerabilities.

Example
```bash
trivy image ubuntu:20.04
```
```bash
2025-03-25T12:00:00.123Z  INFO  Vulnerability scanning in progress...

ubuntu:20.04 (ubuntu 20.04)
=====================================
Total: 3 (CRITICAL: 1, HIGH: 2, MEDIUM: 1, LOW: 0)

┌──────────────┬───────────┬──────────────┬────────────────┬──────────────────┬─────────┐
│ CVE ID       │ Package   │ Version      │ Severity       │ Fixed Version    │ Status  │
├──────────────┼───────────┼──────────────┼────────────────┼──────────────────┼─────────┤
│ CVE-2024-5678│ libc6     │ 2.31-0ubuntu9│ CRITICAL       │ 2.31-0ubuntu9.9  │ Needs Fix │
│ CVE-2023-4321│ openssl   │ 1.1.1f       │ HIGH           │ 1.1.1n           │ Needs Fix │
│ CVE-2023-8765│ libcurl   │ 7.68.0       │ HIGH           │ 7.78.0           │ Needs Fix │
│ CVE-2023-1234│ tar       │ 1.30+dfsg-7  │ MEDIUM         │ 1.34+dfsg-1      │ Needs Fix │
└──────────────┴───────────┴──────────────┴────────────────┴──────────────────┴─────────┘
```
- Key Insights from the Report:
  - `libc6` has a CRITICAL vulnerability (fix available in version `2.31-0ubuntu9.9`).
  - `openssl` and `libcurl` have HIGH severity vulnerabilities.
  - The `tar` package has a MEDIUM severity issue.
 
- Trivy found CRITICAL or HIGH vulnerabilities and exited with a failure, causing subsequent stages (like Archive Security Report) to be skipped.
  - `--exit-code 1` forces Jenkins to fail if any CRITICAL or HIGH vulnerabilities are found.
```groovy
pipeline {
    agent any
    environment {
        IMAGE_NAME = "sample-app"
        IMAGE_TAG = "latest"
        REGISTRY = "sid3121997"
        SCAN_RESULT = "trivy-report.json"
    }

    stages {
        stage('Scan Image with Trivy') {
            steps {
                script {
                    sh """
                    trivy image --exit-code 1 --severity CRITICAL --format json \
                    -o ${SCAN_RESULT} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
        stage('Archive Security Report') {
            steps {
                archiveArtifacts artifacts: "${SCAN_RESULT}", fingerprint: true
            }
        }
    }
}
``` 
