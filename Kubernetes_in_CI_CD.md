### Deploying to a Minikube Cluster Using Jenkins ###
**kubectl Configured for Minikube**
```bash
kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
- `kubectl` must be installed on the Jenkins Agent

**A Kubernetes ServiceAccount with RBAC**
- Create a ServiceAccount for Jenkins:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-sa-role
subjects:
- kind: ServiceAccount
  name: jenkins-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```
- Apply it: `kubectl apply -f jenkins-sa.yaml`
- Get the token:
```bash
kubectl create token jenkins-sa -n default
eyJhbGciOiJSUzI1NiIsImtpZCI6InhqRVdIcFpUaGdnS2JGZm9nY2M3X1FxbU1ZMGpIYzVudklKMmdyY2UzVkUifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQyODExMjAxLCJpYXQiOjE3NDI4MDc2MDEsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiYmFjYzVmNTctZDA2Mi00ZjRhLTk4MWUtYjI2NTZhNDdhMjJhIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImplbmtpbnMtc2EiLCJ1aWQiOiIyOWM1MzczZi1jMTBlLTQyMDUtYjFhOC0yYWQxMTIyOGVhZTIifX0sIm5iZiI6MTc0MjgwNzYwMSwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6amVua2lucy1zYSJ9.ABJ8_XJNfFWrbJKqD0EuneUv411KyavRdiM5NRVAQB9nkbNEx9QIgYYE2fpyGu8DH_h_c6BC6GvumKVFb2qWK-21lz8VpBduqM-DNTkaku6pk9a7rmy_ydtikDh38POX_Sti_6FdpxjAtvF92tsgj4jrieJwULbCCm5xaCba82wA_5IornJUO8bcJUc0YgVU15ulMcQzQNS5pQ6jKnKC0GtlCo-jVs-GwBUd0jTdfl9gJnBdlf8N7yY_T4qvTt6bSqFPoYwa991xVjUwNl2y8O45APonfsaH29N9nl9mHkrZRN4LzXO-r4_AOTU3B7eL-GikRio7215fwDy527XSRQ
```
- Store this token in Jenkins Credentials as a Secret Text (`K8S_SA_TOKEN`).

**Jenkins Pipeline to Deploy to Minikube**
```groovy
pipeline {
    agent any
    environment {
        K8S_API_SERVER = "https://$(minikube ip):8443"
        NAMESPACE = "default"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'git@github.com:nawab312/Sample.git'
            }
        }
        stage('Deploy to Minikube') {
            steps {
                withCredentials([
                    string(credentialsId: 'K8S_SA_TOKEN', variable: 'K8S_TOKEN')
                ]) {
                    script {
                        sh """
                        kubectl config set-cluster minikube --server=$K8S_API_SERVER --insecure-skip-tls-verify=true
                        kubectl config set-credentials jenkins-user --token=$K8S_TOKEN
                        kubectl config set-context minikube --cluster=minikube --user=jenkins-user --namespace=$NAMESPACE
                        kubectl config use-context minikube
                        kubectl apply -f deployment.yaml
                        """
                    }
                }
            }
        }
    }
}
```
