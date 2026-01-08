Security scans in CI/CD are automated checks that look for known, preventable security risks before code reaches production. They look for:
- Known vulnerabilities
- Misconfigurations
- Unsafe dependencies
- Obvious secrets
- Policy violations

***Security scans are Reactive, not Proactive***
- Security scans react to known problems after they already exist; they do not prevent new or unknown problems from being created.
- Concrete examples
  - Example 1: Hardcoded secrets
    - Scanner flags after the secret is committed
    - The secret already leaked
    - Damage is already possible

They do not understand your business logic.

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

