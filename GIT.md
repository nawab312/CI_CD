**Version Control: Centralized vs Distributed**
- **Centralized Version Control (e.g., SVN, Perforce):**
  - A *single central server* stores the entire codebase and version history (e.g., SVN, Perforce).
  - Developers *checkout* code from the central repository and commit changes directly to it.
  - Requires an *internet connection* to commit changes and view history.
  - *Single point of failure*—if the server is down, no one can access or commit code.
- **Distributed Version Control (e.g., Git):**
  - Each developer has a *complete local copy* of the repository, including history (e.g., Git, Mercurial).
  - Changes can be *committed locally* and later pushed to a remote repository
  - Allows *offline work* and supports *branching/merging* efficiently.
  - *No single point of failure*—code can be restored from any developer’s local copy.
    
![image](https://github.com/user-attachments/assets/201bdf7e-ce01-4bd0-9493-182630a53bbc)

- **git fetch** is a command used to retrieve the latest changes from a *remote repository* without merging them into your local branch.
  - It downloads *new commits, branches, and tags* from the remote repository.
  - It *does not modify* your working directory or merge changes automatically.
  - It allows you to check updates before integrating them using `git merge` or `git rebase`.
  ```bash
  git fetch origin # Fetches updates from the origin remote repository.
  git fetch --all # Fetches updates from all remote repositories.
  ```
  - To update your local branch after fetching, you need:
  ```bash
  git merge origin/main  # Merge fetched changes
  git rebase origin/main  # Rebase on the latest remote changes
  ```

- **git pull** is a command that fetches the latest changes from a remote repository and *merges* them into your current local branch.
  - It is a combination of `git fetch` + `git merge`, so it *downloads and applies* remote changes automatically.
  - Keeps your local branch *up to date* with the remote branch.
  ```bash
  git pull origin main #Fetches and merges changes from the main branch of the origin remote repository.
  ```

- **git clone** is a command used to *copy* a remote Git repository to your local machine.
  - It *downloads* the entire repository, including all branches and commit history.
  - It *automatically sets up a remote connection* (typically named `origin`).
  - Useful for getting a fresh copy of a project for development.
  ```bash
  git clone https://github.com/user/repo.git # This creates a local copy of repo.git.
  git clone https://github.com/user/repo.git my-project # This clones the repository into a folder named my-project.
  ```

![image](https://github.com/user-attachments/assets/66c19dbe-7626-4e59-9fd7-9eae5b597942)

- **git diff** is a Git command used to compare differences between two versions of a file, branch, commit, or the working directory. It helps track changes before committing them
  - Checking Staged Changes:
    - To see differences between the staging area and the last commit (i.e., what is ready to be committed):
      ```bash
      git diff --staged
      git diff --cached
      ```
  - Comparing Two Commits:
    ```bash
    git diff abc123 def456
    ```
  - Comparing a Branch with Another
    ```bash
    git diff main feature-branch
    ```
  - Comparing a File Between Two Commits
    ```bash
    git diff <commit1> <commit2> -- filename.txt
    ```

- **git stash** is a Git command used to temporarily save uncommitted changes in a repository without committing them. This allows you to switch branches or work on something else without losing your modifications. The stashed changes can be reapplied later.
  - To save your uncommitted changes:
    ```bash
    git stash
    ```
  - Listing Stashed Changes
    ```
    git stash list
    ```
  - Applying Stashed Changes
    ```bash
    git stash apply #To apply the most recent stash
    git stash apply stash@{1} #To apply a specific stash
    ```
  - To apply and delete the latest stash:
    ```bash
    git stash pop
    ```
  - Use Cases of git stash:
    - *Switching Branches Without Committing* – You need to switch branches but don’t want to commit unfinished work.
    - *Temporary Save for Testing* – You want to test something but keep current progress safe.

 - **git rebase** is a Git command used to *move* or "replay" commits from one branch onto another. It is often used to maintain a cleaner Git history by incorporating upstream changes without creating unnecessary merge commits.
   - **Types of Rebasing**
     - Basic Rebase (`git rebase <base-branch>`) Moves the current branch to start from the latest state of another branch.
       - Example: You are working on `feature-branch` but `main` has new updates. Instead of merging, you can rebase:
         ```bash
         git checkout feature-branch
         git rebase main
         ```
      ![image](https://github.com/user-attachments/assets/789ac0de-563b-4854-a053-55ecc2008caa)
    - Interactive Rebase (`git rebase -i`) Allows you to modify, reorder, squash, or drop commits before applying them.
      ```bash
      git rebase -i HEAD~3
      ```
      This opens an editor to modify the last 3 commits. Common options in interactive rebase:
      - `pick` – Keep the commit as is.
      - `reword` – Change the commit message.
      - `squash` – Merge multiple commits into one.
      - `drop` – Remove a commit.
  - **Handling Rebase Conflicts**
    - If a conflict occurs:
      - Git will pause and show the conflicting files.
      - Manually fix the conflicts.
      - Use `git add <fixed-file>` to mark them as resolved.
      - Continue rebasing with:
        ```bash
        git rebase --continue
        ```
      - If needed, abort the rebase:
        ```bash
        git rebase --abort
        ```If needed, abort the rebase:

  - **git cherry-pick** is a Git command used to apply a specific commit from one branch to another. Instead of merging or rebasing, it allows you to pick a single commit and apply it to the current branch. This is useful when you need to transfer a bug fix or a feature from one branch without merging all changes.
    - *Basic Cherry-Pick* To apply a specific commit to the current branch. This applies commit `a1b2c3d` to the branch you are currently on.
      ```bash
      git cherry-pick <commit-hash>
      ```
    - *Cherry-Pick a Range of Commits* To apply all commits in a range (excluding the first commit). This applies all commits from `a1b2c3d` to `f4e5d6a`.
      ```bash
      git cherry-pick a1b2c3d^..f4e5d6a
      ```
      ```bash
      git cherry-pick c2 c4
      ```
      ![image](https://github.com/user-attachments/assets/21542158-f16d-471a-875e-36d2ef344be7)

- **git commit --amend** allows you to modify the most recent commit without creating a new commit. It is useful when you want to:
  - Fix a commit message
    - Example: You committed a message but realized it was incorrect.
      ```bash
      git commit -m "Bug fix in API"
      ```
    - You want to change it to "Fix authentication bug". Use:
      ```bash
      git commit --amend -m "Fix authentication bug"
      ```
  - Add missing changes to the last commit
    - Example: You committed a change but forgot to add a file.
      ```bash
      echo "print('Bug fixed')" >> app.py
      git add app.py
      git commit -m "Fix authentication bug"
      ```
    - Oops! You forgot another file:
      ```bash
      echo "print('Logging added')" > logs.py
      git add logs.py
      git commit --amend --no-edit
      ```
    - The last commit now includes logs.py, but the commit message remains unchanged
   
### GIT WorkFlows ###

**Feature Branch Workflow**

The Feature Branch Workflow is a Git workflow that encourages developers to create separate branches for each new feature or bug fix, ensuring that the main (or develop) branch always remains stable.

How the Feature Branch Workflow Works
- Clone the Repository
```bash
git clone <repo_url>
cd <repo_name>
```
- Create a New Feature Branch
  - This keeps the feature isolated from main.
  - Naming convention: `feature/login-page`, `feature/api-integration`, etc.
```bash
git checkout -b feature/<feature-name>
```
- Work on the Feature & Commit Changes
```bash
git add .
git commit -m "Added login functionality"
```
- Push the Feature Branch to Remote
```bash
git push origin feature/<feature-name>
```
- Create a Pull Request (PR)
  - Open a PR on GitHub/GitLab/Bitbucket.
  - Request code review from team members.
- Merge the Feature Branch into main
```bash
git checkout main
git pull origin main
git merge feature/<feature-name>
git push origin main
```

**Gitflow Workflow**

*Develop and main branches*

- Instead of a single main branch, this workflow uses two branches to record the history of the project. The main branch stores the official release history, and the develop branch serves as an integration branch for features. It's also convenient to tag all commits in the main branch with a version number.
- The first step is to complement the default main with a develop branch. A simple way to do this is for one developer to create an empty develop branch locally and push it to the server:
```bash
git branch develop
git push -u origin develop
```
- This branch will contain the complete history of the project, whereas main will contain an abridged version. Other developers should now clone the central repository and create a tracking branch for develop.

![image](https://github.com/user-attachments/assets/7f3ecb20-ab21-4667-ab49-73aca16dffd1)

*Feature branches*
- Each new feature should reside in its own branch, which can be pushed to the central repository for backup/collaboration. But, instead of branching off of main, feature branches use develop as their parent branch. When a feature is complete, it gets merged back into develop. Features should never interact directly with main.
- Note that feature branches combined with the develop branch is, for all intents and purposes, the Feature Branch Workflow. But, the Gitflow workflow doesn’t stop there
- Feature branches are generally created off to the latest develop branch.
```bash
git checkout develop
git branch feature_branch
```
- Finishing a feature branch
```bash
git checkout develop
git merge feature_branch
```
![image](https://github.com/user-attachments/assets/27e9cd6a-18e1-4f32-9d6d-0fd481b183b4)

*Release branches*
- Once develop has acquired enough features for a release (or a predetermined release date is approaching), you fork a release branch off of develop. Creating this branch starts the next release cycle, so no new features can be added after this point—only bug fixes, documentation generation, and other release-oriented tasks should go in this branch. Once it's ready to ship, the release branch gets merged into main and tagged with a version number. In addition, it should be merged back into develop, which may have progressed since the release was initiated.
```bash
git checkout develop
git chekout -b release/0.1.0
```
- Once the release is ready to ship, it will get merged it into main and develop, then the release branch will be deleted. It’s important to merge back into develop because critical updates may have been added to the release branch and they need to be accessible to new features.
```bash
git checkout main
git merge release/0.1.0
```
![image](https://github.com/user-attachments/assets/9d8117c1-7928-4b3a-a18d-d90322d32a49)

*Hotfix branches*
- Maintenance or “hotfix” branches are used to quickly patch production releases. Hotfix branches are a lot like release branches and feature branches except they're based on main instead of develop. This is the only branch that should fork directly off of main. As soon as the fix is complete, it should be merged into both main and develop (or the current release branch), and main should be tagged with an updated version number.
```bash
git checkout main
git checkout -b hotfix-branch
```
- Similar to finishing a release branch, a hotfix branch gets merged into both main and develop.
```bash
git checkout main
git merge hotfix-branch
git checkout develop
git merge hotfix-branch
git branch -D hotfix-branch
```
![image](https://github.com/user-attachments/assets/0ae858e7-ecdf-46ed-8cc5-a17b24bd3a0b)

## Scenarios ##

### Scenario 1: Hotfix Deployment in Production ###
Your team follows a **Git Flow** workflow, where the `main` branch is used for production and `develop` is for ongoing work. A **critical bug** is reported in production. You have fixed the issue in a hotfix branch (`hotfix-123`).
- How do you ensure this fix is deployed quickly without disrupting ongoing development?
- How do you merge the fix into `main` and ensure it's also in `develop`?

**Answer**
-  Create a hotfix branch from `main`:
  ```bash
  git checkout main
  git pull origin main
  git checkout -b hotfix-123
  ```
- Fix the issue and commit it:
  ```bash
  git commit -am "Hotfix: Critical Bug Fix"
  git push origin hotfix-123
  ```
- Merge into `main` and deploy
  ```bash
  git checkout main
  git merge hotfix-123
  git push origin main
  ```
- Ensure `develop` also gets it
  ```bash
  git checkout develop
  git merge hotfix-123
  git push origin develop
  ```

### Scenario 2: CI/CD Fails Due to Merge Conflict ###
Your CI/CD pipeline is failing after merging a feature branch into `main`. The error log shows **merge conflicts** in `config.yml`.
- How do you resolve this issue?
- What Git strategies can you use to prevent such conflicts in the future?

**Answer**
- Identify the Conflict using
  ```bash
  git status
  ```
- Open `config.yml`, manually resolve the conflict, and mark it as resolved:
  ```bash
  git add config.yml
  git commit -m "Resolved merge conflict in config.yml"
  git push origin main
  ```
- Prevent future conflicts by:
  - Enforcing *feature branch rebasing* before merging:
    ```bash
    git checkout feature-branch
    git rebase main
    ```

### Scenario 3: Rolling Back a Broken Deployment ###
Your team deployed a faulty release to production. The issue was introduced by commit `abc123`.
- How do you roll back production to the last stable commit?
- How do you fix and re-deploy correctly?

**Answer**
- Identify the last stable commit:
  ```bash
  git log --oneline
  ```
- Option 1: Reset to last stable commit (if not pushed)
  ```bash
  git reset --hard <stable-commit-hash>
  gut push --force origin main
  ```
- Option 2: Revert (safer if already pushed)
  ```bash
  git revert abc123
  git push origin main
  ```
- Fix the issue, test, and redeploy.
      


