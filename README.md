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

## MetalLB setup

```
    kubectl apply -f infra-setup/04-metallb/01-metal-lb-setup.yaml
    
    kubectl wait --namespace metallb-system \
                --for=condition=ready pod \
                --selector=app=metallb \
                --timeout=90s
    # docker network inspect -f '{{.IPAM.Config}}' kind

    podman network inspect -f '{{range .Subnets}}{{if eq (len .Subnet.IP) 4}}{{.Subnet}}{{end}}{{end}}' kind
    # this willl give you the IP address pool used by loadbalancers
    # example: 10.89.0.0/24
    # IMPORTANT: set the IP address pool in the file "infra-setup/04-metallb/02-metallb-ip-addr-pool.yaml"
    # example: 10.89.255.200-10.89.255.250
    
    kubectl apply -f infra-setup/04-metallb/02-metallb-ip-addr-pool.yaml
    
    # test the setup by creating svc with type loadbalancer and see the external IP
    kubectl apply -f apps/sample-app/service-lb.yaml
    $] kubectl get svc -o wide -n sample-app         
    NAME            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE     SELECTOR
    sample-app      ClusterIP      10.96.61.31    <none>          5000/TCP         18d     app=sample-app
    sample-app-lb   LoadBalancer   10.96.23.132   10.89.255.201   5000:32499/TCP   2m20s   app=sample-app

```
https://kind.sigs.k8s.io/docs/user/loadbalancer/


## Istio setup
    
    ```
        brew install istioctl

        kubectl create namespace istio-system
        istioctl install --set profile=demo -y

        # testing the setup
        kubectl create namespace bookinfo
        kubectl label namespace bookinfo istio-injection=enabled
        kubectl apply -f infra-setup/05-istio/01-bookinfo.yaml -n bookinfo

        # wait for pods to be ready and test below command
        kubectl exec "$(kubectl get pod -n bookinfo -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -n bookinfo -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

        kubectl apply -f infra-setup/05-istio/02-bookinfo-gateway.yaml -n bookinfo

        # check if any issue detected by istio
        istioctl analyze -n bookinfo
    ```
### References:

https://istio.io/latest/docs/setup/platform-setup/kind/