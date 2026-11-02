# 06 — Design Decisions

## Project Choice

SpendWise was chosen because managing personal expenses involves several connected functions, including transactions, budgets, and notifications.

These functions provide a practical example of how microservices can work together in one application.

## Microservice Architecture

The application is divided into four microservices to separate different responsibilities.

This structure allows each service to be developed and tested independently.

An API Gateway is used to provide a single entry point for the web client.

## Technology Choices

### Backend — FastAPI

FastAPI was selected because it supports REST APIs and WebSocket communication using Python.

It is also relatively simple to use for developing small web services.

### Frontend — HTML, CSS, and JavaScript

These technologies were selected to build a browser-based interface without requiring a frontend framework.

### Database — SQLite

SQLite was chosen because it is lightweight and easy to configure for development.

Each microservice uses its own database to keep its data independent from other services.

## Communication Decisions

### REST

REST is used for operations that require an immediate response, such as creating a transaction or retrieving user information.

### RabbitMQ

RabbitMQ allows services to exchange events without waiting for other services to complete their processing.

This means budget calculations can happen after a transaction has been saved.

### WebSocket

WebSocket was selected to display budget notifications immediately without requiring users to refresh the page.

## Design Trade-offs

- Using separate microservices requires more configuration than a single application.
- RabbitMQ adds an additional component that needs to be managed.
- Budget totals may not update immediately because events are processed asynchronously.
- Separate databases prevent services from directly accessing each other's data.
- SQLite is suitable for this project's development needs but has limitations for applications with many concurrent users.
