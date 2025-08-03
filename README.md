# 📋 Task Management System

The **Task Management System** is a microservice-based application built to manage tasks with real-time collaboration features. It leverages **NestJS** for scalable microservices, **GraphQL** for a flexible API, **Redis** and **Kafka** for caching and event-driven communication, **Docker** and **Kubernetes** for containerization and orchestration, and **GitHub Actions** for CI/CD. Services communicate internally via **TCP** for specific interactions, with **NGINX** as a reverse proxy. This project demonstrates modern software engineering practices and is ideal for showcasing in a portfolio.

---

## ✨ Features

- 📝 **Task Management** — Create, update, assign, and delete tasks
- 👥 **User Management** — Register, login, and manage user profiles with JWT authentication
- 🔄 **Real-Time Notifications** — Receive live updates for task changes via Redis Pub/Sub or WebSockets
- 🚀 **Event-Driven Architecture** — Use Kafka for asynchronous task events (e.g., logging, notifications)
- 🛠️ **GraphQL API** — Flexible queries, mutations, and subscriptions for frontend interaction
- 🐳 **Containerized Deployment** — Dockerized services orchestrated with Kubernetes
- ⚡ **Scalable Infrastructure** — NGINX for load balancing and GitHub Actions for CI/CD

---

## 🧰 Tech Stack

| Area                     | Tech Used                          |
|--------------------------|------------------------------------|
| **Backend Framework**    | NestJS (TypeScript)                |
| **API**                  | GraphQL                            |
| **Database**             | PostgreSQL/MongoDB                 |
| **Caching/Pub-Sub**      | Redis                              |
| **Event Streaming**      | Kafka                              |
| **Service Communication**| TCP (Node.js `net` module)         |
| **Frontend**             | React/Next.js + Apollo Client      |
| **Containerization**     | Docker                             |
| **Orchestration**        | Kubernetes                         |
| **Reverse Proxy**        | NGINX                              |
| **CI/CD**                | GitHub Actions                     |

---

## Prerequisites

- Node.js 18+
- Docker Desktop (for local containerization)
- Minikube or Kind (for local Kubernetes testing)
- PostgreSQL or MongoDB (local or cloud instance)
- Redis and Kafka (local or cloud-hosted, e.g., Redis Labs, Confluent Cloud)
- GitHub account for CI/CD
- NGINX installed or configured in Kubernetes
- Kubectl and Helm (optional for Kubernetes management)

## Getting Started

1. **Clone the repository**
```bash
git clone https://github.com/your-username/task-management-system.git
cd task-management-system
```

2. **Install dependencies**
```bash
# For monorepo (e.g., using Nx or Turborepo)
npm install
# Or install per service
cd services/user-service && npm install
cd ../task-service && npm install
cd ../notification-service && npm install
cd ../api-gateway && npm install
cd ../../frontend && npm install
```

3. **Set up environment variables**
Create a `.env` file in each service directory (`services/user-service`, etc.) and the frontend:
```env
# User Service
DATABASE_URL=postgresql://user:password@localhost:5432/taskdb
REDIS_URL=redis://localhost:6379
JWT_SECRET=your_jwt_secret

# Task Service
DATABASE_URL=postgresql://user:password@localhost:5432/taskdb
REDIS_URL=redis://localhost:6379
KAFKA_BROKER=localhost:9092

# Notification Service
REDIS_URL=redis://localhost:6379
KAFKA_BROKER=localhost:9092
TCP_PORT=3001

# API Gateway
GRAPHQL_PORT=4000
USER_SERVICE_URL=http://user-service:3000
TASK_SERVICE_URL=http://task-service:3000

# Frontend
NEXT_PUBLIC_GRAPHQL_URL=http://localhost:4000/graphql
```

4. **Run locally (without Docker)**
```bash
# Start each service
cd services/user-service && npm run start:dev
cd ../task-service && npm run start:dev
cd ../notification-service && npm run start:dev
cd ../api-gateway && npm run start:dev
# Start frontend
cd ../../frontend && npm run dev
```

5. **Run with Docker**
```bash
docker-compose up --build
```

6. **Deploy to Kubernetes**
```bash
# Start Minikube
minikube start
# Apply Kubernetes manifests
kubectl apply -f infra/k8s/
# Set up NGINX Ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

## Project Structure

```
task-management-system/
├── services/
│   ├── user-service/         # User management (auth, profiles)
│   ├── task-service/         # Task CRUD operations
│   ├── notification-service/ # Real-time notifications
│   ├── api-gateway/          # GraphQL API and routing
├── frontend/                 # React/Next.js frontend
├── infra/
│   ├── docker/               # Dockerfiles for each service
│   ├── k8s/                  # Kubernetes manifests
├── .github/workflows/        # GitHub Actions CI/CD configs
├── README.md                 # Project documentation
```

## API Documentation

The API uses GraphQL with queries, mutations, and subscriptions for:
- User management (e.g., `register`, `login`, `getUser`)
- Task management (e.g., `createTask`, `updateTask`, `deleteTask`)
- Real-time updates (e.g., `taskUpdated` subscription)

For detailed API documentation, see [api-design.md](./docs/api-design.md).

## Database Schema

The application uses PostgreSQL/MongoDB with the following collections/tables:
- **Users**: Stores user data (ID, email, password hash, role)
- **Tasks**: Stores task data (ID, title, description, assignee, status)
- **Notifications**: Stores notification logs (ID, user, message, timestamp)

For detailed schema information, see [database-structure.md](./docs/database-structure.md).

## Development Guidelines

1. **Code Style**
   - Use TypeScript for all services and frontend code
   - Follow NestJS module structure
   - Use ESLint and Prettier for consistent formatting
   - Implement proper error handling with try-catch

2. **Git Workflow**
   - Create feature branches from `main`
   - Use conventional commits (e.g., `feat: add task creation endpoint`)
   - Submit PRs with clear descriptions
   - Ensure tests pass before merging

3. **Testing**
   - Write unit tests for services using Jest
   - Include integration tests for GraphQL endpoints
   - Test frontend components with React Testing Library
   - Verify TCP and Kafka communication with mock servers

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is proprietary software. All rights reserved.

## Support

For support or inquiries, please contact the repository owner via GitHub Issues.