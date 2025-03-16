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

 


