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
- Build executor is released. Means Jenkins Pauses execution, Saves pipeline state, Releases the executor, Waits for approval. So the agent is no longer blocked. That means: The node becomes free. Another job can use that executor
- UI shows “Proceed” / “Abort”
- An authorized user clicks to continue
- Most common use case: Dev → QA → Staging → Manual Approval → Production

### Who Can Approce ###

**RBAC Control**
```groovy
input message: "Deploy?",
      submitter: "DevOps-Team"
```
- `submitter` restricts who can click “Proceed.”
- `devops-team` is not magically an LDAP group. It must match a user or group recognized by Jenkins Security Realm + Authorization strategy.

**How To Tie Approval to LDAP / AD Groups (Proper Way)**
- Step 1: Configure LDAP/AD in Jenkins
  - Go to: *Manage Jenkins → Security → Security Realm*
  - Select: LDAP or Active Directory. Use the LDAP plugin or Active Directory plugin.
  - This connects Jenkins to: Corporate LDAP, Active Directory
- Step 2: Configure Authorization Strategy
  - *Manage Jenkins → Authorization*
  - Use one of: Matrix-based security or Role-Based Strategy Plugin
  - With Role-Based Strategy Plugin:
    - Create role: `prod-approvers`
    - Assign LDAP group: `CN=DevOps-Team,OU=Groups,DC=company,DC=com`
    - Now Jenkins knows: Anyone in this LDAP group has this role.
- You can also capture who approved using `submitterParameter`
  ```groovy
  stage('Production Approval') {
    steps {
        input message: "Deploy to Production?",
              submitter: "DevOps-Team",
              submitterParameter: "APPROVED_BY"
    }
  }
  ```
  - Then: `echo "Approved by: ${APPROVED_BY}"`
 
### Timeout Risk ###
- What if no one approves?
```groovy
timeout(time: 1, unit: 'HOURS') {
    input message: 'Approve?'
}
```
