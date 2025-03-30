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
    
