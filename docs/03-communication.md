03 — Communication

Communication Styles

SpendWise uses REST, RabbitMQ, and WebSocket for communication between the client and microservices.

* REST: The web client sends requests through the API Gateway to the appropriate service.
* RabbitMQ: Services exchange events asynchronously. The Transaction Service publishes transaction events, and the Budget Service processes them to update spending totals.
* WebSocket: The Notification Service sends real-time alerts to the web client through the API Gateway.

Add Expense Sequence

sequenceDiagram
    actor Person as User
    participant Web as Web Client
    participant GW as API Gateway
    participant TX as Transaction Service
    participant MQ as RabbitMQ
    participant BU as Budget Service
    participant NO as Notification Service
    Person->>Web: Add €30 grocery expense
    Web->>GW: POST /transactions
    GW->>TX: Forward request
    TX->>TX: Save expense
    TX-->>GW: 201 Created
    GW-->>Web: Transaction saved
    TX->>MQ: Publish TransactionCreated
    MQ-->>BU: Deliver TransactionCreated
    BU->>BU: Update spending total
    opt Budget threshold reached
        BU->>MQ: Publish BudgetThresholdReached
        MQ-->>NO: Deliver event
        NO->>NO: Save notification
        NO-->>GW: WebSocket message
        GW-->>Web: Forward notification
        Web-->>Person: Display budget warning
    end

Events

Event	Publisher	Consumer	Purpose
TransactionCreated	Transaction Service	Budget Service	Add a transaction to spending totals
TransactionUpdated	Transaction Service	Budget Service	Update spending totals when a transaction changes
TransactionDeleted	Transaction Service	Budget Service	Adjust spending totals when a transaction is removed
BudgetThresholdReached	Budget Service	Notification Service	Generate a budget warning

Event Data

TransactionCreated

* event_id
* transaction_id
* user_id
* category_id
* amount
* currency
* occurred_at

TransactionUpdated

* event_id
* transaction_id
* user_id
* previous_amount
* new_amount
* category_id
* currency
* occurred_at

TransactionDeleted

* event_id
* transaction_id
* user_id
* category_id
* amount
* currency
* occurred_at

BudgetThresholdReached

* event_id
* budget_id
* user_id
* period
* threshold
* spent
* limit

Event Processing

* RabbitMQ delivers transaction events to the Budget Service.
* The Budget Service updates spending totals when transactions are created, updated, or deleted.
* A budget notification is generated when spending crosses 80% or 100% of the budget limit.
* The Notification Service stores alerts and sends them through WebSocket.
* Event IDs are used to prevent duplicate processing if RabbitMQ delivers the same event more than once.
