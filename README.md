# Common Kubernetes cluster configuration for GitOps
az group create -n aks -l westeurope
az aks create -c 1 -s Standard_B2ms -g aks -x -n aks
az aks get-credentials -n aks -g aks --admin --overwrite-existing

az extension update -n k8s-configuration
az extension update -n k8s-extension

az k8s-configuration flux create -n gitops-demo \
    -g aks \
    -c aks \
    --namespace gitops \
    -t managedClusters \
    --scope cluster \
    -u https://github.com/tomas-iac/common-kubernetes.git \
    --branch main  \
    --kustomization name=infra path=./ prune=true 

az k8s-configuration flux show -g aks -c aks -n gitops-demo -t managedClusters
    
    \
    --kustomization name=apps path=./apps/staging prune=true dependsOn=["infra"]