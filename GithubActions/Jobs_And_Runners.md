<img width="546" height="240" alt="image" src="https://github.com/user-attachments/assets/801633f1-c005-476e-a641-688eea2a6fb8" />

**What is a Runner?**
- A runner is the machine that executes your job. When a workflow triggers, GitHub spins up a fresh virtual machine, runs your job, and destroys it. GitHub provides these out of the box:

  <img width="669" height="158" alt="image" src="https://github.com/user-attachments/assets/d088e5f0-ce57-45c3-9a75-582613f6e487" />
- Always prefer a specific version like ubuntu-22.04 over ubuntu-latest in production — latest can change and break your pipeline unexpectedly. Same lesson as Jenkins agent labels!

