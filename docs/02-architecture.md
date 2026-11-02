# 02 — Architecture

## Overview

SpendWise is organized into four microservices, a web client, an API Gateway, and RabbitMQ.

Each microservice handles a specific part of the application and has its own database.

## Service Responsibilities

| Component | Responsibility |
| --- | --- |
| Web Client | Interface for managing transactions, budgets, and notifications |
| API Gateway | Routes requests and WebSocket connections |
| User Service | Manages accounts, login, and user profiles |
| Transaction Service | Manages income, expenses, and categories |
| Budget Service | Manages monthly budgets and spending totals |
| Notification Service | Stores notifications and delivers alerts |
| RabbitMQ | Transfers events between microservices |

## Architecture Diagram

```mermaid
flowchart TB
    Browser["Web Client"]
    Gateway["API Gateway"]

    Users["User Service"]
    Transactions["Transaction Service"]
    Budgets["Budget Service"]
    Notifications["Notification Service"]

    MQ[("RabbitMQ")]

    UDB[("Users DB")]
    TDB[("Transactions DB")]
    BDB[("Budgets DB")]
    NDB[("Notifications DB")]

    Browser -->|"HTTP / REST"| Gateway
    Browser <-->|"WebSocket"| Gateway

    Gateway -->|"REST"| Users
    Gateway -->|"REST"| Transactions
    Gateway -->|"REST"| Budgets
    Gateway -->|"REST"| Notifications

    Gateway <-->|"WebSocket proxy"| Notifications

    Transactions -->|"REST: User validation"| Users

    Users --> UDB
    Transactions --> TDB
    Budgets --> BDB
    Notifications --> NDB

    Transactions -->|"Transaction events"| MQ
    MQ -->|"Transaction events"| Budgets

    Budgets -->|"BudgetThresholdReached"| MQ
    MQ -->|"Budget alert"| Notifications
```
