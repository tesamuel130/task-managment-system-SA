# API Design Documentation

## Overview
This document outlines the API design, system architecture, CI/CD flow, and implementation details for the **Task Management System**, a microservice-based application built with **NestJS**, **GraphQL**, **Redis**, **Kafka**, **Docker**, **Kubernetes**, **NGINX**, and **GitHub Actions**. The system enables users to create, assign, and track tasks with real-time collaboration features. Services communicate internally via TCP and Kafka, with a GraphQL API exposed to the frontend.

## API Architecture

### Base Configuration
- **API Version**: Embedded in route structure (e.g., `/graphql/v1`)
- **Response Format**: JSON (via GraphQL)
- **Authentication**: JWT-based authentication managed by the User Service
- **Error Handling**: Standardized GraphQL error responses with codes and messages
- **Communication**:
  - **TCP**: For direct service-to-service communication (e.g., Task Service to Notification Service)
  - **Kafka**: For event-driven asynchronous communication
  - **Redis**: For caching and pub/sub real-time updates

### System Components
- **Microservices**:
  - **User Service**: Manages authentication and user profiles
  - **Task Service**: Handles task CRUD operations
  - **Notification Service**: Sends real-time notifications
  - **API Gateway**: Exposes GraphQL API and routes requests
- **Frontend**: React/Next.js with Apollo Client
- **Database**: PostgreSQL/MongoDB for persistent storage
- **Infrastructure**: Docker, Kubernetes, NGINX

## API Routes

### GraphQL API (API Gateway)
- **Endpoint**: `/graphql/v1`
- **Operations**:
  - **Queries**:
    - `getUser(id: ID!): User`: Fetch user details
    - `getTasks(filter: TaskFilter): [Task]`: List tasks with optional filters (e.g., status, assignee)
    - `getTask(id: ID!): Task`: Fetch a single task
  - **Mutations**:
    - `createUser(input: CreateUserInput): User`: Register a new user
    - `login(email: String!, password: String!): AuthPayload`: Authenticate user and return JWT
    - `createTask(input: CreateTaskInput): Task`: Create a new task
    - `updateTask(id: ID!, input: UpdateTaskInput): Task`: Update task details
    - `deleteTask(id: ID!): Boolean`: Delete a task
  - **Subscriptions**:
    - `taskUpdated(taskId: ID!): Task`: Real-time updates for task changes
    - `notificationReceived(userId: ID!): Notification`: Real-time notifications for users

### Internal Service Communication
- **TCP**:
  - Used for direct, low-latency communication (e.g., Task Service notifies Notification Service of task updates)
  - Example: Task Service sends a TCP message with task ID and status
- **Kafka**:
  - Topics: `task.created`, `task.updated`, `task.deleted`
  - Consumers: Notification Service, analytics services
- **Redis Pub/Sub**:
  - Channels: `notifications:<userId>`, `task:<taskId>`
  - Used for real-time updates to frontend via subscriptions

## Security Considerations

1. **Authentication**:
   - JWT tokens issued by User Service, validated by API Gateway
   - Role-based access control (e.g., admin, user)
   - Tokens stored in Redis for session management

2. **Data Validation**:
   - Input validation using NestJS pipes and GraphQL schema
   - Sanitization of user inputs to prevent injection attacks
   - File upload restrictions (if applicable)

3. **Rate Limiting**:
   - Per-user rate limits on GraphQL queries/mutations
   - IP-based limiting via NGINX
   - Burst protection for API Gateway

4. **Error Handling**:
   - Standardized GraphQL error responses (e.g., `{ code: "UNAUTHORIZED", message: "Invalid token" }`)
   - No sensitive information exposed in errors
   - Centralized logging for errors (via Kafka or external service)

## Performance Optimization

1. **Caching Strategy**:
   - Redis caching for user sessions and task metadata
   - Cache TTL: 5 minutes for sessions, 10 minutes for tasks
   - GraphQL query caching at API Gateway for frequently accessed data

2. **Pagination**:
   - Cursor-based pagination for task lists
   - Default page size: 20 items, configurable up to 100

3. **Event-Driven Processing**:
   - Kafka for asynchronous task processing (e.g., notifications, analytics)
   - Reduces latency in critical API paths

## Testing Strategy

1. **Unit Tests**:
   - Service logic (e.g., task creation, user authentication)
   - GraphQL resolvers and schema validation
   - TCP client/server communication

2. **Integration Tests**:
   - API Gateway endpoints (GraphQL queries, mutations, subscriptions)
   - Service-to-service communication (TCP, Kafka)
   - Redis caching and pub/sub

3. **E2E Tests**:
   - User flows (e.g., register, create task, receive notification)
   - Real-time updates via subscriptions
   - Authentication and authorization scenarios

## Monitoring

1. **Metrics**:
   - API response times (GraphQL queries/mutations)
   - Service uptime and error rates
   - Kafka consumer lag and throughput

2. **Logging**:
   - Request/response logging for GraphQL API
   - Error tracking for microservices
   - Performance metrics for TCP and Kafka communication

## Development Guidelines

1. **Code Organization**:
   - Monorepo structure with separate folders for each service (`/services/user-service`, `/services/task-service`, etc.)
   - Shared utilities in `/common` directory
   - Type definitions in `/types` directory
   - Infrastructure configs in `/infra` (Dockerfiles, Kubernetes manifests)

2. **Naming Conventions**:
   - GraphQL schema: CamelCase for types, fields, and arguments
   - Kafka topics: Kebab-case (e.g., `task.created`)
   - Redis keys: Namespaced (e.g., `user:session:<id>`)

3. **CI/CD**:
   - GitHub Actions for automated testing, building, and deployment
   - Pipeline stages: lint, test, build, push Docker images, deploy to Kubernetes
   - Environment-specific configurations (dev, staging, prod)