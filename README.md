
![My project-6](https://github.com/user-attachments/assets/3cbc0fd4-2c51-4fa3-bace-53c2f78cd597)




# Set Up Your Development Environment
- brew install git
- brew install kubectl
- brew install argocd
- brew install minikube


# Step 0: 
- minikube status
- minikube stop
- minikube delete
- minikube start
- kubectl config use-context minikube  # activation
- kubectl config get-contexts # kubectl is pointing to the correct cluster and context.
- kubectl get pods -n argocd #Once Minikube is running, check if Argo CD resources are available and running



# Step 1: 
 -Run the following commands to install Argo CD in your Kubernetes cluster:
- $ kubectl create namespace argocd
- $ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


# Step 2: 
You can access the Argo CD server by port forwarding:
- $ kubectl port-forward svc/argocd-server -n argocd 8080:443
Now you can access the Argo CD UI at http://localhost:8080.
---------------------------------------------------------------
or Install Argo CD CLI command line
- https://github.com/argoproj/argo-cd/releases
- argocd login localhost:8080
- argocd app list



# Step 3: 
Retrieve the Initial Admin Password
- kubectl get secrets -n argocd
- kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode


# Step 4:
Create an Argo CD Application that points to your GitHub repository:

- $ argocd app create fastapi-cicd \
  --repo https://github.com/mehdi-repo/fastapi-cicd.git \
  --path kubernetes \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --revision main



# Step 5: 
Sync the Application
- $ argocd app sync <app-name>


# Step 6:
Monitor the Application
Check the status of your application to see if it has been successfully deployed:

- $ argocd app get <app-name>
