## How Jenkins on EC2 Authenticates to an AWS EKS Cluster ##
In an industry-standard setup, Jenkins running on an EC2 instance typically authenticates to an EKS cluster using AWS IAM roles and **IAM authentication for Kubernetes (IAM for Service Accounts - IRSA or IAM instance roles)**.

### 1. IAM Role-Based Authentication (Recommended) ###
The most secure and scalable way is using an **IAM Role with EC2 Instance Profile**. This allows Jenkins to assume an IAM role that has permissions to interact with EKS.

**Steps:**

**Create an IAM Role for Jenkins EC2**
- If Jenkins needs to *deploy applications* to an EKS cluster, it requires additional permissions beyond just authentication. Below is the industry-standard IAM policy that provides Jenkins the necessary permissions for deployment.

![image](https://github.com/user-attachments/assets/a880b69b-e6bf-4adf-b473-d059594d8b9f)


**Attach the Role to Jenkins EC2 Instance**
- Attach the IAM role to the EC2 instance running Jenkins.

**Update `aws-auth` ConfigMap in EKS**
- The `aws-auth` ConfigMap is a critical Kubernetes configuration in Amazon EKS that *maps AWS IAM identities (users, roles, or instance profiles) to Kubernetes RBAC roles and groups*.
- This allows AWS IAM users or roles to authenticate and interact with the Kubernetes cluster.
- ```bash
  kubectl get configmap aws-auth -n kube-system -o yaml
  ```
- Structure of aws-auth ConfigMap
  - `mapRoles` → Maps IAM roles (recommended for Jenkins).
  - `mapUsers` → Maps IAM users (not recommended for automation).
  - `mapAccounts` → Grants cluster access to entire AWS accounts (not commonly used).
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: aws-auth
    namespace: kube-system
  data:
    mapRoles: |
      - rolearn: arn:aws:iam::123456789012:role/JenkinsEKSRole
        username: jenkins
        groups:
          - system:masters
    mapUsers: |
      - userarn: arn:aws:iam::123456789012:user/dev-user
        username: dev-user
        groups:
          - system:masters
    mapAccounts: |
      - "123456789012"
  ```

**Configure Jenkins to Use AWS CLI for Authentication**
- Use `aws eks update-kubeconfig --region <region> --name <cluster-name>` inside the Jenkins pipeline to generate the `kubeconfig` file dynamically.
