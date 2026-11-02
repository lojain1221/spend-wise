# SpendWise — Personal Budget Management

Neptun Code: BNERC9

SpendWise is a microservice-based web application for tracking income and expenses, managing monthly budgets, and receiving real-time spending alerts. It uses REST APIs, RabbitMQ events, and WebSockets to connect four independent services.

## Components

- Browser client (HTML/CSS/JavaScript)
- API Gateway (FastAPI; not counted among the four services)
- User Service
- Transaction Service
- Budget Service
- Notification Service
- RabbitMQ for asynchronous events and WebSocket for real-time alerts

## Documentation

1. [Overview](docs/01-overview.md)
2. [Architecture](docs/02-architecture.md)
3. [Communication](docs/03-communication.md)
4. [Data Model](docs/04-data-model.md)
5. [API Design](docs/05-api-design.md)
6. [Design Decisions](docs/06-design-decisions.md)

Architecture, communication, and data model diagrams use Mermaid.
