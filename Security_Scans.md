Security scans in CI/CD are automated checks that look for known, preventable security risks before code reaches production. They look for:
- Known vulnerabilities
- Misconfigurations
- Unsafe dependencies
- Obvious secrets
- Policy violations

---

***Security scans are Reactive, not Proactive***
- Security scans react to known problems after they already exist; they do not prevent new or unknown problems from being created.
- Concrete examples
  - Example 1: Hardcoded secrets
    - Scanner flags after the secret is committed
    - The secret already leaked
    - Damage is already possible

They do not understand your business logic.

**What can be done better (Proactive controls)**

*Shift security left — before code is committed*
- Pre-commit hooks (e.g., secret scanning before git commit)
- IDE plugins that flag insecure code while writing it
- Why this matters: Prevents mistakes instead of reporting them later.
- Hard truth: If the first time you detect a secret is in CI, you’re already late.

*Enforce guardrails, not warnings*
- Replace: “Scan and alert” With: “Block unsafe actions”
- Examples:
  - Block commits containing secrets
  - Fail pipelines if infra violates policy (open S3, public SGs)
  - Admission controllers in Kubernetes
 
*Secure-by-default platforms*
- Reduce the chance of error instead of expecting perfection.
- Examples:
  - Short-lived credentials instead of static secrets
  - Managed IAM roles instead of API keys

---

## Main types of security scans (and what they really catch) ##

### SAST — Static Application Security Testing ###
- Scans: Source code
- Finds: Insecure coding patterns
- Examples:
  - SQL injection patterns
  - Hardcoded credentials
  - Unsafe deserialization
- Blind Spot: It doesn’t know how the code is used at runtime

### SCA — Software Composition Analysis ###
- Scans: Dependencies
- Finds: Known CVEs in libraries
- Examples:
  - Log4Shell
  - Outdated OpenSSL
  - Vulnerable npm packages
- Critical weakness: Known CVE ≠ exploitable in your app. No CVE ≠ safe

### DAST — Dynamic Application Security Testing ###
- Scans: Running Application
- Finds: Runtime Vulnerabilities
- Examples:
  - XSS
  - Auth Misconfigurations
  - Open endpoints
- Reality check: DAST is slow, flaky, and shallow without real traffic patterns.

