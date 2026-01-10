# Docker Tutorial - Complete Reference Guide

## Table of Contents
1. [Introduction to Docker](#introduction-to-docker)
2. [Docker Installation](#docker-installation)
3. [Docker Architecture](#docker-architecture)
4. [Basic Docker Commands](#basic-docker-commands)
5. [Working with Images](#working-with-images)
6. [Working with Containers](#working-with-containers)
7. [Dockerfile Deep Dive](#dockerfile-deep-dive)
8. [Docker Compose](#docker-compose)
9. [Docker Networking](#docker-networking)
10. [Docker Volumes](#docker-volumes)
11. [Docker Best Practices](#docker-best-practices)
12. [Multi-stage Builds](#multi-stage-builds)
13. [Docker Registry](#docker-registry)
14. [Troubleshooting](#troubleshooting)

---

## Introduction to Docker

### What is Docker?
- **Containerization platform** that packages applications and their dependencies
- **Lightweight alternative** to virtual machines
- Uses **OS-level virtualization**
- Ensures **"works on my machine"** becomes **"works everywhere"**

### Why Docker?
- **Consistency**: Same environment across dev, staging, production
- **Isolation**: Applications don't interfere with each other
- **Portability**: Run anywhere Docker is installed
- **Efficiency**: Less overhead than VMs
- **Scalability**: Easy to scale applications

### Key Concepts
- **Image**: Read-only template for creating containers
- **Container**: Running instance of an image
- **Dockerfile**: Instructions to build an image
- **Registry**: Repository for storing images (Docker Hub, etc.)

---

## Docker Installation

### Check Installation
```bash
docker --version
docker info
```

### Install Docker (if needed)
- **macOS**: Download Docker Desktop from docker.com
- **Linux**: `sudo apt-get install docker.io` (Ubuntu/Debian)
- **Windows**: Docker Desktop for Windows

### Verify Installation
```bash
docker run hello-world
```

---

## Docker Architecture

### Components
- **Docker Daemon (dockerd)**: Background service managing containers
- **Docker Client**: CLI tool to interact with daemon
- **Docker Engine**: Runtime that executes containers
- **Docker Registry**: Stores Docker images

### How Docker Works
1. Client sends commands to Docker daemon
2. Daemon manages images, containers, networks, volumes
3. Containers share the host OS kernel
4. Each container has isolated filesystem, network, processes

---

## Basic Docker Commands

### System Information
```bash
# Docker version
docker version

# System-wide information
docker info

# Show disk usage
docker system df

# Prune unused data
docker system prune
docker system prune -a  # Remove all unused images
```

### Help Commands
```bash
docker --help
docker <command> --help
```

---

## Working with Images

### Pull Images
```bash
# Pull from Docker Hub
docker pull nginx
docker pull node:18
docker pull node:18-alpine  # Smaller Alpine-based image

# Pull specific tag
docker pull nginx:1.21
docker pull node:18-slim
```

### List Images
```bash
# List all images
docker images
docker image ls

# List with filters
docker images | grep nginx
docker images --filter "dangling=true"
```

### Inspect Images
```bash
# Detailed information
docker inspect <image_name>
docker inspect nginx

# Image history
docker history <image_name>
docker history nginx
```

### Remove Images
```bash
# Remove image
docker rmi <image_id>
docker rmi nginx

# Force remove
docker rmi -f <image_id>

# Remove all unused images
docker image prune -a
```

### Search Images
```bash
# Search Docker Hub
docker search nginx
docker search node
```

### Tag Images
```bash
# Tag an image
docker tag <source_image> <new_tag>
docker tag nginx my-nginx:v1.0
docker tag my-nginx:v1.0 myregistry.com/my-nginx:latest
```

---

## Working with Containers

### Run Containers
```bash
# Run container (foreground)
docker run nginx

# Run in detached mode (background)
docker run -d nginx
docker run -d --name my-nginx nginx

# Run with port mapping
docker run -d -p 8080:80 nginx
docker run -d -p 8080:80 --name web-server nginx

# Run with environment variables
docker run -d -e NODE_ENV=production node:18

# Run with volume mount
docker run -d -v /host/path:/container/path nginx

# Run with interactive terminal
docker run -it ubuntu bash
docker run -it --rm ubuntu bash  # Auto-remove after exit

# Run with custom command
docker run ubuntu echo "Hello World"
docker run node:18 node --version
```

### List Containers
```bash
# Running containers
docker ps

# All containers (including stopped)
docker ps -a

# Latest container
docker ps -l

# Container IDs only
docker ps -q
```

### Container Lifecycle
```bash
# Start container
docker start <container_id>
docker start my-nginx

# Stop container
docker stop <container_id>
docker stop my-nginx

# Restart container
docker restart <container_id>
docker restart my-nginx

# Pause container
docker pause <container_id>

# Unpause container
docker unpause <container_id>

# Kill container (force stop)
docker kill <container_id>
```

### Container Interaction
```bash
# Execute command in running container
docker exec <container_id> <command>
docker exec my-nginx ls -la
docker exec my-nginx cat /etc/nginx/nginx.conf

# Interactive shell in running container
docker exec -it <container_id> bash
docker exec -it my-nginx sh

# Attach to container's main process
docker attach <container_id>
```

### Container Information
```bash
# Container logs
docker logs <container_id>
docker logs my-nginx
docker logs -f my-nginx  # Follow logs
docker logs --tail 100 my-nginx  # Last 100 lines
docker logs --since 10m my-nginx  # Last 10 minutes

# Container stats (resource usage)
docker stats
docker stats <container_id>

# Inspect container
docker inspect <container_id>
docker inspect my-nginx

# Container processes
docker top <container_id>
```

### Remove Containers
```bash
# Remove stopped container
docker rm <container_id>
docker rm my-nginx

# Force remove running container
docker rm -f <container_id>

# Remove all stopped containers
docker container prune

# Remove container after it stops
docker run --rm nginx
```

---

## Dockerfile Deep Dive

### Dockerfile Basics
A Dockerfile is a text file containing instructions to build a Docker image.

### Common Instructions

#### FROM
```dockerfile
# Base image
FROM node:18
FROM node:18-alpine
FROM ubuntu:20.04
FROM nginx:latest
```

#### WORKDIR
```dockerfile
# Set working directory
WORKDIR /app
WORKDIR /usr/src/app
```

#### COPY / ADD
```dockerfile
# Copy files from host to image
COPY package.json .
COPY . .
COPY src/ /app/src/

# ADD can also fetch from URLs and extract archives
ADD https://example.com/file.tar.gz /tmp/
ADD file.tar.gz /tmp/  # Auto-extracts
```

#### RUN
```dockerfile
# Execute commands during build
RUN apt-get update && apt-get install -y curl
RUN npm install
RUN mkdir -p /app/logs
```

#### ENV
```dockerfile
# Set environment variables
ENV NODE_ENV=production
ENV PORT=3000
ENV APP_NAME=myapp
```

#### ARG
```dockerfile
# Build-time variables
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}
```

#### EXPOSE
```dockerfile
# Document which ports the container listens on
EXPOSE 3000
EXPOSE 80 443
```

#### CMD
```dockerfile
# Default command when container starts
CMD ["node", "server.js"]
CMD ["npm", "start"]
CMD node server.js
```

#### ENTRYPOINT
```dockerfile
# Entry point (harder to override than CMD)
ENTRYPOINT ["node"]
CMD ["server.js"]
```

#### USER
```dockerfile
# Run as specific user
USER node
USER 1000
```

#### LABEL
```dockerfile
# Add metadata
LABEL maintainer="your@email.com"
LABEL version="1.0"
```

### Complete Dockerfile Example (Node.js App)
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
ENV NODE_ENV=production
ENV PORT=3000
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

### Build Images
```bash
# Build from Dockerfile
docker build .
docker build -t my-app:latest .
docker build -t my-app:v1.0 .
docker build -t my-app:v1.0 -t my-app:latest .

# Build with build args
docker build --build-arg NODE_ENV=production -t my-app .

# Build from specific Dockerfile
docker build -f Dockerfile.prod -t my-app:prod .

# Build without cache
docker build --no-cache -t my-app .

# Build with progress output
docker build --progress=plain -t my-app .
```

### .dockerignore
Create a `.dockerignore` file to exclude files from build context:
```
node_modules
npm-debug.log
.git
.gitignore
.env
.nyc_output
coverage
.DS_Store
*.md
```

---

## Docker Compose

### What is Docker Compose?
- Tool for defining and running **multi-container** Docker applications
- Uses YAML file to configure services
- Simplifies orchestration of multiple containers

### docker-compose.yml Structure
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - .:/app
    depends_on:
      - db
      - redis

  db:
    image: postgres:14
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Docker Compose Commands
```bash
# Start services
docker-compose up
docker-compose up -d  # Detached mode
docker-compose up --build  # Rebuild images

# Stop services
docker-compose down
docker-compose down -v  # Remove volumes too

# View logs
docker-compose logs
docker-compose logs -f  # Follow logs
docker-compose logs web  # Specific service

# Execute command in service
docker-compose exec web bash
docker-compose exec db psql -U user -d myapp

# Scale services
docker-compose up --scale web=3

# List services
docker-compose ps

# Build services
docker-compose build
docker-compose build --no-cache

# Start/stop/restart
docker-compose start
docker-compose stop
docker-compose restart

# Remove everything
docker-compose down -v --rmi all
```

### Compose File Versions
- Version 3.8: Latest stable (recommended)
- Version 3.7, 3.6: Older but still used
- Version 2: Legacy format

---

## Docker Networking

### Network Types
- **bridge**: Default network for containers
- **host**: Use host's network directly
- **none**: No networking
- **overlay**: For Swarm mode
- **macvlan**: Assign MAC address to container

### Network Commands
```bash
# List networks
docker network ls

# Inspect network
docker network inspect bridge

# Create network
docker network create my-network
docker network create --driver bridge my-network

# Connect container to network
docker network connect my-network my-container

# Disconnect container
docker network disconnect my-network my-container

# Remove network
docker network rm my-network
docker network prune  # Remove unused networks
```

### Container Communication
```bash
# Containers on same network can communicate by name
docker run -d --name web --network my-network nginx
docker run -d --name app --network my-network node:18

# Use container name as hostname
# app can reach web at http://web:80
```

### Port Mapping
```bash
# Map host port to container port
docker run -p 8080:80 nginx
docker run -p 127.0.0.1:8080:80 nginx  # Bind to specific interface
docker run -p 8080:80/tcp nginx  # Specify protocol
docker run -P nginx  # Random port mapping
```

---

## Docker Volumes

### Volume Types
- **Named volumes**: Managed by Docker
- **Bind mounts**: Mount host directory
- **tmpfs mounts**: In-memory storage

### Volume Commands
```bash
# List volumes
docker volume ls

# Create volume
docker volume create my-volume

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume
docker volume prune  # Remove unused volumes
```

### Using Volumes
```bash
# Named volume
docker run -v my-volume:/data nginx

# Bind mount
docker run -v /host/path:/container/path nginx
docker run --mount type=bind,source=/host/path,target=/container/path nginx

# tmpfs mount
docker run --tmpfs /app nginx
```

### Volume in Dockerfile
```dockerfile
VOLUME ["/data"]
VOLUME /var/log
```

### Volume in docker-compose.yml
```yaml
services:
  web:
    volumes:
      - ./app:/app
      - logs:/app/logs
      - /host/path:/container/path

volumes:
  logs:
```

---

## Docker Best Practices

### Image Optimization
1. **Use .dockerignore**: Exclude unnecessary files
2. **Multi-stage builds**: Reduce final image size
3. **Layer caching**: Order Dockerfile instructions wisely
4. **Use specific tags**: Avoid `latest` in production
5. **Minimal base images**: Use Alpine when possible
6. **Combine RUN commands**: Reduce layers

### Security
1. **Don't run as root**: Use USER instruction
2. **Scan images**: `docker scan <image>`
3. **Keep images updated**: Regular security updates
4. **Use secrets**: Don't hardcode credentials
5. **Minimal attack surface**: Only install what's needed

### Performance
1. **Layer ordering**: Put frequently changing layers last
2. **Cache dependencies**: Copy package files before source code
3. **Use build cache**: Leverage Docker's caching mechanism
4. **Optimize COPY**: Be specific about what to copy

### Example Optimized Dockerfile
```dockerfile
# Bad
FROM node:18
COPY . .
RUN npm install
CMD ["node", "server.js"]

# Good
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

FROM node:18-alpine
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
COPY --from=builder /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .
USER nodejs
EXPOSE 3000
CMD ["node", "server.js"]
```

---

## Multi-stage Builds

### Why Multi-stage Builds?
- **Smaller final images**: Only include runtime dependencies
- **Build tools separation**: Keep build tools out of production
- **Security**: Fewer components in final image

### Example Multi-stage Build
```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

## Docker Registry

### Docker Hub
```bash
# Login to Docker Hub
docker login
docker login -u username

# Push image
docker tag my-app:latest username/my-app:latest
docker push username/my-app:latest

# Pull from Docker Hub
docker pull username/my-app:latest
```

### Private Registry
```bash
# Login to private registry
docker login registry.example.com

# Tag for private registry
docker tag my-app:latest registry.example.com/my-app:latest

# Push to private registry
docker push registry.example.com/my-app:latest

# Pull from private registry
docker pull registry.example.com/my-app:latest
```

### Run Local Registry
```bash
# Start local registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag and push to local registry
docker tag my-app:latest localhost:5000/my-app:latest
docker push localhost:5000/my-app:latest
```

---

## Troubleshooting

### Common Issues

#### Container won't start
```bash
# Check logs
docker logs <container_id>

# Check container status
docker ps -a

# Inspect container
docker inspect <container_id>
```

#### Port already in use
```bash
# Find process using port
lsof -i :8080  # macOS/Linux
netstat -ano | findstr :8080  # Windows

# Use different port
docker run -p 8081:80 nginx
```

#### Out of disk space
```bash
# Check disk usage
docker system df

# Clean up
docker system prune -a
docker volume prune
docker image prune -a
```

#### Permission denied
```bash
# Run with specific user
docker run -u $(id -u):$(id -g) nginx

# Fix volume permissions
docker run -v /host/path:/container/path:Z nginx
```

#### Container keeps restarting
```bash
# Check exit code
docker ps -a

# Check logs
docker logs <container_id>

# Run interactively to debug
docker run -it <image> bash
```

### Debugging Commands
```bash
# Execute shell in container
docker exec -it <container_id> sh

# Check container processes
docker top <container_id>

# Check container stats
docker stats <container_id>

# Inspect network
docker network inspect bridge

# Check events
docker events
```

---

## Practical Workflow

### Development Workflow
1. **Create Dockerfile** for your application
2. **Build image**: `docker build -t my-app .`
3. **Test locally**: `docker run -p 3000:3000 my-app`
4. **Use docker-compose** for multi-container apps
5. **Push to registry** when ready

### Production Workflow
1. **Multi-stage build** for optimized images
2. **Tag with version**: `docker tag my-app:latest my-app:v1.0.0`
3. **Push to registry**: `docker push my-app:v1.0.0`
4. **Pull on server**: `docker pull my-app:v1.0.0`
5. **Run with proper config**: Environment variables, volumes, networks

### Example: Node.js App Workflow
```bash
# 1. Create Dockerfile
# 2. Build image
docker build -t my-node-app:latest .

# 3. Run container
docker run -d \
  --name my-app \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -v app-data:/app/data \
  my-node-app:latest

# 4. Check logs
docker logs -f my-app

# 5. Update application
docker build -t my-node-app:v2.0 .
docker stop my-app
docker rm my-app
docker run -d --name my-app -p 3000:3000 my-node-app:v2.0
```

---

## Summary

### Key Takeaways
- **Images** are templates, **containers** are running instances
- **Dockerfile** defines how to build images
- **Docker Compose** orchestrates multi-container applications
- **Networks** enable container communication
- **Volumes** persist data
- **Multi-stage builds** optimize image size
- **Best practices** ensure security and performance

### Essential Commands Cheat Sheet
```bash
# Images
docker build -t name:tag .
docker images
docker rmi image

# Containers
docker run -d -p host:container image
docker ps
docker logs container
docker exec -it container bash
docker stop/start/rm container

# Compose
docker-compose up -d
docker-compose down
docker-compose logs

# System
docker system df
docker system prune
```

---

## Next Steps
- Practice building images for different applications
- Experiment with docker-compose for multi-service apps
- Learn about Docker Swarm for orchestration
- Explore Kubernetes for advanced container orchestration

