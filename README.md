# Common Kubernetes cluster configuration for GitOps

az aks get-credentials -n aks -g aks --admin

az k8s-configuration flux create -n gitops-demo \
    -g aks \
    -c aks \
    --namespace gitops \
    -t managedClusters \
    --scope cluster 
    -u https://github.com/tomas-iac/common-kubernetes.git \
    --branch main  \
    --kustomization name=infra path=. prune=true #\
    #--kustomization name=apps path=./apps/staging prune=true dependsOn=["infra"]