# Helm Chart for sample-webapp

This Helm chart deploys the sample-webapp Docker image to a Kubernetes cluster.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- Azure Container Registry (ACR) credentials configured

## Installation

### 1. Add the Helm Repository

```bash
helm repo add myrepo https://your-helm-repo-url
helm repo update
```

### 2. Install the Chart

```bash
helm install sample-webapp myrepo/sample-webapp \
  --namespace default \
  --create-namespace \
  --set image.repository=acr007.azurecr.io/sample-webapp \
  --set image.tag=<IMAGE_TAG>
```

### 3. Verify Deployment

```bash
kubectl get pods -n default
kubectl get svc -n default
```

## Configuration

The following table lists the configurable parameters of the sample-webapp chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `2` |
| `image.repository` | Image repository | `acr007.azurecr.io/sample-webapp` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `service.type` | Kubernetes Service type | `LoadBalancer` |
| `service.port` | Service port | `80` |
| `autoscaling.enabled` | Enable HPA | `true` |
| `autoscaling.minReplicas` | Minimum replicas | `2` |
| `autoscaling.maxReplicas` | Maximum replicas | `5` |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU utilization | `80` |

## Kubernetes Cluster Setup

### Azure AKS Cluster Creation

```bash
# Set variables
RESOURCE_GROUP="myResourceGroup"
CLUSTER_NAME="myAKSCluster"
LOCATION="eastus"
NODE_COUNT=2
VM_SKU="Standard_D2s_v3"

# Create resource group
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION

# Create AKS cluster
az aks create \
  --resource-group $RESOURCE_GROUP \
  --name $CLUSTER_NAME \
  --node-count $NODE_COUNT \
  --vm-set-type VirtualMachineScaleSets \
  --load-balancer-sku standard \
  --enable-managed-identity \
  --network-plugin azure \
  --network-policy azure \
  --docker-bridge-address 172.17.0.1/16 \
  --service-cidr 10.0.0.0/16 \
  --dns-service-ip 10.0.0.10 \
  --node-vm-size $VM_SKU \
  --enable-cluster-autoscaling \
  --min-count 1 \
  --max-count 5

# Get kubeconfig
az aks get-credentials \
  --resource-group $RESOURCE_GROUP \
  --name $CLUSTER_NAME \
  --overwrite-existing

# Verify cluster
kubectl cluster-info
kubectl get nodes
```

### Configure ACR Access

```bash
# Attach ACR to AKS cluster
ACR_NAME="acr007"
az aks update \
  --name $CLUSTER_NAME \
  --resource-group $RESOURCE_GROUP \
  --attach-acr $ACR_NAME
```

## Uninstall

```bash
helm uninstall sample-webapp --namespace default
```

## Troubleshooting

### Check pod status
```bash
kubectl describe pod <pod-name> -n default
kubectl logs <pod-name> -n default
```

### Check deployment status
```bash
kubectl describe deployment sample-webapp -n default
kubectl rollout status deployment/sample-webapp -n default
```

### Check HPA status
```bash
kubectl get hpa sample-webapp -n default
kubectl describe hpa sample-webapp -n default
```
