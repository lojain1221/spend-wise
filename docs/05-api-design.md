# 05 — API Design

## API Overview

SpendWise uses REST APIs for communication between the web client and microservices.

The web client sends requests through the API Gateway, which forwards them to the appropriate service.

The application also uses WebSocket to send real-time budget notifications.

## API Endpoints

### User Service

The User Service handles account registration, login, and user information.

| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/register` | Register a new user |
| POST | `/auth/login` | Log in and receive an access token |
| GET | `/users/me` | Get the current user's profile |

### Transaction Service

The Transaction Service manages income, expenses, and categories.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/transactions` | Get the user's transactions |
| POST | `/transactions` | Add a new income or expense |
| PUT | `/transactions/{id}` | Update an existing transaction |
| DELETE | `/transactions/{id}` | Delete a transaction |
| GET | `/categories` | Get the user's categories |
| POST | `/categories` | Create a category |
| DELETE | `/categories/{id}` | Delete a category |

### Budget Service

The Budget Service manages monthly budgets and spending totals.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/budgets` | Get the user's monthly budgets |
| POST | `/budgets` | Create a monthly budget |
| PUT | `/budgets/{id}` | Update a budget limit |
| DELETE | `/budgets/{id}` | Delete a budget |

### Notification Service

The Notification Service stores notifications and sends real-time alerts.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/notifications` | Get the user's notifications |
| PATCH | `/notifications/{id}/read` | Mark a notification as read |
| WS | `/ws/notifications` | Receive real-time notifications |

## Example API Requests

### 1. Register a User

**Endpoint:** `POST /auth/register`

Request:

```json
{
  "email": "student@example.com",
  "password": "examplePassword123",
  "display_name": "Student"
}
```

Response:

```json
{
  "id": "user-uuid",
  "email": "student@example.com",
  "display_name": "Student"
}
```

### 2. User Login

**Endpoint:** `POST /auth/login`

Request:

```json
{
  "email": "student@example.com",
  "password": "examplePassword123"
}
```

Response:

```json
{
  "access_token": "example-jwt-token",
  "token_type": "bearer"
}
```

The access token is used to authenticate requests to protected endpoints.

### 3. Add an Expense

**Endpoint:** `POST /transactions`

Request:

```json
{
  "category_id": "category-uuid",
  "type": "expense",
  "amount_cents": 3000,
  "occurred_at": "2026-11-02T12:00:00Z",
  "note": "Groceries"
}
```

Response:

```json
{
  "id": "transaction-uuid",
  "category_id": "category-uuid",
  "type": "expense",
  "amount_cents": 3000,
  "occurred_at": "2026-11-02T12:00:00Z",
  "note": "Groceries"
}
```

After saving the transaction, the Transaction Service publishes a `TransactionCreated` event to RabbitMQ.

The Budget Service receives the event and updates the spending total.

### 4. Create a Budget

**Endpoint:** `POST /budgets`

Request:

```json
{
  "category_id": "category-uuid",
  "month": "2026-11",
  "limit_cents": 30000
}
```

Response:

```json
{
  "id": "budget-uuid",
  "category_id": "category-uuid",
  "month": "2026-11",
  "limit_cents": 30000,
  "spent_cents": 0
}
```

The Budget Service stores the monthly spending limit.

When transactions are added, updated, or deleted, the spending total is adjusted using events received from RabbitMQ.

### 5. Get Notifications

**Endpoint:** `GET /notifications`

Response:

```json
[
  {
    "id": "notification-uuid",
    "budget_id": "budget-uuid",
    "kind": "budget_warning",
    "message": "You have reached 80% of your grocery budget.",
    "created_at": "2026-11-02T12:05:00Z",
    "read_at": null
  }
]
```

Users can retrieve previous notifications even if they were not connected when the alert was generated.

## WebSocket Communication

**Endpoint:** `/ws/notifications`

The web client connects to the Notification Service through the API Gateway.

When a budget threshold is reached:

1. The Budget Service publishes a `BudgetThresholdReached` event.
2. RabbitMQ delivers the event to the Notification Service.
3. The Notification Service saves the notification.
4. The notification is sent through WebSocket.
5. The web client displays the alert.

Example WebSocket message:

```json
{
  "type": "budget_warning",
  "budget_id": "budget-uuid",
  "message": "You have reached 80% of your grocery budget."
}
```

## HTTP Status Codes

| Status Code | Meaning |
|---|---|
| 200 OK | Request completed successfully |
| 201 Created | New record created |
| 204 No Content | Record deleted successfully |
| 400 Bad Request | Invalid request data |
| 401 Unauthorized | Missing or invalid authentication |
| 403 Forbidden | User does not have permission |
| 404 Not Found | Requested record does not exist |
| 409 Conflict | Record conflicts with existing data |
| 500 Internal Server Error | Unexpected server error |

## Security

- Passwords are stored as secure password hashes.
- The User Service handles authentication and issues signed access tokens.
- Protected REST endpoints require a valid access token.
- Each service checks that users can access only their own data.
- User IDs are obtained from the authenticated token rather than the request body.
- WebSocket connections require authentication before notifications are delivered.
- Communication between the browser and API Gateway uses HTTPS and secure WebSocket connections in deployment.

## Internal Service Communication

Some communication happens directly between microservices.

| Source | Destination | Method | Purpose |
|---|---|---|---|
| Transaction Service | User Service | REST | Check whether a user exists |
| Transaction Service | RabbitMQ | Event | Publish transaction changes |
| RabbitMQ | Budget Service | Event | Deliver transaction events |
| Budget Service | RabbitMQ | Event | Publish budget threshold events |
| RabbitMQ | Notification Service | Event | Deliver budget alerts |

The services do not access each other's databases directly.
