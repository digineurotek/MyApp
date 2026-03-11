# ArgoCD Setup on Kubernetes (Minikube / Docker Desktop) – Complete Guide

This guide explains how to install, access, and use **ArgoCD** with a local Kubernetes cluster. It covers the full workflow from cluster setup to deploying an application from Git.

---

# 1. Prerequisites

Before installing ArgoCD, ensure the following tools are installed:

### Required Tools

* Kubernetes cluster (Minikube or Docker Desktop Kubernetes)
* kubectl CLI
* Internet connection

### Verify kubectl

```bash
kubectl version --client
```

### Verify cluster

```bash
kubectl get nodes
```

Expected output:

```
NAME       STATUS   ROLES           AGE
minikube   Ready    control-plane
```

---

# 2. Create ArgoCD Namespace

ArgoCD runs inside its own Kubernetes namespace.

```bash
kubectl create namespace argocd
```

Check namespaces:

```bash
kubectl get ns
```

---

# 3. Install ArgoCD

Run the official installation manifest.

Recommended command (server-side apply):

```bash
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

This command installs:

* ArgoCD Server
* Application Controller
* Repo Server
* Redis
* Dex (authentication)
* CRDs
* Services

---

# 4. Verify Installation

Check pods running in the namespace:

```bash
kubectl get pods -n argocd
```

Expected output (after 1–2 minutes):

```
NAME                                               READY   STATUS
argocd-server                                      1/1     Running
argocd-repo-server                                 1/1     Running
argocd-application-controller                      1/1     Running
argocd-dex-server                                  1/1     Running
argocd-redis                                       1/1     Running
```

---

# 5. Access ArgoCD UI in Browser

By default ArgoCD is not exposed externally.

Use port forwarding:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open browser:

```
https://localhost:8080
```

You will see the **ArgoCD Login Page**.

---

# 6. Get Username and Password

### Username

```
admin
```

### Get Initial Password

Run:

```windows
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String((kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}")))
```

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

Example output:

```
Xk3h7s9Abc
```

Use this password to login.

---

# 7. Login Using ArgoCD CLI (Optional)

Install CLI Windows:

```
winget install argocd

```

Install CLI:

Download from

```
https://github.com/argoproj/argo-cd/releases/latest
```

Login command:

```bash
argocd login localhost:8080
```

Username:

```
admin
```

Password:

```
(initial password from secret)
```

---

# 8. Change Default Password

For security change password after login:

```bash
argocd account update-password
```

You can also delete the initial secret:

```bash
kubectl delete secret argocd-initial-admin-secret -n argocd
```

---

# 9. Create an Application from Git

Example repository:

```
https://github.com/argoproj/argocd-example-apps.git
```

Create application using CLI:

```bash
argocd app create guestbook \
--repo https://github.com/argoproj/argocd-example-apps.git \
--path guestbook \
--dest-server https://kubernetes.default.svc \
--dest-namespace default
```

---

# 10. Check Application Status

```bash
argocd app get guestbook
```

Initial state:

```
Sync Status: OutOfSync
Health Status: Missing
```

---

# 11. Deploy Application (Sync)

Deploy application:

```bash
argocd app sync guestbook
```

ArgoCD will:

1. Pull manifests from Git
2. Apply them to Kubernetes
3. Deploy pods automatically

---

# 12. Verify Deployment

Check pods:

```bash
kubectl get pods
```

You should see guestbook pods running.

---

# 13. Deploy Using ArgoCD UI

Open:

```
https://localhost:8080
```

Steps:

1. Login
2. Click **New App**
3. Enter:

Application Name

```
guestbook
```

Repository

```
https://github.com/argoproj/argocd-example-apps.git
```

Path

```
guestbook
```

Cluster

```
https://kubernetes.default.svc
```

Namespace

```
default
```

Click **Create**

Then click **Sync** to deploy.

---

# 14. Architecture Overview

```
Git Repository
      ↓
ArgoCD
      ↓
Kubernetes Cluster
      ↓
Pods / Services
```

ArgoCD continuously watches Git and automatically deploys updates.

---

# 15. Useful Commands

Check ArgoCD pods

```bash
kubectl get pods -n argocd
```

Check services

```bash
kubectl get svc -n argocd
```

Delete ArgoCD

```bash
kubectl delete namespace argocd
```

Restart Minikube

```bash
minikube stop
minikube start
```

---

# 16. Real DevOps Workflow

Production workflow typically looks like:

```
Developer
    ↓
GitHub Repository
    ↓
ArgoCD (GitOps)
    ↓
Kubernetes Cluster
    ↓
Application Pods
```

When code changes in Git:

1. ArgoCD detects changes
2. Automatically deploys new version
3. Updates running Kubernetes pods

---

# Conclusion

You now have a complete GitOps deployment system:

* Kubernetes cluster
* ArgoCD installed
* Git-based deployment
* Web UI monitoring

This setup is widely used in modern DevOps and cloud-native infrastructure.

---
