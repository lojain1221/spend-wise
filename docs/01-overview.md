# 01 — Overview

## What is SpendWise?
SpendWise is a web application for tracking income and expenses, setting monthly category budgets, and receiving real-time spending alerts.

## How it works
1. A user signs in.
2. They create monthly budgets, such as €200 for groceries.
3. They add an expense and select its category.
4. The expense is saved and the budget total is updated asynchronously.
5. If spending crosses a warning threshold (for example 80%) or exceeds the limit, the user sees an alert without refreshing the browser.
6. The dashboard shows recorded transactions and the latest budget totals.

## Features
- Basic user accounts and sign-in
- Add/list/delete transactions (income or expense)
- Create/view monthly budgets by category
- Calculate spending against a budget
- Real-time warning and over-budget notifications

## Not included
Bank integrations, currency conversion, payment processing, shared household accounts, advanced analytics, and mobile apps.

## System components and communication
| Requirement | Design |
| --- | --- |
| At least four microservices | User, Transaction, Budget, Notification |
| At least one client | Browser-based web client |
| API Gateway | Routes REST requests and proxies WebSocket connections |
| Synchronous communication | REST between client/gateway/services and one internal service lookup |
| Asynchronous communication | RabbitMQ events between services |
| Real-time updates | Notification Service sends WebSocket alerts to browser via Gateway |
| Diagrams | Mermaid architecture and sequence diagrams |
