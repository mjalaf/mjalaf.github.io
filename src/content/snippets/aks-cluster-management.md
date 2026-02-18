---
title: "AKS Cluster Management"
description: "Azure Kubernetes Service commands for cluster operations, deployments, debugging, and scaling."
category: "AKS"
---

## Cluster Operations

```bash
# Get AKS credentials
az aks get-credentials --resource-group myRG --name myAKSCluster --overwrite-existing

# List clusters
az aks list -o table

# Show cluster info
az aks show -g myRG -n myAKSCluster -o table

# Upgrade cluster
az aks get-upgrades -g myRG -n myAKSCluster -o table
az aks upgrade -g myRG -n myAKSCluster --kubernetes-version 1.29.0

# Scale node pool
az aks nodepool scale -g myRG --cluster-name myAKSCluster -n nodepool1 -c 5
```

## kubectl Essentials

```bash
# Get all resources in a namespace
kubectl get all -n my-namespace

# Describe a failing pod
kubectl describe pod pod-name -n my-namespace

# View pod logs (previous crash included)
kubectl logs pod-name -n my-namespace --previous -f

# Exec into a pod
kubectl exec -it pod-name -n my-namespace -- /bin/sh

# Port-forward a service locally
kubectl port-forward svc/my-service 8080:80 -n my-namespace

# Get events sorted by time
kubectl get events -n my-namespace --sort-by='.lastTimestamp'

# Watch pod status changes
kubectl get pods -n my-namespace -w
```

## Deployments & Rollouts

```bash
# Apply manifests
kubectl apply -f deployment.yaml

# Rollout status
kubectl rollout status deployment/my-app -n my-namespace

# Rollback to previous version
kubectl rollout undo deployment/my-app -n my-namespace

# Scale deployment
kubectl scale deployment/my-app --replicas=5 -n my-namespace

# Set image (trigger rolling update)
kubectl set image deployment/my-app app=myregistry.azurecr.io/app:v2 -n my-namespace

# Restart deployment (rolling restart)
kubectl rollout restart deployment/my-app -n my-namespace
```

## Debugging

```bash
# Run a debug pod
kubectl run debug --rm -it --image=nicolaka/netshoot -- /bin/bash

# Check DNS resolution inside the cluster
kubectl run dns-test --rm -it --image=busybox -- nslookup my-service.my-namespace.svc.cluster.local

# Top (resource usage)
kubectl top nodes
kubectl top pods -n my-namespace --sort-by=memory

# Get pod YAML (current state)
kubectl get pod pod-name -n my-namespace -o yaml

# Check RBAC
kubectl auth can-i create pods --namespace my-namespace
```

## AKS + ACR Integration

```bash
# Attach ACR to AKS
az aks update -g myRG -n myAKSCluster --attach-acr myACR

# Verify ACR pull permission
az aks check-acr -g myRG -n myAKSCluster --acr myACR.azurecr.io

# Import an image into ACR
az acr import -n myACR --source docker.io/library/nginx:latest --image nginx:latest
```

## Helm

```bash
# Add a Helm repo
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install a chart
helm install my-ingress ingress-nginx/ingress-nginx \
  --namespace ingress --create-namespace \
  --set controller.replicaCount=2

# Upgrade a release
helm upgrade my-ingress ingress-nginx/ingress-nginx --namespace ingress -f values.yaml

# List releases
helm list -A

# Rollback
helm rollback my-ingress 1 --namespace ingress
```
