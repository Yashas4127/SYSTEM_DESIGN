# Event-Driven Architecture (EDA) — Short Notes

## 1. What is Event-Driven Architecture (EDA)?

**Event-Driven Architecture (EDA)** is a software design pattern where systems communicate through **events** instead of direct API calls.

An **event** means something that happened in the system.

### Examples of Events

```text
UserRegistered
PaymentSucceeded
OrderPlaced
ItemOutOfStock
```

### Simple Idea

Instead of one service directly calling another service:

```text
Service A → Service B
```

Service A publishes an event. Any interested service can listen to that event and react.

```text
Service A → Event → Event Broker
                         ↓
                 Interested Services
```

### Key Idea

**EDA = Communication through events.**

---

# 2. Why Do We Need EDA?

In direct communication, services depend on each other.

```text
Order Service
     ↓
Inventory Service
     ↓
Billing Service
     ↓
Notification Service
```

If one service is slow or unavailable, it can affect the flow.

With EDA:

```text
Order Service
      ↓
  OrderPlaced
      ↓
  Event Broker
   ↙    ↓     ↘
Inventory Billing Notification
```

Services react independently instead of directly calling each other.

### Main Goals

- Loose coupling
- Asynchronous communication
- Scalability
- Independent service development
- Better isolation of failures

---

# 3. Core Components of EDA

There are three main components.

## 1. Event Producer (Publisher)

The service that detects something happened and publishes an event.

Example:

```text
Order Service → publishes OrderCreated
```

## 2. Event Broker / Message Bus

The middleman that routes and delivers events to interested consumers.

Examples:
- Kafka
- RabbitMQ
- Amazon SNS/SQS
- Google Pub/Sub
- Redis Streams

## 3. Event Consumer (Subscriber)

The service that listens for a specific event and takes action.

Example:

```text
Email Service listens for UserRegistered
and sends a welcome email.
```

### Architecture

```text
Producer
   ↓ Event
Event Broker
   ├──→ Consumer 1
   └──→ Consumer 2
```

### Important

**Producer and consumers do not need to communicate directly.**

---

# 4. Event Flow / Lifecycle

A typical event lifecycle:

1. Something happens — e.g. a user places an order.
2. Order Service publishes an `OrderPlaced` event to the broker.
3. The broker stores/routes and delivers the event to interested consumers.
4. Each consumer reacts independently.

### Example

```text
User places order
       ↓
Order Service publishes OrderPlaced
       ↓
Event Broker
   ↙       ↓        ↘
Inventory Billing Notification
   ↓        ↓           ↓
Reduce   Process      Send email
stock    payment
```

None of these services needs to directly call the others.

---

# 5. Event Communication Models

There are two models described in the notes.

## 1. Simple Event Notification

The event only signals that something happened. It does not contain all the extra data.

Example:

```text
"OrderPlaced"
```

The consumer may need to fetch more details from another service.

## 2. Event-Carried State Transfer

The event includes the details required by consumers.

Example:

```json
{
  "eventType": "OrderPlaced",
  "orderId": "1234",
  "items": [
    { "id": "A1", "qty": 2 }
  ],
  "userId": "U10"
}
```

Consumers can act immediately without fetching the same data again.

### Comparison

| Model | Event Contains | Consumer |
|---|---|---|
| Simple Event Notification | Event name / small notification | May fetch more data |
| Event-Carried State Transfer | Required event data | Can act using payload |

---

# 6. Event Brokers (Messaging Infrastructure)

An **Event Broker** receives events from producers and routes them to consumers.

| Broker | Type | Best For | Notes |
|---|---|---|---|
| **Kafka** | Stream | High-throughput event streams | Scalable, persistent, partitioned |
| **RabbitMQ** | Queue | Traditional message queue | Supports routing, ACK, retry |
| **AWS SNS/SQS** | Pub/Sub + Queue | Cloud-based decoupling | Managed AWS services |
| **Redis Streams** | Stream | Fast, lightweight messaging | In-memory, low latency |
| **Google Pub/Sub** | Pub/Sub | Google Cloud systems | Fully managed |

### Key Idea

The broker sits between producers and consumers and handles event delivery.

---

# 7. Real-World Use Cases

## 1. E-commerce Platform

Event:

```text
OrderPlaced
```

Consumers:

```text
Inventory → Update stock
Payment   → Process transaction
Shipping  → Create shipment
Email     → Send confirmation
Analytics → Record event
```

## 2. Banking

Event:

```text
TransactionCompleted
```

Fraud detection listens to the event and analyzes patterns asynchronously.

## 3. Ride-Hailing App

Events:

```text
DriverAssigned
RideStarted
RideCompleted
```

These events trigger notifications, billing, and analytics.

## 4. Social Media

Events:

```text
PostLiked
UserFollowed
```

These can update feeds, notifications, and recommendations.

---

# 8. Benefits of EDA

| Benefit | Explanation |
|---|---|
| **Loose Coupling** | Producers and consumers evolve independently |
| **Scalability** | Add more consumers when workload increases |
| **Asynchronous Processing** | Long-running work does not block the initial flow |
| **Resilience** | One consumer's failure need not stop other consumers |
| **Extensibility** | Add a new consumer by subscribing to an event |

### Example

When `OrderPlaced` is published, a new Analytics Service can subscribe to it without changing the Order Service.

---

# 9. Challenges & Trade-Offs

EDA also introduces complexity.

### 1. Complex Debugging

A flow can pass through multiple services, making end-to-end tracing harder.

### 2. Event Ordering

Events may need to be processed in the correct sequence.

### 3. Duplicate Events

A broker may deliver an event more than once. Consumers should be idempotent.

### 4. Event Schema Evolution

If an event's structure changes, producers and consumers must remain compatible. Versioning matters.

### 5. Eventual Consistency

Different services may update at different times, so data can be temporarily stale.

### 6. Monitoring

Distributed event flows need appropriate logging, metrics, and tracing.

---

# 10. Advanced EDA Patterns

The notes introduce three patterns built on top of basic EDA:

1. Event Sourcing
2. CQRS
3. Saga Pattern

---

# 11. Event Sourcing

**Event Sourcing** stores changes as a sequence of events instead of storing only the latest state.

### Main Idea

Every change becomes a new event appended to an event log.

```text
Event 1 → Event 2 → Event 3 → Event 4
```

The current state can be rebuilt by replaying the events in order.

### Example

For an account:

```text
AccountCreated
MoneyDeposited
MoneyWithdrawn
MoneyDeposited
```

The event log records what happened over time.

### Important Points

- Every change is a new event.
- Events are appended to the event store.
- The event history is the source of truth.
- State can be rebuilt by replaying events.
- The model described in the notes avoids updating/deleting old events.

### Key Idea

**Event Sourcing = Store the history of changes as events.**

---

# 12. CQRS (Command Query Responsibility Segregation)

**CQRS** separates how data is written (commands) from how data is read (queries).

### Traditional Approach

The same database/table is used for both writing and reading.

```text
Application
     ↓
Same Database
  ↙       ↘
Write     Read
```

At scale, read queries may need different optimization from writes.

### With CQRS

```text
              Application
               /       \
              ↓         ↓
        Command Side  Query Side
              ↓         ↑
          Write DB    Read DB
              ↓         ↑
             Event ────┘
```

### Command Side

A command requests a change.

Example:

```text
POST /place-order
```

It validates the payment, saves the order, and emits `OrderPlaced`.

### Query Side

The query side reads data from a read-optimized model.

Example:

```text
GET /orders
```

It fetches a precomputed order summary from the read database.

### Benefits

- Read and write sides can be optimized separately.
- Read database can use cache, indexes, or NoSQL.
- Write database can remain normalized and transactional.

### Key Idea

**CQRS = Separate write operations from read operations.**

---

# 13. Saga Pattern

The **Saga Pattern** manages a business operation that spans multiple services, each with its own database.

In a monolithic application, several table updates may happen in one database transaction.

In microservices:

- Each service owns its database.
- There is no single global transaction across all services.
- A business operation may involve several local transactions.

### Example: Place an Order

```text
Order Service
      ↓
Payment Service
      ↓
Inventory Service
      ↓
Shipping Service
```

Each service performs its own local transaction.

### What if a Step Fails?

A **compensating transaction** is executed to undo the effect of a previous successful step.

Example:

```text
1. Order created
       ↓
2. Payment completed
       ↓
3. Inventory update fails
       ↓
4. Compensate: refund payment
       ↓
5. Compensate: cancel order
```

### Important

- Saga is a sequence of local transactions.
- Each successful step can trigger the next step.
- If a step fails, compensating actions are performed.
- It supports eventual consistency.
- It does not provide one global ACID transaction or a single rollback button.

### Key Idea

**Saga = Coordinate multiple service transactions using compensating actions.**

---

# 14. EDA vs Request-Driven Architecture

| Aspect | REST / Request-Driven | Event-Driven |
|---|---|---|
| Communication | Direct | Through broker |
| Coupling | Tighter | Looser |
| Flow | Usually synchronous | Usually asynchronous |
| Failure | Can propagate through call chain | Can be isolated by consumers |
| Scaling | Can be harder across dependencies | Consumers can scale independently |

---

# 15. When to Use EDA

EDA is useful for:

- Microservices architecture
- Real-time or reactive systems
- Asynchronous workflows
- High scalability needs
- Decoupling services

Avoid adding EDA overhead to very small CRUD applications when direct request/response communication is sufficient.

---

# 16. Practical Demo — RabbitMQ

Example from the notes:

```text
Order Service
   ↓
OrderPlaced event
   ↓
RabbitMQ Event Broker
   ├──→ Email Service
   ├──→ Inventory Service
   └──→ Analytics Service
```

The Order Service publishes `OrderPlaced` to a fanout exchange.

RabbitMQ broadcasts the event to the bound queues, so each interested service can listen and process it independently.

### Key Idea

**One event can trigger multiple independent consumers.**

---

# 17. Important Terms

| Term | Meaning |
|---|---|
| **EDA** | Architecture where services communicate through events |
| **Event** | Something that happened in a system |
| **Producer / Publisher** | Service that publishes an event |
| **Consumer / Subscriber** | Service that listens and reacts |
| **Event Broker** | Routes and delivers events |
| **Message Bus** | Infrastructure for passing messages/events |
| **Event Notification** | Small event signal; consumer may fetch more data |
| **Event-Carried State Transfer** | Event contains data needed by consumers |
| **Event Sourcing** | Stores changes as an ordered event history |
| **CQRS** | Separates command/write and query/read models |
| **Saga** | Coordinates local transactions across services |
| **Compensating Transaction** | Action that counteracts a previous successful step |
| **Eventual Consistency** | Services may become consistent after some delay |
| **Idempotency** | Reprocessing the same event does not cause an incorrect repeated effect |
| **Schema Evolution** | Managing changes to event data structures |
| **Fanout** | One published event is delivered to multiple subscribers |
