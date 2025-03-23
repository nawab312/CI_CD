Your Jenkins pipeline is failing with the error:
```bash
Permission denied (publickey).  
fatal: Could not read from remote repository.  
```
This happens when Jenkins tries to pull code from a private GitHub repository. What steps will you take to resolve this issue?

**SOLUTION**
- Verify SSH Key on Jenkins Server
  - Log in to the Jenkins server: `ssh jenkins@your-server`
  - Check if an SSH key exists: `ls -la ~/.ssh/`
  - If no key exists, generate a new one: `ssh-keygen -t rsa -b 4096 -C "jenkins@example.com"`
  - Save the key in `~/.ssh/id_rsa`
- Add SSH Key to GitHub
  - Copy the public key: `cat ~/.ssh/id_rsa.pub`
  - Go to GitHub → Settings → SSH and GPG keys → New SSH Key
  - Add the copied public key and save it
- Test SSH Connection from Jenkins
  - Run the following command to ensure Jenkins can authenticate with GitHub: `ssh -T git@github.com`
  - Expected output:
    ```bash
    Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
    ```
- Configure SSH Agent in Jenkins
  - Open Jenkins Dashboard → Manage Jenkins → Manage Credentials
  - Under Global credentials, add:
    - Kind: SSH Username with Private Key
    - Username: `git`
    - Private Key: Enter the contents of `~/.ssh/id_rsa`

