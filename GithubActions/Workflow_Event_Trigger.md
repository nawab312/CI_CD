**Workflow File Basics**
- Every workflow is a YAML file inside your repo:
```yaml
.github/
  workflows/
    ci.yml
    deploy.yml
    nightly.yml
```
- Each file = One independent workflow
- They all show up separately in the Actions tab on GitHub
- A repo can have Unlimited Workflows

**The `on` Block — Event Triggers**
- This is the heart of every workflow. It answers: "when should this run?"
- `push` — On Code Push
  ```yaml
  on:
    push:
      branches:
        - main
        - dev
      paths:
        - 'src/**'       # only trigger if files in src/ changed
        - '**.js'        # any .js file changed
  ```
- `pull_request` — On PR Activity
  ```yaml
  on:
    pull_request:
      branches:
        - main           # PR targeting main
      types:
        - opened         # PR opened
        - synchronize    # new commit pushed to PR
        - reopened       # PR reopened
  ```
- `workflow_dispatch` — Manual Trigger. Adds a "Run workflow" button in the GitHub UI. You can even pass inputs:
  ```yaml
  on:
    workflow_dispatch:
      inputs:
        environment:
          description: 'Deploy to which env?'
          required: true
          default: 'staging'
          type: choice
          options:
            - staging
            - production
        debug:
          description: 'Enable debug mode?'
          type: boolean
          default: false
  ```
- `schedule` — Cron Trigger
  ```yaml
  on:
    schedule:
      - cron: '0 2 * * 1'   # every Monday at 2 AM UTC
      - cron: '0 8 * * *'   # every day at 8 AM UTC
  ```
- `release` — On GitHub Release. GitHub Actions cron is always UTC
  ```yaml
  on:
    release:
      types:
        - published      # when a release is published
        - created
  ```
- `workflow_call` — Called by Another Workflow. Makes this workflow reusable — another workflow can call it like a function:
  ```yaml
  on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
  ```
- Combining Multiple Triggers. This workflow runs on push to main, on any PR to main, manually, AND nightly. All in one file.
  ```yaml
  on:
    push:
      branches: [main]
    pull_request:
      branches: [main]
    workflow_dispatch:
    schedule:
      - cron: '0 0 * * *'
  ```
- Filtering — branches, paths, tags
  ```yaml
  on:
    push:
      branches-ignore:
        - 'release/**'   # don't run on release branches
      paths-ignore:
        - '**.md'        # ignore markdown file changes
        - 'docs/**'      # ignore docs folder
      tags:
        - 'v*'           # run only when a version tag is pushed
  ```
