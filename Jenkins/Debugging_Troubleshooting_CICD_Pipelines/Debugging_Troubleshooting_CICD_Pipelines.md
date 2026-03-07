**How do you handle failed builds in a CI/CD pipeline?**

First thing always — read the logs:
```bash
# In Jenkins UI
# Open the failed build → Click "Console Output"
# Read the exact error message
```

Then check:
- Agent status
  - Is the agent online and connected?
  - Sufficient disk/CPU/memory
  - Dependencies installed

Recent code changes
- What changed in the last commit? `git log --oneline -5`

Credentials & Secrets
- Did any token/password expire?
- JFrog credentials, AWS keys, Git tokens

Jenkins itself
- Did any Jenkins plugin get updated?
- Is the JDK/Maven/Node version still correct on agent?
