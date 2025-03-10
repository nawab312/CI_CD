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
