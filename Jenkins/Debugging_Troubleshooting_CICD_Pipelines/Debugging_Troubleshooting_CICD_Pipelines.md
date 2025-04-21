**How do you handle failed builds in a CI/CD pipeline?**

When a build fails, the first step is to identify the nature of the failure. It could be due to compilation errors, failing unit tests, environment issues, dependency resolution, or infrastructure (like Docker daemon errors or disk space issues. Compilation errors are caused by mistakes in the codebase that prevent it from building successfully — like syntax issues or missing dependencies. Environment issues are related to the CI/CD system setup — like missing tools, wrong versions, or misconfigured build runners. I address compilation errors by analyzing the logs and fixing the code, and handle environment issues by updating runner configurations or Docker images

*Prevent Recurrence*
- Add automated retry logic: If a job fails due to a temporary issue, the pipeline automatically retries it without manual intervention
```groovy
retry(3) {
    sh './run-tests.sh'
}
```
  - Transient network failures
    - Example: A pipeline stage pulls a Docker image or fetches a dependency from an external registry (e.g., npm, PyPI, Maven, Docker Hub).
    - Occasionally, due to DNS failures or rate limits, the job fails even though the code is fine.
    - Retry logic helps: Automatically retries the stage — it often succeeds on the second or third attempt when the transient issue clears up.
    `

- Improve tests or pipelines: If builds are failing due to poor tests or unstable pipeline steps, I improve their quality to make them more reliable
- Add pipeline stage timeouts: If a job hangs indefinitely (e.g., waiting for a service or a broken loop), I set time limits to prevent it from blocking the entire pipeline.
- Automate rollback or artifact cleanup: When a build or deploy fails, rollback changes or clean up resources to keep the environment clean
