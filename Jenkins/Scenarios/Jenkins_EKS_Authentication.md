## How Jenkins on EC2 Authenticates to an AWS EKS Cluster ##
In Amazon EKS, **IAM authentication** is used to securely authenticate AWS users, roles, or applications (like Jenkins) with the Kubernetes cluster. However, IAM **only handles authentication, not authorization**. Authorization is managed using **Kubernetes RBAC (Role-Based Access Control)**, which defines what actions are allowed.

The authentication process involves:
- AWS IAM authentication via `aws-iam-authenticator`
- Generating a temporary authentication token (`eks:GetToken`)
- Mapping IAM roles to Kubernetes RBAC using `aws-auth` ConfigMap
- Using `kubectl` to interact with the cluster

### Workflow: How IAM + RBAC Work Together ###
- Jenkins (running on EC2) runs `kubectl apply -f deployment.yaml`.
- AWS IAM validates Jenkins' identity using `aws-iam-authenticator`.
- IAM role (`JenkinsEKSRole`) is mapped to Kubernetes user (`jenkins`) via `aws-auth ConfigMap`.
- Kubernetes RBAC checks if `jenkins` has permission to deploy applications.
- If authorized, Jenkins successfully deploys to EKS.

### IAM Authentication in EKS ###
Unlike a typical Kubernetes cluster where users authenticate using certificates, **EKS relies on AWS IAM and tokens**.
- IAM users or roles do **not** have direct access to Kubernetes.
- Instead, IAM roles are mapped to Kubernetes users via the `aws-auth` ConfigMap.
- AWS provides an authentication mechanism called `aws-iam-authenticator`, which validates IAM identities before granting access.

**Create an IAM Role for Jenkins EC2**
- If Jenkins needs to *deploy applications* to an EKS cluster, it requires additional permissions beyond just authentication. Below is the industry-standard IAM policy that provides Jenkins the necessary permissions for deployment.

![image](https://github.com/user-attachments/assets/a880b69b-e6bf-4adf-b473-d059594d8b9f)

**Attach the Role to Jenkins EC2 Instance**
- Attach the IAM role to the EC2 instance running Jenkins.

**Authentication Flow**
- Request Authentication Token
  - The `kubectl` command triggers a request to authenticate with EKS.
  - `kubectl` runs `aws eks get-token`, which fetches a temporary authentication token.
    ```bash
    aws eks get-token --cluster-name my-eks-cluster
    ```
- IAM Role Validation
  - AWS checks if the user or role has `eks:GetToken` permissions in IAM.
  - If authorized, AWS generates a short-lived authentication token.
- Using the Token in `kubeconfig`
  - The token is stored in the `kubeconfig` file, allowing `kubectl` to use it for cluster access.
  - Example `kubeconfig` entry:
    ```yaml
    users:
    - name: jenkins
      user:
        exec:
          apiVersion: client.authentication.k8s.io/v1beta1
          command: aws
          args:
            - eks
            - get-token
            - --cluster-name
            - my-eks-cluster
    ```

### IAM + Kubernetes RBAC (Authorization) ###
Once authentication is successful, Kubernetes **checks authorization** using **RBAC policies**. This ensures that IAM users/roles can only perform allowed actions.

**How IAM Roles Are Mapped to Kubernetes Users?**
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

### Defining Kubernetes RBAC Permissions ###
Once mapped, Kubernetes **RBAC roles** determine what actions Jenkins can perform.

**Create a Kubernetes Role for Jenkins**
- Instead of giving full `system:masters` access, create a restricted role:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    namespace: default
    name: jenkins-deployer
  rules:
  - apiGroups: [""]
    resources: ["pods", "deployments", "services"]
    verbs: ["get", "list", "create", "update", "delete"]
  ```
- Bind the Role to Jenkins IAM Role
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: jenkins-deployer-binding
    namespace: default
  subjects:
  - kind: User
    name: jenkins
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: jenkins-deployer
    apiGroup: rbac.authorization.k8s.io
  ```
- Modify the aws-auth ConfigMap as follows:
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
  ```

- This ensures that *Jenkins (mapped IAM Role)* can only manage deployments, pods, and services in the `default` namespace.
   
