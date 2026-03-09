**Jenkins Credentials Store Types**
- Jenkins has a built-in credentials store where you save secrets securely so pipelines can use them without hardcoding.
- `Jenkins Dashboard → Manage Jenkins → Credentials`
- Two Scopes:
  ```bash
  Global Scope    → available to ALL jobs across Jenkins
  Project Scope   → available only to a specific job/folder
  ```
- Type 1 — Username & Password
  - Used for: Docker registry login, Nexus, Artifactory, any HTTP basic auth
    ```bash
    ID:       dockerhub-creds
    Username: myuser
    Password: mypassword123
    ```
  - In Pipeline:
    ```groovy
    withCredentials([usernamePassword(
        credentialsId: 'dockerhub-creds',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASS'
    )]) {
        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
    }
    ```
- Type 2 — SSH Username with Private Key
  - Used for: Git clone via SSH, SSH into servers, Ansible
    ```bash
    ID:          github-ssh-key
    Username:    git
    Private Key: -----BEGIN RSA PRIVATE KEY-----
    ```
  - In pipeline:
    ```groovy
    withCredentials([sshUserPrivateKey(
        credentialsId: 'github-ssh-key',
        keyFileVariable: 'SSH_KEY'
    )]) {
        sh 'ssh -i $SSH_KEY user@myserver.com'
    }
    ```
- Type 3 — Secret Text
  - Used for: API tokens, single secret values (Slack token, SonarQube token)
    ```bash
    ID:     sonar-token
    Secret: abc123xyz-sonar-token-value
    ```
  - In pipeline:
    ```groovy
    withCredentials([string(
        credentialsId: 'sonar-token',
        variable: 'SONAR_TOKEN'
    )]) {
        sh 'sonar-scanner -Dsonar.login=$SONAR_TOKEN'
    }
    ```
- Type 4 — Secret File
  - Used for: kubeconfig files, .env files, JSON key files (GCP service account)
    ```bash
    ID:   gcp-service-account
    File: service-account.json
    ```
  - In pipeline:
    ```groovy
    withCredentials([file(
        credentialsId: 'gcp-service-account',
        variable: 'GCP_KEY_FILE'
    )]) {
        sh 'gcloud auth activate-service-account --key-file=$GCP_KEY_FILE'
    }
    ```
- Type 5 — Certificate (PKCS#12)
  - Used for: Client certificate authentication, mutual TLS
    ```bash
    ID:          my-cert
    Certificate: .p12 file
    Password:    certpassword
    ```

**Using Credentials in Pipelines (withCredentials)**
- `withCredentials` is the safest way to use secrets in Jenkins pipelines. It:
  - Injects secrets only inside its block
  - Automatically masks values in logs
  - Cleans up after block exits
```groovy
withCredentials([ /* binding type */ ]) {
    // secrets available here as env variables
    // automatically masked in logs
} // secrets gone after this
```
- Real World Full Pipeline Example:
```groovy
pipeline {
    agent any

    stages {
        stage('Build & Push Docker Image') {
            steps {
                // using username/password
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        docker build -t myapp:${BUILD_NUMBER} .
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
                        docker push myapp:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                // using secret file (kubeconfig)
                withCredentials([file(
                    credentialsId: 'kubeconfig-prod',
                    variable: 'KUBECONFIG'
                )]) {
                    sh 'kubectl apply -f k8s/ --kubeconfig=$KUBECONFIG'
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                // using secret text
                withCredentials([string(
                    credentialsId: 'sonar-token',
                    variable: 'SONAR_TOKEN'
                )]) {
                    sh 'sonar-scanner -Dsonar.login=$SONAR_TOKEN'
                }
            }
        }
    }
}
```
- Multiple Credentials at Once:
```groovy
withCredentials([
    usernamePassword(credentialsId: 'db-creds',
        usernameVariable: 'DB_USER',
        passwordVariable: 'DB_PASS'),
    string(credentialsId: 'api-token',
        variable: 'API_TOKEN'),
    file(credentialsId: 'kubeconfig',
        variable: 'KUBECONFIG')
]) {
    sh '''
        ./deploy.sh \
          --db-user $DB_USER \
          --db-pass $DB_PASS \
          --token $API_TOKEN
    '''
}
```

**AWS Secrets Manager with Jenkins**
- It is an AWS service where you store, rotate and manage secrets centrally.
```bash
Instead of storing secrets in Jenkins credentials store
        ↓
You store them in AWS Secrets Manager
        ↓
Jenkins fetches them at runtime when pipeline runs
        ↓
Secret never lives permanently in Jenkins
```
- How Jenkins Authenticates to AWS Secrets Manager
  - Method 1 — IAM Role on EC2 (Best for Jenkins on EC2): If Jenkins master runs on an EC2 instance, attach an IAM role directly to that EC2.
    ```bash
    Jenkins runs on EC2
            ↓
    EC2 has an IAM Role attached
            ↓
    IAM Role has permission to access Secrets Manager
            ↓
    Jenkins pipeline calls AWS SDK / CLI
            ↓
    AWS automatically uses the EC2 role — no keys needed
    ```
  - Method 2 — IRSA (Best for Jenkins on Kubernetes/EKS): If Jenkins runs on EKS, use IAM Roles for Service Accounts (IRSA). The Jenkins pod's Kubernetes service account is linked to an IAM role.
    ```bash
    Jenkins pod runs on EKS
            ↓
    Pod has a K8s service account
            ↓
    Service account is annotated with IAM role ARN
            ↓
    AWS automatically trusts the pod's identity
            ↓
    Pipeline can call Secrets Manager — no keys needed
    ```
    ```yaml
    # Jenkins service account with IRSA annotation
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: jenkins
      annotations:
        eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/jenkins-role
    ```
  - Method 3 — AWS Access Keys stored in Jenkins (Least preferred): Store AWS access key + secret key as Jenkins credentials and pass them to the pipeline.
    ```bash
    AWS IAM user created with Secrets Manager permissions
            ↓
    Access Key ID + Secret Access Key generated
            ↓
    Stored in Jenkins as Username/Password credential
            ↓
    Pipeline exports them as AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
            ↓
    AWS CLI uses them to fetch secrets
    ```
    - Why this is not preferred:
      - Long-lived static keys = Security risk
      - If Jenkins is compromised → attacker has AWS access
      - Keys need Manual rotation
    ```groovy
    withCredentials([usernamePassword(
        credentialsId: 'aws-creds',
        usernameVariable: 'AWS_ACCESS_KEY_ID',
        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
    )]) {
        sh '''
            aws secretsmanager get-secret-value \
                --secret-id myapp/db/password \
                --region us-east-1
        '''
    }
    ```
- Fetching Secrets in Jenkins Pipeline:
  - Using AWS CLI directly:
    ```groovy
    pipeline {
        agent any
    
        stages {
            stage('Fetch secret and use it') {
                steps {
                    script {
                        // fetch the secret value from AWS
                        def dbPassword = sh(
                            script: '''
                                aws secretsmanager get-secret-value \
                                    --secret-id myapp/prod/db-password \
                                    --region us-east-1 \
                                    --query SecretString \
                                    --output text
                            ''',
                            returnStdout: true
                        ).trim()
    
                        // use it — but never echo it!
                        sh "psql postgresql://admin:${dbPassword}@mydb.rds.amazonaws.com/myapp"
                    }
                }
            }
        }
    }
    ```
  - Using the *AWS Secrets Manager Credentials Provider* Plugin for Jenkins:
    ```groovy
    withCredentials([string(
        credentialsId: 'myapp/prod/api-token',  // this is the AWS secret name
        variable: 'API_TOKEN'
    )]) {
        sh 'curl -H "Authorization: Bearer $API_TOKEN" https://api.example.com'
    }
    ```

**Vault Integration with Jenkins**
- HashiCorp Vault is an external secrets manager. Instead of storing secrets in Jenkins, you store them in Vault and Jenkins fetches them at runtime.
- Why Vault over Jenkins credentials store?
  - Jenkins Credentials Store:
    - Secrets stored in Jenkins itself
    - If Jenkins is compromised, all secrets exposed
    - No secret rotation
    - No audit trail of who used which secret
  - HashiCorp Vault:
    - Secrets stored externally and securely
    - Dynamic secrets (auto-rotate)
    - Full audit trail
    - Secrets expire automatically
    - One place for ALL tools (Jenkins, ArgoCD, apps)
- How it works:
```bash
Pipeline starts
     ↓
Jenkins authenticates to Vault
(using AppRole or K8s auth)
     ↓
Vault returns a short-lived token
     ↓
Jenkins fetches secret using token
     ↓
Secret injected into pipeline
     ↓
Token expires after job finishes
```
- Setup Flow:
  - Install "HashiCorp Vault" plugin in Jenkins
  - Configure Vault URL in Jenkins settings
  - Store AppRole credentials in Jenkins (role-id + secret-id to authenticate to Vault)
  - Use vault() step in pipeline
```groovy
pipeline {
    agent any

    environment {
        // tell Jenkins where Vault is
        VAULT_URL = 'https://vault.mycompany.com'
    }

    stages {
        stage('Fetch secrets from Vault') {
            steps {
                withVault(
                    vaultSecrets: [
                        [
                            path: 'secret/myapp/db',       // Vault path
                            secretValues: [
                                [envVar: 'DB_USER', vaultKey: 'username'],
                                [envVar: 'DB_PASS', vaultKey: 'password']
                            ]
                        ],
                        [
                            path: 'secret/myapp/api',
                            secretValues: [
                                [envVar: 'API_KEY', vaultKey: 'token']
                            ]
                        ]
                    ]
                ) {
                    sh './deploy.sh'  // DB_USER, DB_PASS, API_KEY available here
                }
            }
        }
    }
}
```

**Vault Auth Methods with Jenkins:**
- AppRole: Jenkins uses role-id + secret-id. Most common
- Token: Static Vault token in Jenkins creds. Simple setups
- Kubernetes: Jenkins pod uses K8s service account. Jenkins on K8s
- LDAP: Vault authenticates via LDAP. Enterprise

**Masking Secrets in Logs**
- Jenkins automatically masks secrets used via `withCredentials` — replaces them with `****` in console output.
- How masking works:
  ```groovy
  withCredentials([string(credentialsId: 'api-token', variable: 'TOKEN')]) {
      sh 'curl -H "Authorization: Bearer $TOKEN" https://api.example.com'
  }
  
  // Console output:
  // curl -H "Authorization: Bearer ****" https://api.example.com
  ```
- Masking breaks in these situations — be careful
  - Problem 1 — Base64 encoded secrets:
    ```groovy
    // Secret = "mypassword"
    // base64 = "bXlwYXNzd29yZA=="
    // Jenkins only masks "mypassword", NOT the base64 version
    
    sh 'echo $PASSWORD | base64'
    // Output: bXlwYXNzd29yZA== 
    ```
  - Problem 2 — Printing character by character:
    ```groovy
    // If you loop through secret and print each char
    // masking won't work
    ```
  - Problem 3 — Storing secret in a file then cat-ing it:
    ```groovy
    sh 'cat $SECRET_FILE'
    ```
  - Problem 4 — Accidental echo:
    ```groovy
    // Never do this
    sh 'echo "The password is: $DB_PASS"'
    ```
