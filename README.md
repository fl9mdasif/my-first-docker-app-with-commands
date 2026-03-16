# Docker with TypeScript Backend - Complete Guide

A comprehensive guide for building, deploying, and managing Docker containers with a TypeScript-based backend application. This documentation covers essential Docker commands, best practices, and common workflows for containerized application development.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Image Management](#image-management)
4. [Container Management](#container-management)
5. [Container Inspection](#container-inspection)
6. [Networking](#networking)
7. [Volumes](#volumes)
8. [Cleanup & Maintenance](#cleanup--maintenance)
9. [Build Optimization](#build-optimization)
10. [Debugging & Troubleshooting](#debugging--troubleshooting)
11. [Docker Compose](#docker-compose)
12. [Best Practices](#best-practices)
13. [Common Issues & Solutions](#common-issues--solutions)

---

## Introduction

Docker is a containerization platform that allows you to package your application with all its dependencies into a standardized unit for software development. This guide provides a complete reference for working with Docker in a TypeScript backend environment.

**Why Docker?**

- **Consistency**: Same environment across development, testing, and production
- **Isolation**: Dependencies don't conflict with your system
- **Scalability**: Easy to scale applications horizontally
- **Portability**: Run anywhere Docker is installed

---

## Prerequisites

Before getting started, ensure you have:

- Docker Desktop installed ([Download here](https://www.docker.com/products/docker-desktop))
- Basic understanding of command-line interface
- Node.js and npm knowledge
- TypeScript basics
- Docker version 20.10+

**Verify Installation:**

```bash
docker --version
docker run hello-world
```

---

## Image Management

Docker images are blueprints for containers. They contain all the application code, libraries, and dependencies needed to run your application.

### Building Images

#### Basic Image Build

```bash
docker build -t <image-name>:<tag> .
```

Builds a Docker image from a Dockerfile in the current directory.

**Example:**

```bash
docker build -t myapp:1.0 .
docker build -t myapp:latest .
```

#### Advanced Build Options

**Build with Specific Dockerfile:**

```bash
docker build -f Dockerfile.prod -t <image-name>:<tag> .
```

**Build Without Cache:**

```bash
docker build --no-cache -t <image-name>:<tag> .
```

Use this when you want to rebuild all layers, ignoring cached layers.

**Build with Arguments:**

```bash
docker build --build-arg NODE_ENV=production -t <image-name>:<tag> .
```

### Listing Images

```bash
# List all images
docker image ls
docker images
docker images -a

# View detailed information
docker image inspect <image-id>
```

### Image Tagging

```bash
# Tag an existing image
docker tag <image-id> <new-image-name>:<tag>

# Example
docker tag abc123def myapp:v1.0.0 myapp:latest
```

### Pushing & Pulling Images

**Push to Registry (Docker Hub):**

```bash
# Tag your image with registry address / clone/ rename a image
docker tag myapp:latest username/myapp:latest

# Push to Docker Hub
docker push username/myapp:latest

# Push to private registry
docker push registry.example.com/myapp:latest
```

**Pull from Registry:**

```bash
docker pull <image-name>:<tag>

# Examples
docker pull myapp:latest
docker pull node:18-alpine
docker pull postgres:15
```

### Removing Images

```bash
# Remove by ID or name
docker image rmi <image-id>
docker rmi <image-name>:<tag>

# Remove all unused images
docker image prune

# Force remove (even if used by containers)
docker rmi -f <image-name>:<tag>
```

---

## Container Management

Containers are running instances of Docker images. Think of them as lightweight, portable execution environments.

### Running Containers

#### Basic Container Run

```bash
docker run <image-name>
```

#### Run in Background (Detached Mode)

```bash
docker run -d <image-name>
```

#### Port Mapping (Host:Container)

```bash
docker run -p 5001:5000 <image-name>

# Example - Map port 3000 from container to 8080 on host
docker run -p 8080:3000 myapp:latest
```

#### Run with Container Name

```bash
docker run --name <container-name> <image-name>

# Example
docker run --name my-app-container myapp:latest
```

#### Run and Remove After Exit

```bash
docker run --rm <image-name>
```

Useful for temporary containers or testing.

#### Run with Environment Variables

```bash
docker run -e VAR_NAME=value <image-name>

# Example
docker run -e NODE_ENV=production -e PORT=3000 myapp:latest

# Multiple environment variables
docker run -e NODE_ENV=production -e DB_HOST=localhost -e DB_PORT=5432 myapp:latest
```

#### Run with Volume Mount

```bash
docker run -v /host/path:/container/path <image-name>

# Example - Mount current directory
docker run -v $(pwd):/app myapp:latest

# Named volume
docker run -v my-volume:/app/data myapp:latest

# Read-only mount
docker run -v /host/path:/container/path:ro myapp:latest
```

#### Interactive Terminal

```bash
docker run -it <image-name> /bin/bash

# For Alpine Linux
docker run -it <image-name> sh
```

#### Complete Example

```bash
docker run -d \
  --name myapp-prod \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e PORT=3000 \
  -v /app/logs:/logs \
  --restart unless-stopped \
  myapp:latest
```

### Listing Containers

```bash
# List running containers
docker container ls
docker ps

# List all containers (running and stopped)
docker container ls -a
docker ps -a

# List with custom format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

### Starting & Stopping Containers

```bash
# Stop a running container (graceful shutdown)
docker stop <container-id> <container-name>

# Start a stopped container
docker start <container-id> <container-name>

# Restart a container
docker restart <container-id> <container-name>

# Kill a container (force stop)
docker kill <container-id> <container-name>
```

### Removing Containers

```bash
# Remove stopped container
docker container rm <container-id> <container-name>
docker rm <container-name>

# Remove all stopped containers
docker container prune

# Remove container with volumes
docker rm -v <container-name>
```

### Container Logging

```bash
# View logs
docker logs <container-id>

# Follow logs in real-time
docker logs -f <container-id>

# View last 100 lines
docker logs --tail 100 <container-id>

# View with timestamps
docker logs -t <container-id>

# Combine options
docker logs -f --tail 50 -t <container-name>
```

### Executing Commands in Containers

```bash
# Execute command in running container
docker exec -it <container-id> /bin/bash

# Run shell
docker exec -it <container-id> sh

# Run specific command
docker exec <container-id> npm test

# Example - Access database shell
docker exec -it <container-id> psql -U postgres
```

### Container Management

```bash
# Rename a container
docker rename <old-name> <new-name>

# Pause a container (freeze processes)
docker pause <container-id>

# Unpause a container
docker unpause <container-id>
```

---

## Container Inspection

Understanding what's happening inside your containers is crucial for debugging and optimization.

### Detailed Container Information

```bash
docker inspect <container-id>

# Pretty print JSON output
docker inspect --format='{{json .}}' <container-id> | jq
```

### Running Processes

```bash
# View processes in container
docker top <container-id>

# Example output shows PID, command running inside
```

### Resource Usage Statistics

```bash
# View CPU, memory, network I/O stats
docker stats <container-id>

# View for all containers
docker stats

# View specific format
docker stats --no-stream
```

### Docker Events

```bash
# View events in real-time
docker events

# Filter by container
docker events --filter 'type=container'

# Filter by image
docker events --filter 'type=image'
```

---

## Networking

Docker networking allows containers to communicate with each other and external services.

### Network Commands

```bash
# List all networks
docker network ls

# Create custom network
docker network create <network-name>

# Create a bridge network
docker network create --driver bridge <network-name>

# Inspect network details
docker network inspect <network-name>

# Remove a network
docker network rm <network-name>
```

### Connecting Containers

```bash
# Connect running container to network
docker network connect <network-name> <container-name>

# Disconnect container from network
docker network disconnect <network-name> <container-name>
```

### Network Types

1. **Bridge Network**: Default, good for single host
2. **Host Network**: Share host's network stack (Linux only)
3. **Overlay Network**: For Docker Swarm, multi-host
4. **Macvlan Network**: Assign MAC address to container
5. **None Network**: No networking

**Example - Create and Use Network:**

```bash
# Create network
docker network create my-app-network

# Run multiple containers on same network
docker run -d --name app-server --network my-app-network myapp:latest
docker run -d --name db-server --network my-app-network postgres:15

# Containers can now communicate by name
# From app: curl http://db-server:5432
```

---

## Volumes

Volumes provide persistent data storage and data sharing between containers.

### Volume Management

```bash
# List all volumes
docker volume ls

# Create a named volume
docker volume create <volume-name>

# Inspect volume details
docker volume inspect <volume-name>

# Remove a volume
docker volume rm <volume-name>

# Remove all unused volumes
docker volume prune
```

### Using Volumes

```bash
# Run container with named volume
docker run -v <volume-name>:/container/path <image-name>

# Example
docker run -d --name postgres-db -v postgres-data:/var/lib/postgresql/data postgres:15

# Bind mount (host path)
docker run -v /host/path:/container/path <image-name>

# Current directory
docker run -v $(pwd):/app <image-name>
```

### Volume Types

1. **Named Volumes**: Managed by Docker, recommended for production
2. **Bind Mounts**: Direct host directory mapping, good for development
3. **tmpfs Mounts**: In-memory storage

**Best Practices:**

- Use named volumes for production database data
- Use bind mounts for development source code
- Always backup important volume data

---

## Cleanup & Maintenance

Regular cleanup prevents your system from becoming bloated with unused resources.

### Removing Unused Resources

```bash
# Remove stopped containers
docker container prune

# Remove dangling images
docker image prune

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune
```

### Complete System Cleanup

```bash
# Remove ALL unused data
docker system prune

# With volumes (⚠️ Warning: Removes unused volumes)
docker system prune -a --volumes

# Check disk usage
docker system df
```

| Command                  | Effect                        |
| ------------------------ | ----------------------------- |
| `docker container prune` | Remove all stopped containers |
| `docker image prune`     | Remove dangling images        |
| `docker volume prune`    | Remove unused volumes         |
| `docker system prune`    | Remove all unused data        |

---

## Build Optimization

Optimizing Docker builds improves development speed and production performance.

### Multi-Stage Builds

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

**Benefits:**

- Smaller final image size
- Faster deployments
- Reduced attack surface

### Build Arguments

```bash
# In Dockerfile
ARG NODE_ENV=development

# Build with argument
docker build --build-arg NODE_ENV=production -t myapp:prod .
```

### Caching Strategy

```bash
# Build without cache (slower, but fresh)
docker build --no-cache -t <image-name>:<tag> .

# Use cache (default, faster)
docker build -t <image-name>:<tag> .
```

**Optimize layer caching:**

```dockerfile
# Bad - cache busted on any code change
COPY . /app
RUN npm install

# Good - cache dependencies separately
COPY package*.json /app/
RUN npm ci
COPY . /app
```

---

## Debugging & Troubleshooting

Professional debugging techniques for containerized applications.

### System Information

```bash
# Docker version
docker --version

# System information
docker info

# Disk usage
docker system df

# Detailed disk usage
docker system df -v
```

### Container Debugging

```bash
# View logs
docker logs <container-id>

# Follow logs
docker logs -f <container-id>

# Access shell
docker exec -it <container-id> /bin/bash

# Run specific command
docker exec <container-id> npm test

# Check file system
docker exec <container-id> ls -la /app
```

### Common Issues

| Issue                       | Solution                                                |
| --------------------------- | ------------------------------------------------------- |
| Port already in use         | `docker run -p 8080:3000 ...` (use different host port) |
| Out of disk space           | `docker system prune -a --volumes`                      |
| Container exits immediately | `docker logs <container>` (check error)                 |
| Network connectivity        | `docker exec <container> ping <other-container>`        |
| High memory usage           | `docker stats` (monitor resource usage)                 |

### Network Debugging

```bash
# Check if containers can communicate
docker exec <container-1> ping <container-2>

# Check open ports
docker exec <container> netstat -tlnp

# Check network interfaces
docker exec <container> ip addr
```

---

## Docker Compose

Docker Compose simplifies managing multi-container applications.

### Essential Commands

```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# View service logs
docker-compose logs <service-name>

# Execute command in service
docker-compose exec <service-name> <command>

# Rebuild images
docker-compose build

# Rebuild and start
docker-compose up -d --build

# Remove containers and images
docker-compose down -v

# View running containers
docker-compose ps
```

### Example docker-compose.yml

```yaml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: my-app
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DB_HOST: db
      DB_NAME: myapp
    depends_on:
      - db
    volumes:
      - ./logs:/app/logs
    networks:
      - app-network
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    container_name: my-app-db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    restart: unless-stopped

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

---

## Best Practices

### 1. Image Best Practices

```dockerfile
# Use specific base image versions
FROM node:18-alpine  # Good
FROM node:latest     # Bad

# Keep images small
RUN apt-get update && apt-get install -y package && rm -rf /var/lib/apt/lists/*

# Use .dockerignore
echo "node_modules\n.git\n.env" > .dockerignore

# Run as non-root
RUN useradd -m appuser
USER appuser
```

### 2. Security Best Practices

```bash
# Scan image for vulnerabilities
docker scan myapp:latest

# Run with read-only filesystem
docker run --read-only myapp:latest

# Use resource limits
docker run -m 512m --cpus="0.5" myapp:latest

# Don't store secrets in images
# Use environment variables or secret management
```

### 3. Development Workflow

```bash
# Use bind mounts for hot reload
docker run -v $(pwd):/app myapp:latest

# Use .dockerignore to exclude files
cat > .dockerignore << EOF
node_modules
.git
.env
coverage
EOF

# Use docker-compose for local development
docker-compose up -d
```

### 4. Naming Conventions

```bash
# Use meaningful names
docker build -t company/app-name:major.minor.patch .
docker run --name app-instance-01 company/app-name:1.0.0

# Tag with git commit
docker build -t myapp:$(git rev-parse --short HEAD) .
```

### 5. Logging Best Practices

```bash
# Write logs to stdout/stderr in container
# Instead of writing to files inside container
# Use 'docker logs' to access them

# Configure log drivers
docker run --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 myapp:latest
```

### 6. Documentation

```dockerfile
# Always document your Dockerfile
# Use labels for metadata
LABEL maintainer="your-email@example.com"
LABEL version="1.0.0"
LABEL description="TypeScript backend service"

# Document exposed ports
EXPOSE 3000

# Document entrypoint
ENTRYPOINT ["node", "dist/server.js"]
```

---

## Common Issues & Solutions

### Issue 1: Container Exits Immediately

**Problem:** Container starts and stops without running anything.

**Solution:**

```bash
# Check logs
docker logs <container-id>

# Run interactively to see what's wrong
docker run -it myapp:latest sh

# Check if ENTRYPOINT/CMD exists
docker inspect myapp:latest | grep -A 2 Cmd
```

### Issue 2: Network Connectivity Problems

**Problem:** Containers can't communicate with each other.

**Solution:**

```bash
# Ensure containers are on same network
docker run --network my-network <image>

# Check if service is listening
docker exec <container> netstat -tlnp

# Test connectivity
docker exec <container-1> ping <container-2>
```

### Issue 3: Port Already in Use

**Problem:** Can't bind to port because it's already in use.

**Solution:**

```bash
# Use different port
docker run -p 8080:3000 myapp:latest

# Find what's using the port
lsof -i :3000  # Mac/Linux
netstat -ano | findstr :3000  # Windows

# Kill the process or container
docker stop <container-id>
```

### Issue 4: Out of Disk Space

**Problem:** No space left on device.

**Solution:**

```bash
# Check disk usage
docker system df

# Clean up everything
docker system prune -a --volumes

# Remove specific images
docker rmi <image-name>

# Increase Docker disk limit in Docker Desktop settings
```

### Issue 5: High Memory Usage

**Problem:** Container consuming too much memory.

**Solution:**

```bash
# Monitor resource usage
docker stats <container-id>

# Set memory limit
docker run -m 512m myapp:latest

# Check application logs for memory leaks
docker logs <container-id>

# Use memory profiling tools
docker exec <container-id> npm run profile
```

---

## Quick Reference Table

| Task              | Command                                |
| ----------------- | -------------------------------------- |
| Build image       | `docker build -t myapp:1.0 .`          |
| Run container     | `docker run -d -p 3000:3000 myapp:1.0` |
| Show logs         | `docker logs -f <container>`           |
| Execute command   | `docker exec -it <container> sh`       |
| Stop container    | `docker stop <container>`              |
| Remove container  | `docker rm <container>`                |
| List images       | `docker images`                        |
| Push to registry  | `docker push username/myapp:1.0`       |
| Create network    | `docker network create my-net`         |
| Create volume     | `docker volume create my-vol`          |
| Start compose     | `docker-compose up -d`                 |
| View compose logs | `docker-compose logs -f`               |

---

## Resources

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Node.js Docker Best Practices](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)

---

## Contributing

For issues or improvements to this guide, please refer to the project repository.

**Last Updated:** March 2026  
**Version:** 1.0.0  
**Author:** Senior Docker Engineer
