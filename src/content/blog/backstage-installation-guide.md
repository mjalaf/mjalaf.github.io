---
title: 'Backstage on Linux: Step-by-step Install + Catalog Components + Troubleshooting'
description: 'A practical guide to install Backstage on Ubuntu/Linux, define catalog components, and fix common issues.'
pubDate: 'Dec 11 2025'
---

# Backstage on Linux (Ubuntu): Install Guide + Component Definitions + Troubleshooting

This guide walks you through installing Backstage on a clean Linux
machine (Ubuntu recommended), running it locally or on your LAN, and
defining your first Software Catalog components.

------------------------------------------------------------------------

## Prerequisites

Recommended baseline: - Ubuntu 22.04 / 24.04 - 4 GB RAM minimum (8 GB
recommended) - Git + Node.js LTS + build tools - Optional: Docker

------------------------------------------------------------------------

## 1) System packages

``` bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y   git curl ca-certificates   build-essential make g++ pkg-config   python3 python-is-python3   libsqlite3-dev
```

Verify:

``` bash
git --version
python --version
g++ --version
```

------------------------------------------------------------------------

## 2) Install Node.js using nvm

``` bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

nvm install 20.11.1
nvm use 20.11.1
nvm alias default 20.11.1

node -v
npm -v
```

------------------------------------------------------------------------

## 3) Enable Corepack

``` bash
corepack enable
```

------------------------------------------------------------------------

## 4) Create the Backstage app

``` bash
mkdir -p ~/projects
cd ~/projects

npx @backstage/create-app@latest
```

Enter the app name:

    backstage

Then:

``` bash
cd backstage
```

------------------------------------------------------------------------

## 5) Disable Node snapshot if needed

``` bash
export NODE_OPTIONS="--no-node-snapshot"
```

Optional permanent setup:

``` bash
echo 'export NODE_OPTIONS="--no-node-snapshot"' >> ~/.bashrc
source ~/.bashrc
```

------------------------------------------------------------------------

## 6) Run Backstage

``` bash
yarn start
```

Frontend: http://localhost:3000

Backend: http://localhost:7007

------------------------------------------------------------------------

## 7) Expose Backstage to your LAN

Start:

``` bash
HOST=0.0.0.0 yarn start
```

Find IP:

``` bash
ip a
```

Open from another machine:

    http://YOUR_LINUX_IP:3000

Update app-config.yaml:

``` yaml
app:
  baseUrl: http://YOUR_LINUX_IP:3000

backend:
  baseUrl: http://YOUR_LINUX_IP:7007
  listen:
    host: 0.0.0.0
    port: 7007
  cors:
    origin: http://YOUR_LINUX_IP:3000
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
```

Restart:

``` bash
HOST=0.0.0.0 yarn start
```

------------------------------------------------------------------------

# Software Catalog: Component Definitions

## Component Example

``` yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: payments-service
  description: Handles payment processing
  tags:
    - dotnet
    - api
spec:
  type: service
  lifecycle: production
  owner: group:platform-team
  system: payments-platform
```

## System Example

``` yaml
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: payments-platform
spec:
  owner: group:platform-team
  domain: commerce
```

## Domain Example

``` yaml
apiVersion: backstage.io/v1alpha1
kind: Domain
metadata:
  name: commerce
spec:
  owner: group:platform-team
```

## Resource Example

``` yaml
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: payments-db
spec:
  type: database
  owner: group:platform-team
  system: payments-platform
```

## API Example

``` yaml
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: payments-api
spec:
  type: openapi
  lifecycle: production
  owner: group:platform-team
  system: payments-platform
  definition:
    $text: https://raw.githubusercontent.com/YOUR_ORG/YOUR_REPO/main/openapi.yaml
```

------------------------------------------------------------------------

# Troubleshooting

## TypeError: Failed to fetch

Ensure backend.baseUrl and app.baseUrl are set correctly and backend
listens on 0.0.0.0.

## Backend not listening on 7007

``` bash
ss -tulnp | grep -E '3000|7007'
```

Check backend logs.

## Native module build errors

Install build tools and verify Node version:

``` bash
sudo apt install -y build-essential python3 make g++ pkg-config
node -v
```

Remove node_modules and reinstall:

``` bash
rm -rf node_modules
yarn install
```

## Permission issues

``` bash
sudo chown -R $USER:$USER backstage
```

## Firewall issues

``` bash
sudo ufw allow 3000
sudo ufw allow 7007
```

------------------------------------------------------------------------

Backstage is now installed and ready for further customization and
integration.
