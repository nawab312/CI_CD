There are Four different layers where env vars can exist in Jenkins.

### OS-Level Environment Variables (On the Slave/Agent Machine) ###
- They belong to the operating system process environment. Any process started on that machine may inherit them: Jenkins agent, Manual SSH sessions, Other services. Key Characteristics
  - Exist even if Jenkins is not installed
  - Affect everything on that machine
  - Not visible inside Jenkins UI
- Configured in: `/etc/environment`, `~/.bashrc`, `~/.profile`, `/etc/profile`, Systemd Service Files (If agent runs as Service)
- ```bash
  export JAVA_HOME=/usr/lib/jvm/java-17
  export PATH=$JAVA_HOME/bin:$PATH
  ```
- If the Jenkins agent runs as a *systemd* service, it does NOT load `.bashrc`. You may need to configure:
  ```bash
  sudo systemctl edit jenkins-agent

  # Or Define env variables in
  /etc/systemd/system/jenkins-agent.service
  ```
- Then:
  ```bash
  systemctl daemon-reload
  systemctl restart jenkins-agent
  ```

### Jenkins Node-Level Environment Variables ###
- They are injected by Jenkins into build processes launched on that node. Key Characteristics:
  - Only available inside Jenkins builds
  - Do NOT exist at OS level. Do NOT affect other system processes
  - Managed inside Jenkins UI
  - These are injected at build runtime
- Configured in: *Manage Jenkins → Nodes → Select Node → Configure → Node Properties → Environment Variables*
- These apply only to that specific agent. Use this when:
  - Different slaves have different JDK versions
  - Different tool paths per node

### Global Jenkins Environment Variables ###
- Configured in: *Manage Jenkins → Configure System → Global properties → Environment variables*
- These apply to all nodes.
- These are injected at build runtime. They don’t affect the agent OS itself

### Pipeline-Level Environment Variables ###
- Defined in Jenkinsfile
  ```jenkinsfile
    pipeline {
    agent any
    environment {
      JAVA_HOME = '/usr/lib/jvm/java-17'
    }
  }
  ```
- Or Inside Stage:
  ```jenkinsfile
  withEnv(["JAVA_HOME=/usr/lib/jvm/java-17"]) {
    sh 'echo $JAVA_HOME'
  }
  ```


At runtime, Jenkins builds an environment map for the process it launches. If the same variable exists in multiple layers, the value defined closest to the execution context wins.
- OS-level environment (Inherited by agent process)
- Global Jenkins environment variables
- Node-Level Environment Variables
- Pipeline environment {} block
- withEnv() block (Most Specific /Highest Priority)
