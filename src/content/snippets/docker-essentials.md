---
title: "Docker Essentials"
description: "Common Docker commands for container management, image building, networking, and troubleshooting."
category: "Docker"
---

## Container Lifecycle

```bash
# Run a container (detached, with restart policy)
docker run -d --name myapp --restart unless-stopped -p 8080:80 myimage:latest

# Execute a command inside a running container
docker exec -it container_name /bin/bash

# View container logs (follow + timestamps)
docker logs -f --tail 100 --timestamps container_name

# Inspect container details
docker inspect container_name | jq '.[0].NetworkSettings'

# Copy files to/from container
docker cp localfile.txt container_name:/path/in/container/
docker cp container_name:/path/in/container/file.txt ./local/
```

## Image Management

```bash
# Build with build args and no cache
docker build --no-cache --build-arg ENV=production -t myapp:1.0 .

# Multi-stage build (Dockerfile)
# Stage 1: Build
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:22-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

```bash
# Tag and push to registry
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0
docker push myregistry.azurecr.io/myapp:1.0

# Prune unused images
docker image prune -a --filter "until=168h"  # older than 7 days

# Save/load images (offline transfer)
docker save myapp:1.0 | gzip > myapp-1.0.tar.gz
docker load < myapp-1.0.tar.gz
```

## Docker Compose

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: mydb
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

```bash
# Compose commands
docker compose up -d
docker compose logs -f app
docker compose down -v         # remove volumes too
docker compose ps
docker compose exec app sh
```

## Cleanup

```bash
# Nuclear cleanup (remove everything unused)
docker system prune -a --volumes

# Remove stopped containers
docker container prune

# Remove dangling images
docker image prune

# Check disk usage
docker system df
```

## Troubleshooting

```bash
# Check container resource usage
docker stats --no-stream

# View container events
docker events --since "1h"

# Debug networking
docker run --rm --net container:target_container nicolaka/netshoot

# Check container health
docker inspect --format='{{.State.Health.Status}}' container_name
```
