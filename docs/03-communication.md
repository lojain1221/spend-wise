# 03 — Communication

## Communication Styles

SpendWise uses REST, RabbitMQ, and WebSocket for communication between the web client and microservices.

- **REST:** The web client sends HTTP requests through the API Gateway to the appropriate service. The Transaction Service also uses REST to check whether a user exists before saving a transaction.
- **RabbitMQ:** Services exchange events asynchronously. The Transaction Service publishes transaction events, and the Budget Service processes them to update spending totals.
- **WebSocket:** The Notification Service sends real-time budget alerts to the web client through the API Gateway.

## Add Expense Sequence

The following diagram shows what happens when a user adds an expense and the spending amount reaches a budget threshold.

```mermaid
sequenceDiagram
    actor Person as User
    participant Web as Web Client
    participant GW as API Gateway
    participant TX as Transaction Service
    participant US as User Service
    participant MQ as RabbitMQ
    participant BU as Budget Service
    participant NO as Notification Service

    Person->>Web: Enter €30 grocery expense
    Web->>GW: POST /transactions
    GW->>TX: Forward request
    TX->>US: Check if user exists
    US-->>TX: User exists
    TX->>TX: Save expense
    TX-->>GW: 201 Created
    GW-->>Web: Transaction saved
    TX->>MQ: Publish TransactionCreated
    MQ-->>BU: Deliver TransactionCreated
    BU->>BU: Update monthly spending total

    opt Budget threshold reached
        BU->>MQ: Publish BudgetThresholdReached
        MQ-->>NO: Deliver event
        NO->>NO: Save notification
        NO-->>GW: WebSocket message
        GW-->>Web: Forward notification
        Web-->>Person: Display budget warning
    end
```

## Events

The services use RabbitMQ to exchange events when transactions are created, updated, or deleted and when a budget threshold is reached.

| Event | Publisher | Consumer | Purpose |
|---|---|---|---|
| TransactionCreated | Transaction Service | Budget Service | Add a new transaction to spending totals |
| TransactionUpdated | Transaction Service | Budget Service | Update spending totals when a transaction changes |
| TransactionDeleted | Transaction Service | Budget Service | Adjust spending totals when a transaction is deleted |
| BudgetThresholdReached | Budget Service | Notification Service | Create a budget warning |

### Event Data

Each event contains the information needed by the receiving service.

**TransactionCreated**
- event_id
- transaction_id
- user_id
- category_id
- amount
- currency
- occurred_at

**TransactionUpdated**
- event_id
- transaction_id
- user_id
- old_category_id
- new_category_id
- old_amount
- new_amount
- currency
- occurred_at

**TransactionDeleted**
- event_id
- transaction_id
- user_id
- category_id
- amount
- currency
- occurred_at

**BudgetThresholdReached**
- event_id
- budget_id
- user_id
- period
- threshold
- spent
- limit

## Communication Flow

### 1. Adding an Expense

1. The user enters an expense in the web client.
2. The web client sends the request to the API Gateway.
3. The Gateway forwards the request to the Transaction Service.
4. The Transaction Service checks the user through the User Service.
5. The Transaction Service saves the expense in its database.
6. The Transaction Service publishes a TransactionCreated event to RabbitMQ.
7. The Budget Service receives the event and updates the spending total.
8. If a budget threshold is reached, the Budget Service publishes a BudgetThresholdReached event.
9. The Notification Service receives the event and saves a notification.
10. The notification is sent to the web client through WebSocket.

### 2. Updating or Deleting an Expense

1. The user updates or deletes an existing expense.
2. The Transaction Service saves the change in its database.
3. The Transaction Service publishes a TransactionUpdated or TransactionDeleted event.
4. The Budget Service receives the event and adjusts the spending total.
5. The updated spending information is available to the web client through REST.

### 3. Real-Time Notifications

1. The web client opens a WebSocket connection through the API Gateway.
2. The Gateway forwards the connection to the Notification Service.
3. When a budget threshold is reached, the Notification Service receives the event from RabbitMQ.
4. The Notification Service saves the notification in its database.
5. The notification is sent through the WebSocket connection.
6. The web client displays the warning to the user.

## Error Handling

- REST requests return HTTP status codes to indicate success or failure.
- If a service cannot process a RabbitMQ event, the message is not acknowledged until processing succeeds.
- Services use event IDs to avoid processing the same event more than once.
- If the WebSocket connection is lost, the client can reconnect and retrieve saved notifications through REST.
- Budget totals may take a short time to update because transaction events are processed asynchronously.

## Budget Thresholds

SpendWise generates notifications when spending reaches certain percentages of the budget limit.

| Threshold | Notification |
|---|---|
| 80% | Warning that the budget is almost reached |
| 100% | Alert that the budget limit has been reached or exceeded |

A notification is created when spending crosses a threshold, rather than every time a new transaction is added while spending remains above that threshold.
