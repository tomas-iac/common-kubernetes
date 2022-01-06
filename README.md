# Common Kubernetes cluster configuration for GitOps
```
# Create resource group
az group create -n aks -l westeurope

# Create VNETs
az network vnet create -n project1-prod-vnet -g aks --address-prefixes 10.0.0.0/16
az network vnet subnet create -n aks-subnet -g aks --vnet-name project1-prod-vnet --address-prefix 10.0.0.0/24
az network vnet subnet create -n lb-subnet -g aks --vnet-name project1-prod-vnet --address-prefix 10.0.1.0/24
az network vnet create -n project1-staging-vnet -g aks --address-prefixes 10.1.0.0/16
az network vnet subnet create -n aks-subnet -g aks --vnet-name project1-staging-vnet --address-prefix 10.1.0.0/24
az network vnet subnet create -n lb-subnet -g aks --vnet-name project1-staging-vnet --address-prefix 10.1.1.0/24
az network vnet create -n project2-prod-vnet -g aks --address-prefixes 10.2.0.0/16
az network vnet subnet create -n aks-subnet -g aks --vnet-name project2-prod-vnet --address-prefix 10.2.0.0/24
az network vnet subnet create -n lb-subnet -g aks --vnet-name project2-prod-vnet --address-prefix 10.2.1.0/24

# Create clusters
az aks create -n project1-prod-aks \
    -c 1 \
    -s Standard_B2ms \
    -g aks \
    -x \
    --network-plugin azure \
    --vnet-subnet-id $(az network vnet subnet show -n aks-subnet -g aks --vnet-name project1-prod-vnet --query id -o tsv) \
    --service-cidr 172.16.0.0/22 \
    --dns-service-ip 172.16.0.10 \
    -y --no-wait
az aks create -n project1-staging-aks \
    -c 1 \
    -s Standard_B2ms \
    -g aks \
    -x \
    --network-plugin azure \
    --vnet-subnet-id $(az network vnet subnet show -n aks-subnet -g aks --vnet-name project1-staging-vnet --query id -o tsv) \
    --service-cidr 172.16.0.0/22 \
    --dns-service-ip 172.16.0.10 \
    -y --no-wait
az aks create -n project2-prod-aks \
    -c 1 \
    -s Standard_B2ms \
    -g aks \
    -x \
    --network-plugin azure \
    --vnet-subnet-id $(az network vnet subnet show -n aks-subnet -g aks --vnet-name project2-prod-vnet --query id -o tsv) \
    --service-cidr 172.16.0.0/22 \
    --dns-service-ip 172.16.0.10 \
    -y --no-wait

# Get credentials
az aks get-credentials -n project1-prod-aks -g aks --admin --overwrite-existing
az aks get-credentials -n project1-staging-aks -g aks --admin --overwrite-existing
az aks get-credentials -n project2-prod-aks -g aks --admin --overwrite-existing

# Setup GitOps configurations
## Get CLI extensions
az extension update -n k8s-configuration
az extension update -n k8s-extension

## Create baseline cluster config
az k8s-configuration flux create -c project1-prod-aks \
    -n baseline \
    -g aks \
    --namespace gitops \
    -t managedClusters \
    --scope cluster \
    -u https://github.com/tomas-iac/common-kubernetes.git \
    --branch main  \
    --interval 60s \
    --kustomization name=infra path=/clusters/project1-prod-aks prune=true sync_interval=60s retry_interval=60s
az k8s-configuration flux create -c project1-staging-aks \
    -n baseline \
    -g aks \
    --namespace gitops \
    -t managedClusters \
    --scope cluster \
    -u https://github.com/tomas-iac/common-kubernetes.git \
    --branch main  \
    --interval 60s \
    --kustomization name=infra path=/clusters/project1-staging-aks prune=true sync_interval=60s retry_interval=60s
az k8s-configuration flux create -c project2-prod-aks \
    -n baseline \
    -g aks \
    --namespace gitops \
    -t managedClusters \
    --scope cluster \
    -u https://github.com/tomas-iac/common-kubernetes.git \
    --branch main  \
    --interval 60s \
    --kustomization name=infra path=/clusters/project2-prod-aks prune=true sync_interval=60s retry_interval=60s

```