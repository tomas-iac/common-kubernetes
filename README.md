# Common Kubernetes cluster configuration for GitOps
This repo is referenced by AKS GitOps solution with Flux v2 to manage common Kubernetes cluster configuration and deployment of services on top such as NGINX Ingress controller, cert-manager or KEDA scaler. To demonstrate ability to start from shared base (to get things DRY - Do Not Repeate Yourself) we go from there to overlay with environments and each environment is in our case than overlayd with individual cluster (in our demo to setup fixed external Ingress IP - example of something tham must be cluster specific).

See full context at [https://github.com/tomas-iac/docs](https://github.com/tomas-iac/docs)

