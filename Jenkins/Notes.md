- **Jenkins** is an open-source automation server written in Java. It helps automate the parts of software development related to building, testing, and deploying applications, making the process faster and more reliable
- Jenkins supports **distributed builds** by using multiple machines (agents/nodes) to execute jobs, which improves performance and enables scaling.
  - Master Node (Controller): Handles scheduling, user interactions, and job configurations. Does not execute builds by default
  - Agent Node (Worker): A separate machine that connects to the master to execute build tasks.
  - Benefits of Distributed Builds:
    - Parallel execution of jobs.
    - Dedicated environments for specific builds.

![image](https://github.com/user-attachments/assets/94b7ee97-76f1-41ed-afb9-a4a1aabbd9fa)

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




