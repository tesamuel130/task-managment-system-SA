# 📋 Task Management System Architecture Guide

This document outlines the architecture, structure, conventions, and workflow of the **Task Management System** built using **NestJS**, **Docker**, **Kubernetes**, **NGINX**, **GitHub Actions**, **Redis**, **Kafka**, and **GraphQL** for a microservice-based application with real-time collaboration features.

---

## 📁 Project Structure

```
task-management-system/
├── services/                     # Microservices
│   ├── user-service/            # User management (auth, profiles)
│   ├── task-service/            # Task CRUD operations
│   ├── notification-service/    # Real-time notifications
│   ├── api-gateway/             # GraphQL API gateway
├── frontend/                    # React/Next.js frontend
├── infra/                       # Dockerfiles, Kubernetes manifests
│   ├── docker/                  # Dockerfiles for services
│   ├── k8s/                     # Kubernetes manifests
├── .github/workflows/           # GitHub Actions CI/CD configs
├── docker-compose.yml           # Local development setup
└── .env                        # Environment variables
```

---

## 🧱 Architecture Overview

The system uses a **microservice architecture** with event-driven and real-time capabilities:

- **NestJS**: Core framework for microservices and API Gateway.
- **GraphQL**: Unified API for frontend communication.
- **Redis**: Caching and pub/sub for real-time updates.
- **Kafka**: Event streaming for asynchronous communication.
- **TCP**: Direct service-to-service communication.
- **Docker**: Containerization for services and frontend.
- **Kubernetes**: Orchestration for scalability and resilience.
- **NGINX**: Reverse proxy and load balancing.
- **GitHub Actions**: CI/CD for automated testing and deployment.

---

## 🧠 State Management

- **Redis**:
  - Caches user sessions and task metadata for performance.
  - Pub/Sub for real-time notifications (e.g., task updates).
- **Kafka**:
  - Publishes and consumes events (e.g., task creation, updates).
  - Used for logging and triggering notifications.
- **Database**:
  - PostgreSQL/MongoDB for persistent storage of users and tasks.

---

## 🔌 Database Schema (PostgreSQL Example)

**Tables**:

- `users`  
  → `id`, `name`, `email`, `password_hash`, `role`

- `tasks`  
  → `id`, `title`, `description`, `status`, `assigned_user_id`, `created_at`

- `notifications`  
  → `id`, `user_id`, `task_id`, `message`, `created_at`

**Redis Cache**:
- Key: `user:{id}:session` → Stores JWT tokens.
- Key: `task:{id}` → Caches task metadata.

**Kafka Topics**:
- `task-events`: Task creation, update, deletion events.
- `notification-events`: Triggers for notifications.

---

## 🔐 Authentication

- **JWT**: Handled by User Service for secure authentication.
- **Redis**: Caches JWT tokens for session management.
- **API Gateway**: Validates tokens and routes requests to services.

---

## 🌍 Real-Time Features

- **Redis Pub/Sub**: Publishes task updates for real-time frontend notifications.
- **GraphQL Subscriptions**: Frontend subscribes to task and notification updates.
- **TCP Communication**:
  - Task Service sends task updates to Notification Service via TCP.
  - Implemented using Node.js `net` module in NestJS.
- **Kafka**: Publishes task events for asynchronous processing (e.g., logging, notifications).

---

## ⚙️ GitHub Actions CI/CD

**`.github/workflows/ci.yml`**

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Build and push services
        run: |
          docker build -t user-service:latest ./services/user-service
          docker build -t task-service:latest ./services/task-service
          docker build -t notification-service:latest ./services/notification-service
          docker build -t api-gateway:latest ./services/api-gateway
          docker push user-service:latest
          docker push task-service:latest
          docker push notification-service:latest
          docker push api-gateway:latest
```

---

## 🐳 Docker and Kubernetes

- **Docker**:
  - Each service and frontend is containerized.
  - Example Dockerfile for Task Service:
    ```dockerfile
    FROM node:18-alpine
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    RUN npm run build
    CMD ["npm", "run", "start:prod"]
    ```

- **Kubernetes**:
  - Deployments for each service with replicas for scalability.
  - NGINX as ingress controller for routing.
  - Example Deployment for Task Service:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: task-service
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: task-service
      template:
        metadata:
          labels:
            app: task-service
        spec:
          containers:
          - name: task-service
            image: task-service:latest
            ports:
            - containerPort: 3000
    ```

---

## 🚀 Workflow

1. **Development**:
   - Use `docker-compose.yml` for local development.
   - Run services with `docker-compose up`.

2. **Testing**:
   - Write unit and integration tests with Jest.
   - Run tests in CI pipeline.

3. **Deployment**:
   - Push Docker images to a registry (e.g., Docker Hub).
   - Deploy to Kubernetes (local with Minikube or cloud with EKS/GKE).

4. **Monitoring**:
   - Use Kubernetes logs and metrics for monitoring.
   - Optionally integrate Prometheus/Grafana for observability.

---

## 📡 Service Communication

- **TCP**: Task Service communicates with Notification Service for real-time updates.
- **Kafka**: Publishes events for asynchronous processing.
- **GraphQL**: Frontend interacts with API Gateway for queries, mutations, and subscriptions.

---

## 🔧 Setup Instructions

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd task-management-system
   ```

2. Set up environment variables in `.env`:
   ```env
   DATABASE_URL=postgresql://user:password@localhost:5432/taskdb
   REDIS_URL=redis://localhost:6379
   KAFKA_BROKERS=localhost:9092
   JWT_SECRET=your-secret
   ```

3. Start services locally:
   ```bash
   docker-compose up
   ```

4. Access the frontend at `http://localhost:3000` and GraphQL API at `http://localhost:4000/graphql`.

---