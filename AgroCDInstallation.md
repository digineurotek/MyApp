# ArgoCD Installation on Kubernetes (Minikube) – Complete Step-by-Step Guide

This guide explains how to install and run ArgoCD on a local Kubernetes cluster using Minikube. It also explains the purpose of each command so the workflow is easy to understand.

---

# 1. Check Available Kubernetes Contexts

First check which Kubernetes clusters are configured in your kubeconfig.

```
kubectl config get-contexts
```

Example output:

```
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop
          minikube         minikube         minikube         default
```

Explanation:

* Kubernetes can manage multiple clusters.
* Each cluster configuration is called a **context**.
* The `*` indicates the currently active cluster.

---

# 2. Switch to Minikube Cluster

To run ArgoCD on Minikube, switch the active cluster:

```
kubectl config use-context minikube
```

Now all `kubectl` commands will run against the **Minikube Kubernetes cluster**.

---

# 3. Get the Latest ArgoCD Version

Use the GitHub API to retrieve the latest ArgoCD release version.

```
$version = (Invoke-RestMethod https://api.github.com/repos/argoproj/argo-cd/releases/latest).tag_name
```

Explanation:

This command performs the following steps:

1. Sends a request to the GitHub API.
2. GitHub returns release information in JSON format.
3. The command extracts the `tag_name` field from the JSON response.
4. The value represents the latest ArgoCD version (example: `v2.11.3`).
5. The version is stored inside the `$version` variable.

This variable can later be used in installation commands.

---

# 4. Create the ArgoCD Namespace

ArgoCD runs inside its own namespace.

```
kubectl create namespace argocd
```

Verify the namespace:

```
kubectl get ns
```

Example output:

```
NAME              STATUS
default           Active
kube-system       Active
argocd            Active
```

---

# 5. Install ArgoCD

Install ArgoCD using the official Kubernetes manifest:

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/$version/manifests/install.yaml
```

What this command does:

* Downloads the ArgoCD Kubernetes manifests
* Creates required resources inside the cluster

Resources created include:

* Deployments
* Services
* StatefulSets
* ConfigMaps
* Secrets
* CRDs (Custom Resource Definitions)
* RBAC roles and permissions

After installation, ArgoCD components start running inside Kubernetes pods.

---

# 6. Verify Installation

Check if ArgoCD pods are running:

```
kubectl get pods -n argocd
```

Example output:

```
NAME                                      READY   STATUS
argocd-server                             1/1     Running
argocd-repo-server                        1/1     Running
argocd-application-controller             1/1     Running
argocd-dex-server                         1/1     Running
argocd-redis                              1/1     Running
```

---

# 7. Access the ArgoCD Web UI

ArgoCD is not exposed externally by default.

Use port forwarding to access it locally:

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open your browser and go to:

```
https://localhost:8080
```

---

# 8. Get Login Credentials

Username:

```
admin
```

Retrieve the initial password:

```
kubectl get secret argocd-initial-admin-secret \
-n argocd \
-o jsonpath="{.data.password}" | base64 --decode
```

Use this password to login to the ArgoCD dashboard.

---

# 9. Delete ArgoCD (Optional)

If you want to remove ArgoCD completely:

```
kubectl delete namespace argocd
```

Deleting the namespace removes:

* All ArgoCD pods
* Services
* Configurations
* Secrets

---

# 10. ArgoCD Architecture

```
Git Repository
      ↓
ArgoCD
      ↓
Kubernetes Cluster
      ↓
Pods / Services
```

ArgoCD continuously monitors a Git repository and automatically deploys changes to Kubernetes.

This model is known as **GitOps**.

---

# 11. Typical DevOps Workflow

```
Developer commits code
        ↓
GitHub repository updates
        ↓
ArgoCD detects change
        ↓
Kubernetes deployment updated
        ↓
Application pods automatically updated
```

---

# Summary

In this guide you learned how to:

* Switch Kubernetes contexts
* Retrieve the latest ArgoCD version
* Create an ArgoCD namespace
* Install ArgoCD
* Access the ArgoCD UI
* Retrieve login credentials
* Remove ArgoCD if needed

You now have a fully working **GitOps deployment environment using ArgoCD and Kubernetes**.

---
