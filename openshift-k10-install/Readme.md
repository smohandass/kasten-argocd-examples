
# Follow instructions to install Kasten using argoCD on Openshift environment.

Step 1 - Install argocd
https://argo-cd.readthedocs.io/en/stable/getting_started/

Step 2 - install argocd cli
https://argo-cd.readthedocs.io/en/stable/cli_installation/

step 3 - Login Using The CLI

Get the argocd initial password 
argocd admin initial-password -n argocd

argocd login openshift-gitops-server-openshift-gitops.apps.bm2.redhat.hpecic.net --username admin --password <password>

Step 5 - Login to the Openshift cluster using oc login command

Step 6 - 

```
kubectl create namespace kasten-io
```

create the Service Account


```
cat<<EOF | kubectl create -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: k10-dex-sa
  namespace: kasten-io
  annotations:
    serviceaccounts.openshift.io/oauth-redirecturi.dex: https://k10-route-kasten-io.apps.bm2.redhat.hpecic.net/k10/dex/callback
EOF
```

Create the secret 

```
cat<<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: k10-dex-sa-secret
  namespace: kasten-io
  annotations:
    kubernetes.io/service-account.name: "k10-dex-sa"
EOF
```

Retrieve the token for the secret

```
my_token=$(oc -n kasten-io get secret k10-dex-sa-secret -o jsonpath='{.data.token}' | base64 -d)
echo $my_token
```

Create configmap

```
oc get secret router-ca -n openshift-ingress-operator -o jsonpath='{.data.tls\.crt}'' | base64 -d > custom-ca-bundle.pem

oc --namespace kasten-io create configmap custom-ca-bundle-store --from-file=custom-ca-bundle.pem
```

Add role to user

```
oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n kasten-io
```

Step 7 - create an application

```
argocd app create kasten \
      --app-namespace openshift-gitops \
      --dest-namespace kasten-io \
      --dest-server https://kubernetes.default.svc \
      --project default \
      --repo https://github.com/smohandass/kasten-argocd-examples.git \
      --path openshift-k10-install  \
      --revision main \
      --directory-recurse \
      --sync-policy automated \
      --self-heal
```

