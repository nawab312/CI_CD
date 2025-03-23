- **Jenkins** is an open-source automation server written in Java. It helps automate the parts of software development related to building, testing, and deploying applications, making the process faster and more reliable
- Jenkins supports **distributed builds** by using multiple machines (agents/nodes) to execute jobs, which improves performance and enables scaling.
  - Master Node (Controller): Handles scheduling, user interactions, and job configurations. Does not execute builds by default
  - Agent Node (Worker): A separate machine that connects to the master to execute build tasks.
  - Benefits of Distributed Builds:
    - Parallel execution of jobs.
    - Dedicated environments for specific builds.

![image](https://github.com/user-attachments/assets/94b7ee97-76f1-41ed-afb9-a4a1aabbd9fa)

- **Deploy Java Code on Tomcat using Master-Slave Architecture**

![image](https://github.com/user-attachments/assets/6db09837-2580-4567-9a7b-c3e9732eb349)


- **Setting up a Node**
  - Navigate to Manage Jenkins > Manage Nodes and Clouds.
  - Click New Node and provide the following details:
    - Node Name: A unique name for the node.
    - Usage: Choose *Only build jobs with label expressions* to restrict specific builds to this node.
    - Launch Method: Launch agent via SSH

    ![image](https://github.com/user-attachments/assets/ea87203d-54b3-44c9-b417-76a94de2b3b8)  ![image](https://github.com/user-attachments/assets/240454c1-b870-4cb9-ae0d-3cdf1a4466c3)

  - Add credentials under the Global scope:
    - Type: SSH Username with Private Key.
    - Username: The SSH user for the agent machine (e.g., `ubuntu`, `jenkins)`.
    - Private Key: Paste the private key for SSH access (e.g., the content of your `~/.ssh/id_rsa file`).
   
- **Builds not being evenly distributed across Jenkins slave nodes, and how can this imbalance be fixed in a Jenkins Master-Slave setup?** https://github.com/nawab312/CI_CD/blob/main/Jenkins/Scenarios/Scenario1.md
- **Builds are failing on the slave nodes.** https://github.com/nawab312/CI_CD/blob/main/Jenkins/Scenarios/Scenario2.md

- **Multiple Labels or Nodes**
```groovy
pipeline {
    agent { label 'python || linux' }  // Run on nodes with either 'python' or 'linux'
    stages {
        stage('Build') {
            steps {
                echo 'Running on a node with either python or linux'
                // Build steps here
            }
        }
    }
}
```

- **Jenkins Jobs**
  - Job is a unit of work that defines a specific build, test, and deployment process.

- **Jenkins Source Code Management (SCM)**
  - Source Code Management (SCM) in Jenkins allows the CI/CD pipeline to pull the latest version of the application code from a version control system (VCS) (Git (GitHub, GitLab, Bitbucket, AWS CodeCommit))
  - In Jenkins Source Code Management (SCM), there are two models to trigger CI/CD pipelines:
    - **Pull-Based Model (Polling SCM)**
      - Jenkins periodically checks the SCM (Git, SVN, etc.) for changes and triggers a build if new commits are detected.
      - Jenkins runs a cron job at a fixed interval to check for updates in the repository.
      - If new commits are found, Jenkins triggers a build.
    - **Push-Based Model (Webhook Triggers)**
      - The SCM (GitHub, GitLab, Bitbucket, etc.) notifies Jenkins immediately when a new commit is pushed.
      - A *Webhook* is configured in the repository (GitHub, GitLab, Bitbucket).
      - When a developer pushes new code, the webhook sends a notification to Jenkins.
      - Jenkins triggers the pipeline instantly.
```groovy
pipeline {
    agent any
    triggers {
        pollSCM('H/5 * * * *')  // Check for new commits every 5 minutes
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repo.git'
            }
        }
    }
}
```

```groovy
pipeline {
    agent any
    triggers {
        githubPush() // Jenkins triggers build when a GitHub webhook is received
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repo.git'
            }
        }
    }
}
```

**Variables in Jenkins**
- **Pipeline-specific variables:**
  - Variables that are defined within a specific pipeline script and can only be accessed within that pipeline. Can be defined using the def keyword or by assigning values directly.
  ```groovy
  pipelines {
      agent any
      environment {
          def myString = "Hello World"
      }
      stages {
          stage("Demo") {
              steps {
                  echo "${myString}"
              }
          }
      }
  }
    ```
- **Global Environment variables:**
  - Predefined variables that provide information about the Jenkins environment, such as BUILD_NUMBER, JENKINS_URL, and WORKSPACE
  ```groovy
  pipeline {
      agent any
      stages {
          stage('Example') {
              steps {
                  echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
              }
          }
      }
  }
  ```

- **Pass Values Dynamically To Jenkins** https://github.com/nawab312/CI_CD/blob/main/Jenkins/Parameters.md
 
- **Create a Pipeline with Parameters** https://github.com/nawab312/CI_CD/blob/main/Jenkins/Scenarios/Scenario4.md
 
- **Input step in a Jenkins pipeline is and how it works?**
  - The input step in Jenkins is used to pause the pipeline execution and wait for human interaction or approval before continuing. This step is commonly used for manual intervention, such as when you want someone to approve the deployment to production or review specific changes before proceeding with further steps.
  - https://github.com/nawab312/CI_CD/blob/main/Jenkins/Jenkinsfile/User-Input/Jenkinsfile-1
  - https://github.com/nawab312/CI_CD/blob/main/Jenkins/Jenkinsfile/User-Input/Jenkinsfile-2

- **Using Configuration for each Environment** https://github.com/nawab312/CI_CD/blob/main/Jenkins/Scenarios/Scenario-5.md

- **Secrets Management** https://github.com/nawab312/CI_CD/blob/main/Jenkins/Scenarios/Scenario6.md

- **Jenkins On EC2 Authenticating with EKS** https://github.com/nawab312/CI_CD/blob/main/Jenkins/Scenarios/Jenkins_EKS_Authentication.md

- **Jenkins for Terraform** https://github.com/nawab312/CI_CD/blob/main/Jenkins/Jenkins_Terraform.md




