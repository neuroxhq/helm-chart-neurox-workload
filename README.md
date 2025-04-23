# Neurox Workload Helm Chart

The Neurox Workload management cluster is where GPU workloads run on GPU nodes. When deployed standalone, it does not require ingress nor persistent disk. Typically, the Neurox Workload components are installed together with Neurox Control plane components in a single combined Kubernetes cluster.

This outlines the requirements needed to deploy standalone Neurox Workload components into additional Kubernetes GPU clusters. Neurox Workload can autodetect many Cloud Service Provider (CSP) environments, automatically surfacing metadata such as region or availability zone, as well as identify models of GPUs attached.

## Multi-Cluster setup
One of the best features of Neurox is to monitor multiple Neurox Workload clusters from a single Neurox Control plane. Common use cases include joining GPU clusters from various cloud providers or even on-prem clusters.
Please see our [pricing plans](https://neurox.com/pricing) to determine how many Neurox Workload clusters may be joined into a Neurox Control cluster.

## Cluster requirements
- Kubernetes and CLI 1.29+
- Helm CLI 3.8+
- 4 CPUs
- 8 GB of RAM
- At least 1 GPU node

## Prerequisites

You will need both __NVIDIA GPU Operator__ and __Kube Prometheus Stack__ to run the Neurox workload chart.

__NVIDIA GPU Operator__

Required to run GPU workloads. Install with:
```
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update
helm install --create-namespace -n gpu-operator gpu-operator nvidia/gpu-operator --version=v25.3.0
```
For more information on how to configure NVIDIA GPU operator: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html#procedure

__Kube Prometheus Stack__

Required to gather metrics. Install with:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
# This is the minimum required configuration. Feel free to enable components if you need them.
helm install --create-namespace -n monitoring kube-prometheus-stack prometheus-community/kube-prometheus-stack --set alertmanager.enabled=false --set grafana.enabled=false --set prometheus.enabled=false
```
For more information on how to configure kube-prometheus-stack: https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-state-metrics

## Credentials
- Your Neurox subdomain
- Your Neurox Workload auth secret (provided by Neurox Control)
- Your Neurox registry username and password

## Example install
To join a Neurox Workload cluster to an existing Neurox Control cluster, you can obtain the install script by going to your Neurox Control portal > Clusters > New Cluster button and a fully generated install script (with the auth secret) will be available to copy/paste.

The example below was based on the output of the generated install script:
```
CLUSTER_NAME=my-workload-cluster-name # customize this

NEUROX_DOMAIN=replace-this.goneurox.com
WORKLOAD_AUTH_SECRET=yourworkloadauthsecret
NEUROX_HELM_REGISTRY=oci://ghcr.io/neuroxhq/helm-charts
NEUROX_IMAGE_REGISTRY=registry.neurox.com
NEUROX_USERNAME=replace-this-goneurox-com
NEUROX_PASSWORD=yourregistrypassword

kubectl create ns neurox
kubectl create secret generic -n neurox neurox-control-auth --from-literal=shared-secret=${WORKLOAD_AUTH_SECRET}
kubectl create secret docker-registry -n neurox neurox-image-registry --docker-server=${NEUROX_IMAGE_REGISTRY} --docker-username=${NEUROX_USERNAME} --docker-password=${NEUROX_PASSWORD}

helm install neurox-workload ${NEUROX_HELM_REGISTRY}/neurox-workload --namespace neurox --set global.workloadCluster.name=${CLUSTER_NAME} --set global.controlHost=${NEUROX_DOMAIN}
```
