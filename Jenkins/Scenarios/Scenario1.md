You have setup a Jenkins master-slave architecture and you notice that the builds are not being distributed evenly across available slave nodes. WHAT COULD BE ISSUE AND HOW YOU WILL REOLVE THEM. SCENARIO:
- Master Node: JenkinsMaster (Handles job scheduling, manages the Jenkins environment)
- Slave Nodes:
  - SlaveNode1 (labeled linux, with 2 executors).
  - SlaveNode2 (labeled windows, with 2 executors).
  - SlaveNode3 (labeled linux, with 3 executors)

The master node schedules and triggers builds, and the slave nodes are expected to execute them. However, you've noticed that most builds are running on SlaveNode3, and SlaveNode1 and SlaveNode2 are underutilized, resulting in an imbalance.

**Potential Issues and Resolutions:**
- **Inconsistent Node Labels Configuration(Uneven Load Distribution):** If jobs are configured with specific labels (e.g., linux or windows), Jenkins will try to schedule those jobs on the corresponding nodes. If the jobs aren’t labeled correctly or are not being distributed properly, some nodes may end up handling more jobs than others.
For example, if a job is set to run only on linux nodes, it will run on any linux-labeled node (either SlaveNode1 or SlaveNode3). But if SlaveNode3 has more executors, it will handle more jobs than SlaveNode1.
```groovy
pipeline {
    agent none  // This makes sure the pipeline does not run on the master node, but will run on slaves.

    stages {
        stage('Linux Build') {
            agent {
                label 'linux'  // This will run the job on any available node with the 'linux' label
            }
            steps {
                echo 'Building on a Linux node...'
                sh 'echo "Building Linux part of the pipeline"'
            }
        }
        
        stage('Windows Build') {
            agent {
                label 'windows'  // This will run the job on any available node with the 'windows' label
            }
            steps {
                echo 'Building on a Windows node...'
                bat 'echo "Building Windows part of the pipeline"'
            }
        }

        stage('Linux Cleanup') {
            agent {
                label 'linux'  // This runs cleanup on a linux node again, can run on any linux node
            }
            steps {
                echo 'Cleaning up on a Linux node...'
                sh 'echo "Cleaning up Linux part of the pipeline"'
            }
        }
    }
}
```
  - If SlaveNode3 has more executors (3) than SlaveNode1 (which only has 2), JobA (which runs on linux nodes) will end up mostly running on SlaveNode3, as it has more executors. This results in an uneven load across the available linux nodes. How to Resolve the Issue:
    - *Adjust the Number of Executors on Nodes:*
      - Go to Manage Jenkins > Manage Nodes and Clouds.
      - Adjust the number of executors on SlaveNode3 (for example, reduce from 3 to 2) to balance the load more evenly with SlaveNode1.
    - *Ensure Proper Load Distribution with a Throttling Plugin:*
      - Throttle Concurrent Builds Plugin in Jenkins is used to control the number of concurrent builds running on specific nodes. Limits how many builds can run concurrently on a node or across nodes with specific labels. Let’s say you want to throttle the builds to allow a maximum of 1 concurrent build per node (even though SlaveNode1 has 2 executors and SlaveNode3 has 3 executors).
```groovy
pipeline {
    agent none  // Do not run the pipeline on the master node.

    stages {
        stage('Linux Build') {
            agent { label 'linux' }  // Runs on a node with 'linux' label
            options {
                throttleJobProperty(
                    categories: ['linux-builds'],
                    maxConcurrentPerNode: 1,  // Only 1 concurrent build per node
                    maxConcurrentTotal: 2,    // Allow 2 total concurrent builds across all nodes
                    throttleEnabled: true,
                    throttleOption: 'project'
                )
            }
            steps {
                echo 'Building on a Linux node...'
                sh 'echo "Building Linux part of the pipeline"'
            }
        }

        stage('Windows Build') {
            agent { label 'windows' }  // Runs on a node with 'windows' label
            options {
                throttleJobProperty(
                    categories: ['windows-builds'],
                    maxConcurrentPerNode: 1,  // Only 1 concurrent build per node
                    maxConcurrentTotal: 2,    // Allow 2 total concurrent builds across all nodes
                    throttleEnabled: true,
                    throttleOption: 'project'
                )
            }
            steps {
                echo 'Building on a Windows node...'
                bat 'echo "Building Windows part of the pipeline"'
            }
        }
    }
}
```      
    
