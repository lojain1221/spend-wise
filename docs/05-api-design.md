# 05 — API Design

## API Overview

SpendWise uses an API Gateway to handle requests from the web client and forward them to the appropriate microservice.

Each microservice manages its own functionality. REST APIs are used for synchronous communication, RabbitMQ is used for asynchronous events, and WebSocket is used for real-time notifications.

## API Endpoints

| Method | Path | Service | Purpose |
| --- | --- | --- | --- |
| POST | `/auth/register` | User Service | Create a new account |
| POST | `/auth/login` | User Service | Log in to an account |
| GET | `/users/me` | User Service | Get user profile |
| GET | `/transactions` | Transaction Service | View transactions |
| POST | `/transactions` | Transaction Service | Add income or expense |
| PUT | `/transactions/{id}` | Transaction Service | Update a transaction |
| DELETE | `/transactions/{id}` | Transaction Service | Delete a transaction |
| GET | `/categories` | Transaction Service | View categories |
| POST | `/categories` | Transaction Service | Create a category |
| GET | `/budgets` | Budget Service | View monthly budgets |
| POST | `/budgets` | Budget Service | Create a monthly budget |
| GET | `/notifications` | Notification Service | View notifications |
| PATCH | `/notifications/{id}/read` | Notification Service | Mark a notification as read |

## WebSocket Communication

SpendWise uses WebSocket to send budget notifications to users in real time.

The web client connects through the API Gateway, which forwards the WebSocket connection to the Notification Service.

| Protocol | Endpoint | Service | Purpose |
| --- | --- | --- | --- |
| WebSocket | `/ws/notifications` | Notification Service | Send real-time budget alerts |

When a budget threshold is reached, the Notification Service sends an alert to the connected user.

## Internal Service Communication

The microservices communicate using REST and RabbitMQ.

### REST Communication

The Transaction Service communicates with the User Service to verify that a user account exists.

### RabbitMQ Communication

- The Transaction Service publishes events when transactions are created, updated, or deleted.
- The Budget Service receives transaction events and updates spending totals.
- When a budget threshold is reached, the Budget Service publishes an event.
- The Notification Service receives the event and creates a notification.

## Authentication and Security

- User passwords are stored as hashes.
- The User Service handles registration and login.
- Access tokens are used to authenticate requests.
- Users can access only their own transactions, budgets, and notifications.
- WebSocket connections require authentication before notifications are delivered.
