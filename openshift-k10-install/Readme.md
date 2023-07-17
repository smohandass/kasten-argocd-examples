
# Follow instructions to install Kasten using argoCD on Openshift environment.

Step 1 - Install argocd
https://argo-cd.readthedocs.io/en/stable/getting_started/

Step 2 - install argocd cli
https://argo-cd.readthedocs.io/en/stable/cli_installation/

step 3 - Login Using The CLI

Get the argocd initial password 
argocd admin initial-password -n argocd

argocd login openshift-gitops-server-openshift-gitops.apps.bm2.redhat.hpecic.net --username admin --password <password>

Step 4 - create an application

Create An Application From A Git Repository

argocd app create kasten -f argocd-create.yaml -p $NODE_NAME=node3 $VALUE_FILE=values/values.yaml $REPO_PATH=k8s-helm
