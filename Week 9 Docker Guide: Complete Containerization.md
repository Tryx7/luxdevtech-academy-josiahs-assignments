
# Docker Guide: Complete Containerization

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Docker Installation](#docker-installation)
4. [Basic Docker Concepts](#basic-docker-concepts)
5. [Containerizing Your Application](#containerizing-your-application)
6. [Docker Compose for Multi-Service Applications](#docker-compose-for-multi-service-applications)
7. [Assignment Project Structure](#assignment-project-structure)
8. [Step-by-Step Implementation](#step-by-step-implementation)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)
11. [Submission Guidelines](#submission-guidelines)

## Introduction

This guide provides comprehensive instructions for containerizing applications using Docker for your class assignment. Docker allows you to package applications with all their dependencies, ensuring consistent behavior across different environments.

### Why Docker for Assignments?

- **Consistency**: Same environment on your laptop and when grading
- **Isolation**: No conflicts between different project dependencies
- **Reproducibility**: Easy to share and run your work
- **Modern Skills**: Industry-standard containerization technology

## Prerequisites

### Software Requirements

- **Operating System**: Windows 10/11, macOS 10.15+, or Ubuntu 18.04+
- **Docker Desktop** (Windows/Mac) or **Docker Engine** (Linux)
- **Git** for version control
- **Code Editor**: VS Code (recommended) with Docker extension

### Knowledge Requirements

- Basic command line usage
- Fundamental understanding of your programming language
- Basic networking concepts

## Docker Installation

### Windows/macOS

1. Download Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop)
2. Install and follow setup wizard
3. Enable WSL 2 backend on Windows (recommended)
4. Start Docker Desktop

### Linux (Ubuntu)

```bash
# Update package index
sudo apt update

# Install Docker
sudo apt install docker.io

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to docker group (avoid sudo for docker commands)
sudo usermod -aG docker $USER
# Log out and log back in for changes to take effect
```

### Verification

```bash
docker --version
docker-compose --version
docker run hello-world
```

## Basic Docker Concepts

### Key Terminology

- **Image**: Blueprint/template for containers (like a class in OOP)
- **Container**: Running instance of an image (like an object instance)
- **Dockerfile**: Recipe for building images
- **Volume**: Persistent data storage
- **Network**: Communication between containers

### Essential Commands

```bash
# Image management
docker images
docker pull <image-name>
docker build -t <tag> .

# Container management
docker ps
docker run -d -p 80:80 --name myapp <image>
docker stop <container>
docker rm <container>

# System management
docker system prune
docker logs <container>
```

## Containerizing Your Application

### Sample Dockerfile Structure

```dockerfile
# Base image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements first (better caching)
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Set environment variables
ENV PYTHONUNBUFFERED=1

# Define entry point
CMD ["python", "app.py"]
```

### Language-Specific Dockerfiles

#### Python Application

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "main.py"]
```

#### Node.js Application

```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

#### Java Application

```dockerfile
FROM openjdk:11-jre-slim

WORKDIR /app

COPY target/myapp.jar .

EXPOSE 8080
CMD ["java", "-jar", "myapp.jar"]
```

### Building and Running

```bash
# Build image
docker build -t my-assignment-app .

# Run container
docker run -d -p 8000:8000 --name assignment-container my-assignment-app

# Test application
curl http://localhost:8000
```

## Docker Compose for Multi-Service Applications

### Sample docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
    depends_on:
      - db
    volumes:
      - ./app:/app

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Common Docker Compose Commands

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs

# Rebuild and restart
docker-compose up -d --build

# Scale services
docker-compose up -d --scale web=3
```

## Assignment Project Structure

### Recommended Directory Structure

```
your-assignment/
├── docker-compose.yml
├── Dockerfile
├── .dockerignore
├── README.md
├── requirements.txt (or package.json, etc.)
├── src/
│   ├── main.py (or app.js, etc.)
│   └── ...
├── config/
│   └── ...
├── data/ (if needed)
└── tests/
    └── ...
```

### Sample .dockerignore

```
.git
.gitignore
README.md
.env
*.pyc
__pycache__
node_modules
.DS_Store
Dockerfile*
docker-compose*
.vscode
```

## Step-by-Step Implementation

### Phase 1: Basic Containerization

1. **Create Dockerfile**
2. **Build and test image locally**
3. **Verify application works in container**

### Phase 2: Multi-Service Setup

1. **Create docker-compose.yml**
2. **Define service dependencies**
3. **Configure networking and volumes**

### Phase 3: Optimization

1. **Implement proper logging**
2. **Add health checks**
3. **Optimize image size**

### Complete Working Example

#### Dockerfile

```dockerfile
FROM python:3.9-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements and install
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user (security best practice)
RUN useradd -m -r appuser && chown -R appuser /app
USER appuser

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Start application
CMD ["python", "src/main.py"]
```

#### docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - ENVIRONMENT=development
      - DEBUG=True
    volumes:
      - ./src:/app/src:ro
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  database:
    image: postgres:13
    environment:
      POSTGRES_DB: assignment_db
      POSTGRES_USER: student
      POSTGRES_PASSWORD: student123
    volumes:
      - db_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  db_data:
```

#### Build and Run Script (build.sh)

```bash
#!/bin/bash

echo "Building assignment application..."
docker-compose down
docker-compose build --no-cache
docker-compose up -d

echo "Waiting for services to start..."
sleep 10

echo "Application is running at http://localhost:8000"
echo "Database is available at localhost:5432"
echo "Use 'docker-compose logs' to view logs"
```

## Best Practices

### Security

```dockerfile
# Use official base images
FROM python:3.9-slim

# Don't run as root
RUN useradd -m -r appuser && chown -R appuser /app
USER appuser

# Use .dockerignore to exclude sensitive files
```

### Performance

```dockerfile
# Multi-stage builds for compiled languages
FROM node:16 as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:16-alpine
COPY --from=builder /app/node_modules ./node_modules
COPY . .
```

### Maintainability

```dockerfile
# Use specific version tags
FROM python:3.9-slim

# Label your image
LABEL maintainer="your.name@university.edu"
LABEL version="1.0"
LABEL description="Class assignment application"

# Install security updates
RUN apt-get update && apt-get upgrade -y
```

### Development vs Production

```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  app:
    build: .
    volumes:
      - .:/app  # Live reload for development
    environment:
      - DEBUG=True
```

## Troubleshooting

### Common Issues and Solutions

#### Port Already in Use

```bash
# Find what's using the port
sudo netstat -tulpn | grep :8000

# Use different port
docker run -p 8001:8000 my-app
```

#### Container Won't Start

```bash
# Check logs
docker logs <container-name>

# Run interactively to debug
docker run -it my-app /bin/bash
```

#### Build Failures

```bash
# Build with no cache
docker build --no-cache -t my-app .

# See build steps
docker build --progress=plain -t my-app .
```

#### Permission Issues

```bash
# Fix file permissions
chmod +x build.sh

# Docker daemon not running
sudo systemctl start docker
```

### Debugging Commands

```bash
# Inspect running containers
docker ps
docker inspect <container>

# Check resource usage
docker stats

# View container processes
docker top <container>

# Execute commands in running container
docker exec -it <container> /bin/bash
```

## Submission Guidelines

### Required Files for Submission

1. **Dockerfile** - Your application containerization
2. **docker-compose.yml** - Multi-service configuration
3. **README.md** - Setup and run instructions
4. **.dockerignore** - Files to exclude from build
5. **build.sh** (optional) - Build and run script

### Submission Checklist

- [ ] Dockerfile builds without errors
- [ ] All services start with docker-compose up
- [ ] Application is accessible on specified ports
- [ ] No hardcoded passwords in files
- [ ] README contains clear instructions
- [ ] All code is properly commented

### Sample README for Submission

```markdown
# Assignment Title

## Quick Start

1. Clone repository: `git clone <repo-url>`
2. Run: `docker-compose up -d`
3. Access: http://localhost:8000

## Services

- **Web App**: Port 8000
- **Database**: Port 5432 (PostgreSQL)

## Development

- `docker-compose up -d --build` - Rebuild and start
- `docker-compose logs -f` - View logs
- `docker-compose down` - Stop services

## Features

- Feature 1: Description
- Feature 2: Description
```

### Grading Criteria

- **Functionality** (40%): Application works as specified
- **Docker Implementation** (30%): Proper containerization
- **Documentation** (20%): Clear setup instructions
- **Best Practices** (10%): Security and optimization

## Additional Resources

### Learning Materials

- [Docker Official Documentation](https://docs.docker.com/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

### Useful Tools

- **Docker Extension for VS Code**
- **Docker Desktop** (includes Kubernetes)
- **Lens IDE** for container management

### Common Assignment Patterns

- Web application + database
- Microservices architecture
- Data processing pipeline
- API backend + frontend

---

**Remember**: The goal is not just making it work, but understanding how and why it works. Document your learning process and challenges faced!

*Good luck with your assignment!* 🐳
