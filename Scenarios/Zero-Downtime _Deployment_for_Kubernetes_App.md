**Scenario:** Your 3-tier banking app runs on Kubernetes. How do you ensure zero downtime during a CI/CD release?

Follow-ups:
  -   How would you configure Rolling Updates in Kubernetes?
  -   What role does Ingress and Service Mesh (e.g., Istio) play in CI/CD?

### Zero-Downtime Deployment for a Kubernetes-Based Banking App ###
Ensuring zero downtime for a 3-tier banking application on Kubernetes during CI/CD releases requires a Rolling Update strategy, blue-green/canary deployment patterns, traffic management, and monitoring. 

**Configuring Rolling Updates in Kubernetes**
- Rolling updates allow a *gradual replacement* of old pods with new ones without downtime.
- Set RollingUpdate strategy in the Deployment manifest.
```yaml
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # At most, 1 pod can be down
      maxSurge: 2         # Allow 2 extra pods to spin up before termination
```

**Blue-Green & Canary Deployments (Advanced CI/CD Strategies)**
For critical banking apps, Rolling Updates alone may not be enough. Consider Blue-Green or Canary Deployments.
- **Blue-Green Deployment (Two Separate Environments)**
  - Traffic is switched from `blue (old)` to `green (new)` when green is ready.
  - If issues occur, rollback instantly by switching traffic back.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: banking-ingress
spec:
  rules:
  - host: banking.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: banking-green  # Switch traffic to green environment
            port:
              number: 80
```
- **Canary Deployment (Gradual Traffic Shifting)**
  - A percentage of traffic is gradually shifted from v1 to v2.
  - If errors increase, rollback immediately.
```yaml
# If errors increase, rollback immediately.
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: banking-route
spec:
  hosts:
    - banking.example.com
  http:
    - route:
        - destination:
            host: banking-app
            subset: v1
          weight: 80  # 80% traffic to v1
        - destination:
            host: banking-app
            subset: v2
          weight: 20  # 20% traffic to v2
```

**What role does Ingress and Service Mesh (e.g., Istio) play in CI/CD?**
- An **Ingress Controller** manages external traffic into a Kubernetes cluster, ensuring seamless deployment updates without downtime. In a CI/CD pipeline, it plays a key role in:
  - *Traffic Routing & Load Balancing:* Directs user traffic to different versions of an application (e.g., v1 and v2 during deployment). Supports blue-green and canary deployments by dynamically switching traffic between versions.
  - *SSL/TLS Termination* Offloads HTTPS termination at the ingress layer, reducing application overhead.
  - *Zero-Downtime Deployments* By updating the Ingress rules, we can shift traffic instantly to the new version (green deployment) without killing old pods (blue).
- A **Service Mesh** like Istio provides advanced traffic management, observability, and security, which are essential for enterprise-grade CI/CD pipelines.
  - *Traffic Management & Canary Releases* Enables gradual traffic shifting between application versions using Istio VirtualServices.
  - *A/B Testing & Feature Flags* Routes specific users (e.g., premium banking customers) to a new version before full rollout. Supports header-based, cookie-based, or weight-based routing for precise control.
  - *Security & mTLS (Mutual TLS)* Secures communication between microservices using automatic encryption (TLS). Eliminates the need for hardcoded API keys or shared secrets in CI/CD pipelines by enforcing secure, automated service-to-service authentication using mTLS, JWT, and fine-grained access policies. Example Workflow::
    - Service A wants to talk to Service B.
    - Instead of sending an API key, Istio automatically establishes an mTLS connection.
    - Both services verify each other's identity using Istio-issued certificates.
    - If authentication fails, the request is denied before it even reaches the application.

- **In most enterprise CI/CD pipelines, Ingress and Istio work together:**
  - Ingress handles external traffic.
  - Istio manages internal microservice communication and traffic shifting.
