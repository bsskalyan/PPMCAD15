# Docker Commands Cheat Sheet

Quick reference for all Docker commands and concepts covered in training.

---

## Container Lifecycle

### Running Containers

```bash
# Basic run
docker run <image>

# Run in detached mode (background)
docker run -d <image>

# Run with port mapping (host:container)
docker run -p 8080:80 <image>

# Run with name
docker run --name my-container <image>

# Run with environment variables
docker run -e KEY=value <image>
docker run -e DB_HOST=postgres -e DB_PORT=5432 <image>

# Run with env file
docker run --env-file .env <image>

# Run with volume mount (named volume)
docker run -v my-volume:/data <image>

# Run with bind mount (development)
docker run -v /host/path:/container/path <image>
docker run -v $(pwd):/app <image>

# Run with resource limits
docker run --memory="512m" --cpus="1.0" <image>

# Run as specific user
docker run --user 1000:1000 <image>

# Run with custom network
docker run --network my-network <image>

# Run with restart policy
docker run --restart unless-stopped <image>
# Options: no, on-failure, always, unless-stopped

# Run with health check override
docker run --health-cmd="curl -f http://localhost/" <image>

# Auto-remove container after exit
docker run --rm <image>

# Interactive terminal
docker run -it <image> /bin/bash

# Complete example
docker run -d \
  --name web-app \
  --network frontend-net \
  -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html \
  -e ENV=production \
  --memory="512m" \
  --cpus="1.0" \
  --restart unless-stopped \
  nginx:alpine
```

### Managing Containers

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container gracefully (SIGTERM)
docker stop <container-id or name>

# Stop with timeout
docker stop -t 30 <container-id>

# Kill container immediately (SIGKILL)
docker kill <container-id or name>

# Start a stopped container
docker start <container-id or name>

# Restart a container
docker restart <container-id or name>

# Pause container (freeze processes)
docker pause <container-id or name>

# Unpause container
docker unpause <container-id or name>

# Rename container
docker rename old-name new-name

# Wait for container to stop
docker wait <container-id or name>

# Remove a container
docker rm <container-id or name>

# Remove a running container (force)
docker rm -f <container-id or name>

# Remove multiple containers
docker rm container1 container2 container3

# Remove all stopped containers
docker container prune
```

### Interacting with Containers

```bash
# View container logs
docker logs <container-id or name>

# Follow logs in real-time
docker logs -f <container-id or name>

# Show last N lines
docker logs --tail 100 <container-id or name>

# Show logs with timestamps
docker logs -t <container-id or name>

# Show logs since specific time
docker logs --since 10m <container-id or name>
docker logs --since 2024-01-01T00:00:00 <container-id or name>

# Execute command in running container
docker exec <container-id or name> <command>

# Execute as specific user
docker exec -u root <container-id or name> <command>

# Get interactive shell
docker exec -it <container-id or name> /bin/bash
# For alpine-based images
docker exec -it <container-id or name> /bin/sh

# Run command with environment variable
docker exec -e VAR=value <container-id or name> <command>

# View container stats (CPU, memory, I/O)
docker stats <container-id or name>

# View running processes in container
docker top <container-id or name>

# View container filesystem changes
docker diff <container-id or name>

# Copy files from container to host
docker cp <container-id>:/path/in/container /path/on/host

# Copy files from host to container
docker cp /path/on/host <container-id>:/path/in/container

# Copy directory recursively
docker cp -a <container-id>:/app /backup

# Export container filesystem as tar
docker export <container-id> > container.tar
```

---

## Image Management

### Working with Images

```bash
# List images
docker images

# List with digests
docker images --digests

# List image IDs only
docker images -q

# Pull an image from registry
docker pull <image>:<tag>

# Pull specific version
docker pull nginx:1.21-alpine

# Pull all tags of an image
docker pull -a nginx

# Build image from Dockerfile
docker build -t <image-name>:<tag> .

# Build with custom Dockerfile name
docker build -t <image-name> -f Dockerfile.custom .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t <image-name> .

# Build without cache
docker build --no-cache -t <image-name> .

# Build and tag multiple tags
docker build -t myapp:latest -t myapp:1.0 -t myapp:stable .

# Build specific target in multi-stage
docker build --target production -t <image-name> .

# Tag an image
docker tag <source-image> <target-image>:<tag>
docker tag myapp:latest myusername/myapp:1.0

# View image history (layers)
docker history <image>

# Inspect image details
docker inspect <image>

# Remove an image
docker rmi <image-id or name>

# Force remove image
docker rmi -f <image-id or name>

# Remove multiple images
docker rmi image1 image2 image3

# Remove unused images (dangling)
docker image prune

# Remove all unused images
docker image prune -a

# Remove images older than X hours
docker image prune -a --filter "until=24h"
```

### Docker Hub Operations

```bash
# Login to Docker Hub
docker login

# Login to specific registry
docker login registry.example.com

# Login with credentials
docker login -u username -p password

# Logout from Docker Hub
docker logout

# Push image to Docker Hub
docker push <username>/<image>:<tag>

# Push all tags
docker push -a <username>/<image>

# Search for images
docker search <term>

# Search with filters
docker search --filter "stars=100" nginx

# Pull from private registry
docker pull registry.example.com/image:tag
```
---

## Dockerfile Instructions

### Essential Instructions

```dockerfile
# Base image (ALWAYS first instruction)
FROM <image>:<tag>
FROM node:16-alpine
FROM python:3.9-slim AS builder

# Set working directory
WORKDIR /app

# Copy files from host to image
COPY <src> <dest>
COPY package.json .
COPY . /app
COPY --chown=user:group app.py /app/

# Add files (can extract archives and fetch URLs)
ADD <src> <dest>
ADD archive.tar.gz /app/
ADD https://example.com/file.txt /app/

# Run commands during build
RUN <command>
RUN apt-get update && apt-get install -y curl
RUN npm install
RUN pip install --no-cache-dir -r requirements.txt

# Set environment variables (runtime)
ENV KEY=value
ENV NODE_ENV=production PORT=3000
ENV PATH=/app/bin:$PATH

# Build arguments (build-time only)
ARG VERSION=1.0
ARG BUILD_DATE
FROM node:${VERSION}
RUN echo "Building version ${VERSION}"

# Expose ports (documentation only)
EXPOSE <port>
EXPOSE 80 443
EXPOSE 8080/tcp

# Define mount points
VOLUME ["/data"]
VOLUME /var/log /var/db

# Switch user (IMPORTANT for security!)
USER <username>
USER node
USER 1000:1000

# Set labels (metadata)
LABEL maintainer="dev@example.com"
LABEL version="1.0"
LABEL description="My application"

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

# Health check options
HEALTHCHECK CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1
HEALTHCHECK NONE  # Disable inherited health check

# Set default command (can be overridden)
CMD ["executable", "param1", "param2"]

# Set entrypoint (harder to override)
ENTRYPOINT ["executable"]
```

## Volumes

### Volume Commands

```bash
# List all volumes
docker volume ls

# List with filter
docker volume ls --filter "dangling=true"

# Create a named volume
docker volume create my-volume

# Create with driver options
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.1,rw \
  --opt device=:/path/to/dir \
  my-nfs-volume

# Inspect volume details
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Remove multiple volumes
docker volume rm vol1 vol2 vol3

# Remove all unused volumes
docker volume prune

# Remove volumes older than X hours
docker volume prune --filter "until=24h"
```
---

## Networking

### Network Commands

```bash
# List networks
docker network ls

# Create network
docker network create my-network

# Create with specific driver
docker network create --driver bridge my-network

# Create with subnet
docker network create --subnet=172.18.0.0/16 my-network

# Create internal network (no external access)
docker network create --internal backend-network

# Create with custom gateway
docker network create \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  my-network

# Connect container to network
docker network connect my-network container-name

# Connect with IP address
docker network connect --ip 172.20.0.10 my-network container-name

# Connect with alias
docker network connect --alias db my-network postgres-container

# Disconnect container from network
docker network disconnect my-network container-name

# Inspect network (see connected containers)
docker network inspect my-network

# Remove network
docker network rm my-network

# Remove all unused networks
docker network prune
```

## System & Cleanup

### System Information

```bash
# Display system-wide information
docker info

# Show Docker version
docker version

# Show Docker disk usage
docker system df

# Detailed disk usage
docker system df -v

# Show real-time events
docker events

# Show events with filter
docker events --filter 'type=container'

# Show resource usage
docker stats

# Show resource usage for all containers
docker stats --all
```

### Cleanup Commands

```bash
# Remove all stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove all unused images (not just dangling)
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove unused build cache
docker builder prune

# Remove everything unused
docker system prune

# Remove everything (including volumes)
docker system prune -a --volumes

# Remove with filters
docker system prune --filter "until=24h"
docker image prune --filter "until=168h"  # 1 week

# Force removal (no confirmation)
docker system prune -f
```
---