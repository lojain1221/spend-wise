# 05 — API Design

## API Overview

The web client accesses SpendWise services through the API Gateway.

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

## WebSocket Endpoint

| Protocol | Endpoint | Service | Purpose |
| --- | --- | --- | --- |
| WebSocket | `/ws/notifications` | Notification Service | Deliver real-time budget alerts |

## Authentication and Security

- Registration and login are handled by the User Service.
- Passwords are stored as hashes.
- Access tokens are required for protected endpoints.
- Users can access only their own transactions, budgets, and notifications.
- WebSocket connections require authentication.
