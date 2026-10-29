# 06 — Design decisions

## Why this project?
An expense updates a budget and may trigger an alert. This creates a clear workflow across independent services without bank integrations or payment processing.

## Why four services?
- **User:** identity and profiles
- **Transaction:** the source of truth for income and expense entries
- **Budget:** limits and calculated spending
- **Notification:** alert storage and live delivery

Each service has a distinct responsibility and its own database.

## Why FastAPI and JavaScript?
Python/FastAPI can support REST and WebSockets with a relatively simple codebase. A plain HTML/CSS/JavaScript browser client keeps the UI lightweight. 

## Why REST plus RabbitMQ?
REST provides clear request/response interactions. RabbitMQ allows budget calculations and notifications to happen after a transaction is saved, without blocking the user-facing request.

## Why WebSockets?
A persistent browser connection allows an alert to appear without reloading. This is more meaningful here than repeatedly polling for notifications.

## Why an API Gateway?
It provides one browser-facing entry point and routes HTTP and WebSocket traffic. It is infrastructure, not a fifth business microservice.

## Known trade-offs
- Event-driven totals are eventually consistent.
- A message broker adds setup complexity.
- Authentication across services requires careful handling.
- SQLite is easy for development but would not be the choice for a scaled deployment.
- Reliable event publication and idempotent consumption require extra implementation work.

## Development steps
1. Implement User and Transaction services with REST endpoints.
2. Add Budget and Notification services and independent databases.
3. Connect RabbitMQ events and event consumers.
4. Add Gateway routing, WebSocket proxy, and browser client.
5. Test transaction → budget → alert end to end.

