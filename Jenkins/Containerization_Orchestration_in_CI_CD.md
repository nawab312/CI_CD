### Deploying to a Minikube Cluster Using Jenkins ###
**kubectl Configured for Minikube**
```bash
kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
- `kubectl` must be installed on the Jenkins Agent

**A Kubernetes ServiceAccount with RBAC**
- Create a ServiceAccount for Jenkins:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-sa-role
subjects:
- kind: ServiceAccount
  name: jenkins-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```
- Apply it: `kubectl apply -f jenkins-sa.yaml`
- Get the token: The below command is an alternative command introduced in Kubernetes 1.24+ to create a token directly for a service account.
```bash
kubectl create token jenkins-sa -n default
eyJhbGciOiJSUzI1NiIsImtpZCI6InhqRVdIcFpUaGdnS2JGZm9nY2M3X1FxbU1ZMGpIYzVudklKMmdyY2UzVkUifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQyODExMjAxLCJpYXQiOjE3NDI4MDc2MDEsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiYmFjYzVmNTctZDA2Mi00ZjRhLTk4MWUtYjI2NTZhNDdhMjJhIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImplbmtpbnMtc2EiLCJ1aWQiOiIyOWM1MzczZi1jMTBlLTQyMDUtYjFhOC0yYWQxMTIyOGVhZTIifX0sIm5iZiI6MTc0MjgwNzYwMSwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6amVua2lucy1zYSJ9.ABJ8_XJNfFWrbJKqD0EuneUv411KyavRdiM5NRVAQB9nkbNEx9QIgYYE2fpyGu8DH_h_c6BC6GvumKVFb2qWK-21lz8VpBduqM-DNTkaku6pk9a7rmy_ydtikDh38POX_Sti_6FdpxjAtvF92tsgj4jrieJwULbCCm5xaCba82wA_5IornJUO8bcJUc0YgVU15ulMcQzQNS5pQ6jKnKC0GtlCo-jVs-GwBUd0jTdfl9gJnBdlf8N7yY_T4qvTt6bSqFPoYwa991xVjUwNl2y8O45APonfsaH29N9nl9mHkrZRN4LzXO-r4_AOTU3B7eL-GikRio7215fwDy527XSRQ
```
- There is one more command to get the Token
  ```bash
  kubectl get secret jenkins-sa-token -o jsonpath='{.data.token}' | base64 --decode
  ```
- Store this token in Jenkins Credentials as a Secret Text (`K8S_SA_TOKEN`).

**Jenkins Pipeline to Deploy to Minikube**
```groovy
pipeline {
    agent any
    environment {
        K8S_API_SERVER = "https://$(minikube ip):8443"
        NAMESPACE = "default"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'git@github.com:nawab312/Sample.git'
            }
        }
        stage('Deploy to Minikube') {
            steps {
                withCredentials([
                    string(credentialsId: 'K8S_SA_TOKEN', variable: 'K8S_TOKEN')
                ]) {
                    script {
                        sh """
                        kubectl config set-cluster minikube --server=$K8S_API_SERVER --insecure-skip-tls-verify=true
                        kubectl config set-credentials jenkins-user --token=$K8S_TOKEN
                        kubectl config set-context minikube --cluster=minikube --user=jenkins-user --namespace=$NAMESPACE
                        kubectl config use-context minikube
                        kubectl apply -f deployment.yaml
                        """
                    }
                }
            }
        }
    }
}
```

---

## How Jenkins on EC2 Authenticates to an AWS EKS Cluster ##
In Amazon EKS, **IAM authentication** is used to securely authenticate AWS users, roles, or applications (like Jenkins) with the Kubernetes cluster. However, IAM **only handles authentication, not authorization**. Authorization is managed using **Kubernetes RBAC (Role-Based Access Control)**, which defines what actions are allowed.

The authentication process involves:
- AWS IAM authentication via `aws-iam-authenticator`
- Generating a temporary authentication token (`eks:GetToken`)
- Mapping IAM roles to Kubernetes RBAC using `aws-auth` ConfigMap
- Using `kubectl` to interact with the cluster

### Workflow: How IAM + RBAC Work Together ###
- Jenkins (running on EC2) runs `kubectl apply -f deployment.yaml`.
- AWS IAM validates Jenkins' identity using `aws-iam-authenticator`.
- IAM role (`JenkinsEKSRole`) is mapped to Kubernetes user (`jenkins`) via `aws-auth ConfigMap`.
- Kubernetes RBAC checks if `jenkins` has permission to deploy applications.
- If authorized, Jenkins successfully deploys to EKS.

### IAM Authentication in EKS ###
Unlike a typical Kubernetes cluster where users authenticate using certificates, **EKS relies on AWS IAM and tokens**.
- IAM users or roles do **not** have direct access to Kubernetes.
- Instead, IAM roles are mapped to Kubernetes users via the `aws-auth` ConfigMap.
- AWS provides an authentication mechanism called `aws-iam-authenticator`, which validates IAM identities before granting access.

**Create an IAM Role for Jenkins EC2**
- If Jenkins needs to *deploy applications* to an EKS cluster, it requires additional permissions beyond just authentication. Below is the industry-standard IAM policy that provides Jenkins the necessary permissions for deployment.

![image](https://github.com/user-attachments/assets/a880b69b-e6bf-4adf-b473-d059594d8b9f)

**Attach the Role to Jenkins EC2 Instance**
- Attach the IAM role to the EC2 instance running Jenkins.

**Authentication Flow**
- Request Authentication Token
  - The `kubectl` command triggers a request to authenticate with EKS.
  - `kubectl` runs `aws eks get-token`, which fetches a temporary authentication token.
    ```bash
    aws eks get-token --cluster-name my-eks-cluster
    ```
- IAM Role Validation
  - AWS checks if the user or role has `eks:GetToken` permissions in IAM.
  - If authorized, AWS generates a short-lived authentication token.
- Using the Token in `kubeconfig`
  - The token is stored in the `kubeconfig` file, allowing `kubectl` to use it for cluster access.
  - Example `kubeconfig` entry:
    ```yaml
    users:
    - name: jenkins
      user:
        exec:
          apiVersion: client.authentication.k8s.io/v1beta1
          command: aws
          args:
            - eks
            - get-token
            - --cluster-name
            - my-eks-cluster
    ```

### IAM + Kubernetes RBAC (Authorization) ###
Once authentication is successful, Kubernetes **checks authorization** using **RBAC policies**. This ensures that IAM users/roles can only perform allowed actions.

**How IAM Roles Are Mapped to Kubernetes Users?**
- The `aws-auth` ConfigMap is a critical Kubernetes configuration in Amazon EKS that *maps AWS IAM identities (users, roles, or instance profiles) to Kubernetes RBAC roles and groups*.
- This allows AWS IAM users or roles to authenticate and interact with the Kubernetes cluster.
- ```bash
  kubectl get configmap aws-auth -n kube-system -o yaml
  ```
- Structure of aws-auth ConfigMap
  - `mapRoles` → Maps IAM roles (recommended for Jenkins).
  - `mapUsers` → Maps IAM users (not recommended for automation).
  - `mapAccounts` → Grants cluster access to entire AWS accounts (not commonly used).
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: aws-auth
    namespace: kube-system
  data:
    mapRoles: |
      - rolearn: arn:aws:iam::123456789012:role/JenkinsEKSRole
        username: jenkins
        groups:
          - system:masters
    mapUsers: |
      - userarn: arn:aws:iam::123456789012:user/dev-user
        username: dev-user
        groups:
          - system:masters
    mapAccounts: |
      - "123456789012"
  ```

### Defining Kubernetes RBAC Permissions ###
Once mapped, Kubernetes **RBAC roles** determine what actions Jenkins can perform.

**Create a Kubernetes Role for Jenkins**
- Instead of giving full `system:masters` access, create a restricted role:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    namespace: default
    name: jenkins-deployer
  rules:
  - apiGroups: [""]
    resources: ["pods", "deployments", "services"]
    verbs: ["get", "list", "create", "update", "delete"]
  ```
- Bind the Role to Jenkins IAM Role
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: jenkins-deployer-binding
    namespace: default
  subjects:
  - kind: Group
    name: jenkins-deployer-group
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: jenkins-deployer
    apiGroup: rbac.authorization.k8s.io
  ```
- Modify the aws-auth ConfigMap as follows:
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: aws-auth
    namespace: kube-system
  data:
    mapRoles: |
      - rolearn: arn:aws:iam::123456789012:role/JenkinsEKSRole
        username: jenkins
        groups:
          - jenkins-deployer-group
  ```

- This ensures that *Jenkins (mapped IAM Role)* can only manage deployments, pods, and services in the `default` namespace.

**Jenkins with Kubernetes**

Deploying Jenkins on Kubernetes and running CI/CD pipelines with ephemeral agents is the best practice in modern DevOps. It ensures scalability, resource efficiency, and parallel execution.

Why Run Jenkins on Kubernetes?
- Scalability → Easily scale Jenkins based on demand.
- Resiliency → Kubernetes handles failures, ensuring high availability.
- Ephemeral Agents → Jenkins dynamically provisions temporary pods for CI/CD tasks.
- Cloud-Native CI/CD → Works seamlessly with AWS EKS, GCP GKE, Azure AKS, OpenShift.

Steps to Deploy Jenkins on Kubernetes
- Add the Jenkins Helm Chart Repository
  ```bash
  helm repo add jenkinsci https://charts.jenkins.io
  helm repo update
  ```
- Install Jenkins Using Helm
  ```bash
  helm install jenkins jenkinsci/jenkins -n jenkins --create-namespace
  ```
- Get Admin Password & Access Jenkins
  ```bash
  kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode
  ```
  - Now, access Jenkins using the NodePort service:
    ```bash
    kubectl get svc -n jenkins
    ```
  - ```bash
    http://<JENKINS_NODE_IP>:<PORT>
    ```

Running CI/CD Pipelines in Kubernetes (Jenkins Agents on Kubernetes)

By default, Jenkins Master doesn't run builds. Instead, it uses dynamic Kubernetes agents to execute CI/CD pipelines efficiently. How Kubernetes Agents Work in Jenkins
- Jenkins master runs in a Kubernetes pod.
- When a pipeline starts, Jenkins dynamically provisions a new agent pod.
- The agent pod executes CI/CD tasks (build, test, deploy).
- After completion, the agent pod is destroyed, freeing resources.
- Kubernetes automatically scales Jenkins agents based on workload.

Configuring Jenkins to Use Kubernetes for Dynamic Agents
- Install the Kubernetes Plugin in Jenkins
  - Go to Manage Jenkins → Manage Plugins.
  - Search for "Kubernetes Plugin" and install it.
  - Restart Jenkins.
- Configure Kubernetes Cloud in Jenkins
  - Go to Manage Jenkins → Manage Nodes and Clouds.
  - Click "Configure Clouds" → "Add a new cloud" → Select Kubernetes.
  - Fill in:
    ```bash
    # Kubernetes URL:
    kubectl cluster-info | grep 'Kubernetes control plane' | awk '{print $NF}'
    ```
    ```bash
    # Jenkins URL:
    kubectl get svc -n jenkins | grep jenkins | awk '{print $3}'
    ```
  - Pod Labels: `jenkins-agent` Jenkins Tunnel: `jenkins:50000`

Writing a Jenkins Pipeline for Kubernetes Agents
- Now, let’s write a Jenkinsfile to use ephemeral Kubernetes agents:
```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                app: jenkins-agent
            spec:
              containers:
              - name: maven
                image: maven:3.8.5-jdk-11
                command: ["/bin/sh", "-c", "cat"]
                tty: true
              - name: node
                image: node:18
                command: ["/bin/sh", "-c", "cat"]
                tty: true
            '''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Test') {
            steps {
                container('node') {
                    sh 'npm test'
                }
            }
        }
    }
}
```

*In a Jenkins pipeline deploying a containerized application to a Minikube cluster, following  ensures that the application deployment is updated with the latest image while avoiding cached versions*
```bash
kubectl rollout restart deployment <deployment-name>
```

    
