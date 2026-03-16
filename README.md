Prerequisites

Install the following tools before starting.

1. Install Docker

Docker is required because Minikube uses Docker to run the Kubernetes cluster.

Download:
https://www.docker.com/products/docker-desktop/

Verify installation:
`docker --version`

2. Install kubectl

kubectl is the Kubernetes command line tool used to interact with the cluster.

Download:
https://kubernetes.io/docs/tasks/tools/

Verify:
`kubectl version --client`

3. Install Minikube

Minikube runs a local Kubernetes cluster.

Download:
https://minikube.sigs.k8s.io/docs/start/

Verify installation:
`minikube version`

4. Install ArgoCD CLI

Download ArgoCD CLI:

https://argo-cd.readthedocs.io/en/stable/cli_installation/

Windows installation example:
`curl -sSL -o argocd.exe https://github.com/argoproj/argo-cd/releases/latest/download/argocd-windows-amd64.exe`

Move it to a folder like:
C:\Program Files\argocd

Run using:
`.\argocd.exe version`


<br><br>
<br>

Step 1: Start Kubernetes Cluster

Start Minikube:
`minikube start --driver=docker`

minikube start --driver=docker:
`minikube status`

Check nodes:
`kubectl get nodes`


<br><br>
<br>

Step 2: Create ArgoCD Namespace:
`kubectl create namespace argocd`

<br><br>
<br>

<br><br>
<br>

Step 3: Install ArgoCD

Install ArgoCD in Kubernetes:
`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

Check pods:
`kubectl get pods -n argocd`

Wait until all pods show:
`Running`

<br><br>
<br>

Step 4: Access ArgoCD UI

Forward port:
`kubectl port-forward svc/argocd-server -n argocd 8080:443`

Open in browser:
https://localhost:8080

<br<br><br>
<br>

Step 5: Get ArgoCD Admin Password

Retrieve and decode the initial admin password in PowerShell:

`$pass = kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($pass))`

Username:
admin


<br><br>
<br>

Step 6: Create GitOps Repository

Create repository structure:
GitOps-demo
│
└── k8s
    └── deployment.yaml

Example deployment file:
Make your Github repo with the same files


<br><br>
<br>

Step 7: Create ArgoCD Application

In ArgoCD UI:

Click New App

Fill:

Application Name:
gitops-demo

gitops-demo:
Repository URL
https://github.com/<yourusername>/GitOps-demo

Path:
k8s

Cluster:
https://kubernetes.default.svc

Namespace:
default

Enable:
Automatic Sync
Prune
Self Heal

Create application



<br><br>
<br>

Step 8: Verify Deployment
Check pods:
`kubectl get pods`

Example output:
`gitops-demo-xxxx Running
gitops-demo-xxxx Running`


<br><br>
<br>

Step 9: Test GitOps Workflow

1. Edit deployment in GitHub:
replicas: 2

Change to:
replicas: 4

Commit changes.
ArgoCD will detect the change and automatically update Kubernetes.

Verify:
`kubectl get pods`

Now you should see:
`4 pods running`

or to watch Kubernetes resources update in real time:
`kubectl get pods -w`

Example output will update automatically:
`gitops-demo-7d8f7f8b5c-abcde   Running
gitops-demo-7d8f7f8b5c-fghij   Running
gitops-demo-7d8f7f8b5c-klmno   Running`

2. Watch ArgoCD pods in real time:
`kubectl get pods -n argocd -w`

This lets you see if ArgoCD components restart or update.

3. Watch deployment updates
Useful when replicas change:
`kubectl get deployment -w`

4. Watch ArgoCD logs in real time
To see ArgoCD detecting Git changes:
`kubectl logs -n argocd deployment/argocd-application-controller -f`
<br><br>

GitOps workflow:
<pre>
Developer → Git push → Git Repository
                         ↓
                 ArgoCD detects change
                         ↓
                    Sync cluster
                         ↓
                    Kubernetes cluster
</pre>
