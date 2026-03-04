The `input` step:
- Pauses pipeline execution
- Waits for human interaction
- Can collect additional runtime values
- Resumes only after approval

```groovy
stage('Production Approval') {
    steps {
        input message: 'Approve deployment to PROD?',
              ok: 'Deploy'
    }
}
```

- Pipeline enters a waiting state
- Build executor is released. Means Jenkins Pauses execution, Saves pipeline state, Releases the executor, Waits for approval. So the agent is no longer blocked. That means: The node becomes free Another job can use that executor
- UI shows “Proceed” / “Abort”
- An authorized user clicks to continue
