**Scenario:** Your company has *Dev*, *Staging*, and *Production* environments. How would you design a CI/CD pipeline that ensures proper testing before code reaches production?

**Source Code Management (SCM) Strategy**
- Use Git branching strategy (e.g., `feature`, `develop`, `staging`, `main`)
- **Feature Branches (`feature/*`)**
  - Who works here? Developers working on new features.
  - Naming Convention: `feature/new-login-ui`, `feature/add-cart-api`.
  - Workflow:
    - A developer creates a new branch from `develop`
    - Commits code frequently and pushes changes.
    - Runs unit *tests & static analysis* before merging.
    - Creates a *Pull Request (PR)* for review.
    - After approval, the feature branch is merged into develop.
