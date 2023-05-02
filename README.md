# My-Argo-CD-project
Create a new cluster


kind create cluster --name argocd
Let's create a namespace for agrocd


kubectl create namespace argocd
And install the CRDs that Argo is based on


kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.0.0-rc3/
Argo is installed as CRD inside our cluster


kubectl get all -n argocd
Now that we have that, we want to access the UI.


kubectl port-forward svc/argocd-server -n argocd 8080:443
You can get the password through the following command. Make sure that you port-forwarded beforehand.


kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
And then log into


argocd login localhost:8080
This will allow us to update our password — the username is "admin"


argocd account update-password
We will open localhost next and log into the UI.

Let's deploy an application. First, through the UI and then through the CLI. You can install the Argo CD CLI through the following commands:


VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
Here is the documentation: https://argoproj.github.io/argo-cd/cli_installation/

We are going to be using the following repository

https://github.com/AnaisUrlichs/react-article-display

Now we will go ahead and tell the Argo UI about my Helm Chart. If you already have a repository with a Helm Chart, YAML files, or any other YAML manifests, you could use that instead.

Alternatively, you can tell Argo CD about the repository through the following command or similar:


argocd app create react-app --repo https://github.com/anais-codefresh/react-article-display.git --path charts/example-chart --dest-server https://kubernetes.default.svc --dest-namespace default
Once the app has been deployed, we can take a look


kubectl get all

argocd app get react-app
We can also ensure that our desired state is synced with the actual state in our cluster


argocd app sync react-app
So what happens if we make any manual changes with the resources within our cluster?

Let's go ahead and patch the service into type NodePort


kubectl patch svc react-app-example-chart --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'
And check the status again


argocd app get react-app
