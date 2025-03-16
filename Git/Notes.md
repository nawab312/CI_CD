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
