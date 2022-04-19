# GitOps for Everything

This repo is showcasing: 
1. infrastructure, managed service and application deployment based on GitOps principles, and 
2. a split of responsibility between Dev and Platform teams.

# Installation

## Pre-requisites

If you want to reproduce this setup, then follow the full installation steps
- Clone this repo and update all the repo URLs accordingly
- Create a management cluster.
- Configure proxy in /etc/environment with proxy and add your cluster IP ranges to no_proxy
- Connect to the vpn

## Steps

0. # Setup proxy environment for argocd

## Retrieve the standard values.yaml from the helm repo
```
helm inspect values argo/argo-cd > values.yaml
```

## Customize it with proxy settings
```
./merge.sh values.yaml proxy.yaml > values-proxy.yaml
```

1. Install ArgoCD in the management cluster
```
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm upgrade --install argocd argo/argo-cd \
  --namespace argocd --create-namespace \
  --values proxy/values-proxy.yaml \
  --wait
```

2. Login and change the default password
```
export PASS=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
argocd login --insecure --username admin --password $PASS --grpc-web argo-cd.$INGRESS_HOST.nip.io
argocd account update-password --current-password $PASS --new-password admin123 --grpc-web
```

3. Provide Gitlab credentials (deploy token This is an example)

Log in Gitlab, create a [deploy token](https://docs.gitlab.com/ee/user/project/deploy_tokens/) and then run
```
argocd repo add https://gitlabe2.ext.net.nokia.com/cns-ba-ta-as/gitops-everything.git \
  --username <username> --password <token> --project production --insecure-skip-server-verification
```
Alternatively the repo connection details can be 
[configured declaratively](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup).


4. Open ArgoCD in your browser

Assuming port forwarding from 80 to 8080
```
http://argo-cd.127.0.0.1.nip.io:8080/applications
```

5. Create the ArgoCD project and production applications
```
kubectl apply --filename argocd/project.yaml
kubectl apply --filename argocd/apps.yaml
```
