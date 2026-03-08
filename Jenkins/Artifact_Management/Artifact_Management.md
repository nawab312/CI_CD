- *Storing Build Artifacts: Use repositories like Nexus, Artifactory, AWS S3, or Docker Hub to store build artifacts for consistency and traceability.*
- *Versioning & Tagging Releases: Use version numbers and tags to ensure proper tracking and deployment of specific versions across environments.*

**Storing Build Artifacts**

When we talk about storing build artifacts, we're referring to the process of saving the output of your build process (e.g., compiled binaries, Docker images, libraries) in a centralized repository. 
This is crucial to ensure the availability, traceability, and consistency of artifacts throughout the lifecycle of an application. Popular Artifact Repositories:
- Nexus: A popular artifact repository manager that can store, manage, and retrieve build artifacts. It supports various formats, such as Maven (for Java), npm (for JavaScript), and Docker images.
- Artifactory: Another widely used artifact repository that works similarly to Nexus but with more advanced features, like high availability and integration with CI/CD pipelines. It supports a broad set of packaging formats (Maven, npm, Docker, etc.).
- AWS S3: While not a traditional artifact repository, Amazon's Simple Storage Service (S3) is often used to store artifacts (e.g., tarballs, ZIP files, Docker images) due to its scalability and durability.
- Docker Hub: A container registry used specifically for storing and sharing Docker images, often used in CI/CD pipelines for containerized applications.

Why is it important?
- Centralizes storage: Artifacts are stored in one place, making it easy to retrieve them later.
- Prevents "works on my machine" problems: Ensures that the same artifact is used across all environments (development, staging, production).

**Versioning**

Versioning ensure that the correct version of the artifact is deployed to different environments and prevent confusion or errors during deployment.
- Versioning: Each artifact is assigned a unique version number (e.g., `1.0.0`, `2.1.1`). This version number helps in tracking which version of the application is deployed, tested, or running in production.
  - Semantic Versioning (SemVer): This is a popular versioning strategy, where version numbers consist of three parts: `MAJOR.MINOR.PATCH`. For example:
    - `1.0.0`: Initial release (major)
    - `1.1.0`: New features (minor)
    - `1.1.1`: Bug fixes (patch)
 
Why is it important?
- Traceability: Versioning ensures you can trace which version of an artifact was deployed, making it easier to debug or roll back if needed.
- Reproducibility: You can recreate exactly the same environment with the same artifact by pulling the specific version from the repository.

**Artifact Promotion**
- Artifact promotion is the practice of building a deployable artifact exactly once, storing it in a versioned repository, and then moving (promoting) that same artifact through environments — Dev → Staging → Production — rather than rebuilding at each stage. *Only the config changes at each environment — never the artifact itself*

How It Works Technically — JAR / WAR
- Developer pushes code
  - git push origin main → CI pipeline (Jenkins / GitHub Actions) wakes up
  - The pipeline is triggered automatically on every push.
- CI builds artifact ONCE
  - mvn build → payment-service-v1.2.3.jar
  - Metadata is stamped: Git commit hash, build time, who triggered, test results.
- Artifact pushed to Repository
  - artifactory.company.com/libs-dev/payment-service/v1.2.3/payment-service-v1.2.3.jar
- Deploy to DEV + automated tests
  - Pipeline pulls v1.2.3 from repo and deploys to DEV server. Unit tests, API tests, smoke tests run.
- Quality Gate check
  - Unit tests ■ | Security scan ■ | Coverage >80% ■ | QA approval ■
  - All gates must pass. Only then does promotion happen.
- Promote to STAGING
  - Same v1.2.3.jar deployed to staging. Config changes (DB URL = staging DB). Binary is identical.
  - In JFrog, this means moving the artifact from libs-dev-local → libs-staging-local.
- Promote to PRODUCTION
  - Same v1.2.3.jar. Config now points to production DB & services.
```bash
libs-dev-local ← artifact lands here first (after build)
libs-staging-local ← promoted here after dev tests pass
libs-release-local ← promoted here for production use
BEFORE: payment-service-v1.2.3.jar status: 'dev-tested'
AFTER: payment-service-v1.2.3.jar status: 'release-ready' (same file!)
```

Docker Image Promotion — Modern Microservices
```bash
registry.company.com / payment-service : v1.2.3
        ↑                    ↑              ↑
    Where stored         App name     Version tag
# AWS: 123456.dkr.ecr.ap-south-1.amazonaws.com/payment-service:v1.2.3
# GCP: gcr.io/mycompany/payment-service:v1.2.3
# JFrog: mycompany.jfrog.io/docker/payment-service:v1.2.3
```
- SHA Digest — The True Identity
  - Every Docker image gets a SHA256 digest automatically. Even if someone changes the tag (e.g. v1.2.3 → latest), the SHA never changes. This is how you prove it's the exact same image throughout the entire promotion chain.
- Build image once: `docker build -t payment-service:v1.2.3 .`
- Push to dev registry: `docker push mycompany.jfrog.io/docker-dev/payment-service:v1.2.3`
- Security scan: Trivy / Snyk scans image layers for CVEs. CRITICAL/HIGH vulnerabilities block promotion.
- Deploy to DEV + test: Kubernetes pulls image. Unit, API, integration tests run automatically.
- Retag & push to staging: docker tag ...docker-dev/... ...docker-staging/... docker push ...docker-staging/...
- Deploy to STAGING: Same image. Config changes via env vars / ConfigMaps. QA tests & sign-off.
- Retag & push to prod: docker tag ...docker-staging/... ...docker-prod/... docker push ...docker-prod/...
- Deploy to PRODUCTION

Promotion is almost always automated — the CI/CD pipeline calls the JFrog API or Docker registry API to move the artifact. Humans are only involved as an approval gate before production, clicking approve in a tool like Spinnaker or Jenkins. Nobody manually copies files in JFrog day-to-day

*BINARY SIGNING*
- After your CI pipeline builds an artifact (JAR or Docker image), it digitally signs it using a private key. This creates a signature attached to the artifact.
- Later, when anyone tries to deploy it, they verify the signature using a public key to confirm:
  - This artifact was built by our official CI pipeline (not a hacker)
  - Nobody has tampered with it since it was signed
  - This is the exact same file that was tested
- Docker Image Signing — Real World Example. The most common tool today is *Cosign* (by Sigstore):
```code
# Step 1 — CI pipeline SIGNS the image after building
cosign sign \
  --key cosign.key \
  mycompany.jfrog.io/docker-prod/payment-service:v1.2.3

# This attaches an invisible signature to the image in the registry
# The SHA digest is what gets signed — not the tag
```
```code
# Step 2 — Before deployment, Kubernetes VERIFIES it
cosign verify \
  --key cosign.pub \
  mycompany.jfrog.io/docker-prod/payment-service:v1.2.3
```
  
