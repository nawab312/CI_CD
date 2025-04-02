**Secrets Management (Vault, AWS Secrets Manager, Jenkins Credentials)**

Managing secrets securely is critical in CI/CD pipelines to prevent unauthorized access, data breaches, and accidental exposure of sensitive information. 
In a CI/CD environment, secrets include API keys, database credentials, SSH keys, and cloud access tokens, which must be stored securely and retrieved dynamically during pipeline execution.


![image](https://github.com/user-attachments/assets/7068e95a-2f68-4096-ae04-fd1166ec2b1e)

*Jenkins Credentials Store*

Jenkins provides a built-in credentials store to securely store and manage secrets used in pipelines.
- Go to Jenkins Dashboard → Manage Jenkins → Manage Credentials.
- Select Global credentials (or another appropriate scope).
- Click Add Credentials and enter:
  - Type: Secret text, username/password, or a file.
  - ID: Unique identifier (e.g., `MY_SECRET`).
```groovy
pipeline {
    agent any
    environment {
        API_KEY = credentials('MY_SECRET') // Retrieves secret securely
    }
    stages {
        stage('Deploy') {
            steps {
                withCredentials([string(credentialsId: 'MY_SECRET', variable: 'SECRET')]) {
                    sh 'echo "Using secret safely without exposing it in logs"'
                }
            }
        }
    }
}
```

*AWS Secrets Manager*

AWS Secrets Manager securely stores, retrieves, and rotates secrets such as database passwords, API keys, and encryption keys.
- Secrets are encrypted and stored securely in AWS.
- IAM roles ensure only authorized users/services can access them.
- Automatic secret rotation for credentials like RDS passwords.

How to Use AWS Secrets Manager in a CI/CD Pipeline
- Go to AWS Console → Secrets Manager → Store a New Secret.
- Choose a secret type (Key-Value, RDS credentials, etc.).
- Assign a name (e.g., `/dev/api_key`).
- Configure IAM permissions to allow Jenkins to access secrets.
```groovy
pipeline {
    agent any
    stages {
        stage('Fetch Secrets') {
            steps {
                script {
                    def secretValue = sh(script: "aws secretsmanager get-secret-value --secret-id /dev/api_key --query SecretString --output text", returnStdout: true).trim()
                    env.API_KEY = secretValue
                }
                sh 'echo "Using secret securely in pipeline..."'
            }
        }
    }
}
```

*HashiCorp Vault*

HashiCorp Vault is a dynamic secrets management tool designed for multi-cloud and on-prem environments.
- Secrets are not stored in CI/CD systems—retrieved dynamically.
- Support for automatic expiration and revocation of secrets.
- Supports AWS, Kubernetes, databases, and more.

How to Integrate HashiCorp Vault in CI/CD Pipelines
- Deploy Vault server and enable secrets engine (e.g., `kv-v2`).
- Create a secret at `secret/data/my-app`:
```json
{
    "api_key": "my-secure-api-key"
}
```
- Set up authentication for Jenkins (e.g., AppRole, Token).
```groovy
withVault(configuration: [vaultUrl: 'https://vault.example.com'], secrets: [
    [path: 'secret/data/my-app', engineVersion: 2, secretValues: [
        [envVar: 'API_KEY', vaultKey: 'api_key']
    ]]
]) {
    sh 'echo "Using API Key securely..."'
}
```

**DevSecOps in Jenkins**

DevSecOps stands for "Development, Security, and Operations," emphasizing the integration of security practices into the CI/CD pipeline. 
In the context of Jenkins, DevSecOps means incorporating security measures at every stage of the software development and deployment lifecycle. 
The goal is to automate security testing and monitoring within the pipeline to detect and mitigate security vulnerabilities earlier in the process.

This is How Devops Flow look Like

![image](https://github.com/user-attachments/assets/3328918b-66c3-49fe-b8e2-9f68df4dab53)

This is How DevSecops Flow look Like

![image](https://github.com/user-attachments/assets/1865e459-3e47-494b-8723-61dd0d703086)

![image](https://github.com/user-attachments/assets/798b521f-d1aa-4a92-b26c-b6ab53f780cc)

![image](https://github.com/user-attachments/assets/1a4ee66f-5203-45d3-81d9-69ee577d8331)

*Network Vulnerability Assessment (NVA)*
- Jenkins Integration: Use network security scanning tools like Rapid7 Nexpose, OpenVAS, or Tenable Nessus in Jenkins pipelines to detect vulnerabilities in network configurations.
- Example: Automate scanning of infrastructure components like load balancers, firewalls, and cloud networking configurations for misconfigurations or security weaknesses.

*Cloud Vulnerability Assessment (CVA)*
- Jenkins Integration: Use CloudHealth by VMware, AWS Security Hub, or Prowler to check cloud security best practices.
- Example: Automate cloud security posture management (CSPM) scans in Jenkins for AWS, Azure, or GCP deployments.

*Container Vulnerability Assessment (KVA)*
- Jenkins Integration: Automate container scanning using Trivy, Clair, or Amazon ECR scanning before pushing images to registries.
- Example: Run security checks on Docker images and Kubernetes deployments to detect vulnerabilities in containerized applications.

*Static Code Analysis (SVA)*
- Jenkins Integration: Perform Static Application Security Testing (SAST) using Fortify, SonarQube, or Checkmarx in Jenkins.
- Example: Scan source code during Jenkins builds to find hardcoded secrets, insecure coding patterns, and vulnerabilities.

*Software Composition Analysis (SCA)*
- Jenkins Integration: Use WhiteSource, OWASP Dependency-Check, or Snyk for third-party dependency scanning.
- Example: Jenkins can automatically check for license compliance and known CVEs in open-source libraries.

*Dynamic Vulnerability Assessment (DVA)*
- Jenkins Integration: Run Dynamic Application Security Testing (DAST) with OWASP ZAP, Burp Suite, or Micro Focus Fortify.
- Example: Jenkins can trigger automated penetration tests on running applications to simulate real-world attack scenarios.

*Penetration Testing (PEN)*
- Jenkins Integration: Automate pentesting tools like Metasploit, Kali Linux scripts, or OWASP ZAP to simulate attacks.
- Example: Jenkins can execute scheduled penetration tests on applications and APIs in a staging environment before deployment.

---

**Software Supply Chain Security in CI/CD**

Software Supply Chain Security ensures that every component in your CI/CD pipeline—from dependencies to builds and deployments—is secure, verified, and free from vulnerabilities. 
It prevents supply chain attacks, where attackers compromise open-source dependencies, build tools, or deployment pipelines.

Key Components of Supply Chain Security in CI/CD
- SBOM (Software Bill of Materials)
- Signing Artifacts & Images

*SBOM (Software Bill of Materials)*

A Software Bill of Materials (SBOM) is a detailed inventory of all the components in a software application, including:
- Source Code (Custom-built code)
- Libraries & Dependencies (Open-source & third-party)
- Build Tools (Maven, Gradle, NPM, etc.)
- Container Images
- Licensing & Compliance Details

Why is SBOM Important in CI/CD?
- Detect Vulnerabilities: Helps in identifying security risks in dependencies before deployment.
- Compliance & Auditing: Required for standards like NIST, ISO 27001, SOC 2, and PCI-DSS.
- Prevents Supply Chain Attacks: Ensures you’re not using compromised third-party components.
- Software Provenance: Tracks where each component originated from and whether it’s trusted.

How to Generate an SBOM in Jenkins?
- Tool: CycloneDX (Industry-standard SBOM format)
- Tool: SPDX (Software Package Data Exchange)

```groovy
// Example: Using CycloneDX to Generate SBOM in a Jenkins Pipeline

pipeline {
    agent any
    stages {
        stage('Generate SBOM') {
            steps {
                script {
                    sh "mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom"
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/bom.json', fingerprint: true
        }
    }
}
```

*Signing Artifacts & Images*

Signing ensures that build artifacts (JARs, WARs, binaries, container images, etc.) are trusted and untampered throughout the CI/CD pipeline. It guarantees that the software being deployed was not altered by attackers.
- Artifact Signing = Protects software packages (JARs, Docker images, etc.).
- Container Image Signing = Ensures Docker images are secure before deployment.

Signing Artifacts in Jenkins (JAR/WAR Packages)
- Tool: GPG (GNU Privacy Guard) for Code Signing
- Tool: Cosign (by Sigstore) for Container Image Signing

```groovy
// Example: Signing JAR Files in Jenkins using GPG

pipeline {
    agent any
    environment {
        GPG_KEY_ID = 'YOUR-GPG-KEY-ID'
    }
    stages {
        stage('Build Artifact') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Sign Artifact') {
            steps {
                sh "gpg --batch --yes --detach-sign --armor -u ${GPG_KEY_ID} target/myapp.jar"
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/myapp.jar.asc', fingerprint: true
        }
    }
}
```

```groovy
// Example: Signing JAR Files in Jenkins using GPG

pipeline {
    agent any
    environment {
        COSIGN_PASSWORD = credentials('cosign-key-password')  // Store safely in Jenkins credentials
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }
        stage('Sign Image') {
            steps {
                sh 'cosign sign --key cosign.key myapp:latest'
            }
        }
    }
}
```





