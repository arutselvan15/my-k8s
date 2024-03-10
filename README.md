# my-k8s

# Infra Setup

## Pre-requisites:

## MacOS

```
    brew install kind
    brew install kubectl
    brew install podman
```
### Podman desktop

Follow the instructions to install podman on your machine.

https://podman-desktop.io/docs/installation/macos-install

https://podman-desktop.io/docs/podman/creating-a-podman-machine

## Kubernetes cluster

### Steps:

```
    kind create cluster --name my-k8s
```

Or use the podman desktop to create a cluster.

### References:

https://podman-desktop.io/docs/kind/creating-a-kind-cluster

## Kubernetes dashboard

### Steps:

```
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
    # if above does not work, use the local file "kubectl apply -f infra-setup/dasbhoard/01-dashboard-setup.yaml"
    kubectl apply -f infra-setup/01-dasbhoard/02-create-admin-user.yaml
    kubectl -n kubernetes-dashboard create token admin-user
    # use the token to login in the dashboard
    kubectl proxy
```

Kubectl will make Dashboard available at http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

### References:

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

## ArgoCD

```
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    # if above does not work use the local file "kubectl apply -n argocd -f infra-setup/02-argocd/01-argo-cd-setup.yaml"

    kubectl port-forward svc/argocd-server -n argocd 8080:443
    # user name: admin
    # password: kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## GitOps setup

```
    kubectl apply -f infra-setup/03-gitops-setup/01-repo-creds.yaml
```

### References:

https://argo-cd.readthedocs.io/en/stable/getting_started/