# Database Structure Documentation

## Overview
This document outlines the database structure for the Task Management System, a microservice-based application for managing tasks with real-time collaboration. The application uses **PostgreSQL** as the primary database for persistent storage, **Redis** for caching and pub/sub, and **Kafka** for event-driven communication.

## Entities

### Users
- **Table**: `users`
- **Description**: Stores user information and profile details for authentication and authorization.
- **Fields**:
  - `id`: UUID (Primary Key)
  - `name`: String (required)
  - `email`: String (required)
  - `password_hash`: String (required, hashed password)
  - `role`: String (enum: ['user', 'admin'], default: 'user')
  - `created_at`: Timestamp
  - `updated_at`: Timestamp
- **Indexes**:
  - `{ email: 1 }` (unique)
  - `{ id: 1 }`

### Tasks
- **Table**: `tasks`
- **Description**: Stores task information, including details and status.
- **Fields**:
  - `id`: UUID (Primary Key)
  - `title`: String (required)
  - `description`: String
  - `status`: String (enum: ['pending', 'in_progress', 'completed'], default: 'pending')
  - `assigned_user_id`: UUID (ref: 'Users', nullable)
  - `created_by`: UUID (ref: 'Users')
  - `due_date`: Timestamp (nullable)
  - `created_at`: Timestamp
  - `updated_at`: Timestamp
- **Indexes**:
  - `{ id: 1 }`
  - `{ assigned_user_id: 1 }`
  - `{ created_by: 1 }`

### Notifications
- **Table**: `notifications`
- **Description**: Stores notification details for task updates or assignments.
- **Fields**:
  - `id`: UUID (Primary Key)
  - `user_id`: UUID (ref: 'Users')
  - `task_id`: UUID (ref: 'Tasks')
  - `message`: String (required)
  - `type`: String (enum: ['info', 'warning', 'alert'], default: 'info')
  - `read_status`: Boolean (default: false)
  - `created_at`: Timestamp
- **Indexes**:
  - `{ id: 1 }`
  - `{ user_id: 1 }`
  - `{ task_id: 1 }`

### Task Events
- **Kafka Topic**: `task_events`
- **Description**: Stores task-related events for asynchronous processing (e.g., logging, analytics, notifications).
- **Fields**:
  - `event_id`: UUID
  - `task_id`: UUID (ref: 'Tasks')
  - `event_type`: String (enum: ['created', 'updated', 'assigned', 'completed'])
  - `payload`: JSON (details of the event, e.g., task title, status)
  - `timestamp`: Timestamp
- **Partitions**: 4 (for scalability)

### Redis Cache
- **Structure**: Key-Value Store
- **Description**: Used for caching user sessions and task metadata, and for pub/sub to enable real-time notifications.
- **Keys**:
  - `session:<user_id>`: Stores JWT session data (TTL: 24 hours)
  - `task:<task_id>`: Stores cached task metadata (TTL: 1 hour)
  - `pubsub:notifications`: Channel for real-time notification updates

## Relationships

1. **Users -> Tasks (Created By)**:
   - One-to-Many relationship.
   - A user can create multiple tasks, but each task is created by one user.

2. **Users -> Tasks (Assigned To)**:
   - One-to-Many relationship.
   - A user can be assigned multiple tasks, but each task is assigned to one user (or none).

3. **Tasks -> Notifications**:
   - One-to-Many relationship.
   - A task can generate multiple notifications (e.g., for assignment or status updates), but each notification is linked to one task.

4. **Users -> Notifications**:
   - One-to-Many relationship.
   - A user can receive multiple notifications, but each notification is linked to one user.

## Notes

1. **Timestamps**: All tables include `created_at` and `updated_at` timestamps for tracking record creation and updates.
2. **Enums**: Fields like `status` in `tasks` and `type` in `notifications` use ENUM types to ensure data consistency.
3. **Real-Time Updates**: Redis pub/sub is used for real-time notifications, triggered when tasks are updated or assigned.
4. **Event-Driven Architecture**: Kafka is used to publish task events (e.g., creation, assignment) for asynchronous processing, such as logging or sending notifications.
5. **Caching**: Redis caches user sessions and task metadata to reduce database load and improve performance.
6. **Scalability**: The schema is normalized to reduce redundancy, and Kafka partitions ensure scalability for event processing.
7. **Indexes**: Indexes are defined for frequently queried fields (e.g., `email`, `user_id`, `task_id`) to optimize performance.
8. **Extensibility**: The schema can be extended with additional tables (e.g., for task comments or analytics) or fields (e.g., task priority) as needed.
9. **Microservices**: Each microservice (User Service, Task Service, Notification Service) interacts with specific tables:
   - **User Service**: Manages `users` table.
   - **Task Service**: Manages `tasks` table and publishes to `task_events` Kafka topic.
   - **Notification Service**: Manages `notifications` table and consumes from `task_events` Kafka topic.
10. **TCP Communication**: Services communicate directly via TCP for specific operations (e.g., Task Service sends task updates to Notification Service for immediate processing).