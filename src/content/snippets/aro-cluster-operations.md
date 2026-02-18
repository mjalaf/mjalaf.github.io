---
title: "ARO Cluster Operations"
description: "Azure Red Hat OpenShift commands for cluster management, project administration, and deployments."
category: "ARO"
---

## Cluster Setup & Access

```bash
# Create an ARO cluster
az aro create \
  --resource-group myRG \
  --name myAROCluster \
  --vnet myVNet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet \
  --pull-secret @pull-secret.txt

# Get cluster credentials
az aro list-credentials -g myRG -n myAROCluster

# Get API server URL
az aro show -g myRG -n myAROCluster --query apiserverProfile.url -o tsv

# Get console URL
az aro show -g myRG -n myAROCluster --query consoleProfile.url -o tsv

# Login with oc CLI
oc login <API_SERVER_URL> -u kubeadmin -p <PASSWORD>
```

## Project (Namespace) Management

```bash
# Create a new project
oc new-project my-app-project --display-name="My App"

# Switch project
oc project my-app-project

# List all projects
oc get projects

# Delete a project
oc delete project my-app-project
```

## Deployments

```bash
# Deploy from image
oc new-app myregistry.azurecr.io/myapp:latest --name=myapp

# Expose service as route (HTTPS)
oc expose svc/myapp
oc create route edge myapp --service=myapp --port=8080

# Scale deployment
oc scale dc/myapp --replicas=3

# Set environment variables
oc set env dc/myapp NODE_ENV=production DB_HOST=mydb

# Trigger a new deployment
oc rollout latest dc/myapp

# Rollback
oc rollout undo dc/myapp
```

## Build Configurations

```bash
# Create build from Dockerfile in Git repo
oc new-build https://github.com/user/repo.git --name=myapp --strategy=docker

# Start a build
oc start-build myapp --follow

# View build logs
oc logs build/myapp-1

# List builds
oc get builds
```

## Troubleshooting

```bash
# Check pod events
oc describe pod pod-name

# View logs
oc logs pod-name -f --previous

# Remote shell into pod
oc rsh pod-name

# Debug a failing pod
oc debug pod-name

# Check cluster operators status
oc get co

# Node status
oc get nodes -o wide
oc adm top nodes
```

## Security & RBAC

```bash
# Grant edit role to a user
oc adm policy add-role-to-user edit user@example.com -n my-project

# Allow containers to run as root (not recommended for prod)
oc adm policy add-scc-to-user anyuid -z default -n my-project

# View security context constraints
oc get scc

# Check who can perform an action
oc adm policy who-can create pods -n my-project
```
