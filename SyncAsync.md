# Synchronous & Asynchronous Communication — Short Notes

## 1. Communication in Modern Architectures

In modern architectures, services constantly communicate with each other.

Examples:
- Payment Service
- Order Service
- Notification Service
- Inventory Service

There are two main ways services communicate:

1. **Synchronous Communication**
2. **Asynchronous Communication**

### Main Difference

**Synchronous → Wait for the response**

**Asynchronous → Don't wait; continue working**

---

# 2. Synchronous Communication

In synchronous communication, the client sends a request and **waits until the server sends a response**.

### Flow

```text
Client → Request → Server
Client ← Response ← Server
```

### Steps

1. Client sends a request.
2. Client waits.
3. Server processes the request.
4. Server sends a response.
5. Client continues after receiving the response.

### Real-Life Example

Imagine ordering food at a restaurant and **waiting there until the food is ready**.

Another example:

```text
Client → Order Request
       ↓
      Wait
       ↓
Order Confirmed
       ↓
    Continue
```

---

# 3. Advantages of Synchronous Communication

- **Simple and predictable** — you get the result immediately.
- **Easy to debug and trace** — request and response are directly connected.
- **Easier to understand and implement**.
- Good when the next operation depends on the current response.

---

# 4. Disadvantages of Synchronous Communication

- **Slower if the server is slow**.
- **Tight coupling** — client and server are directly dependent.
- **One slow service can block the whole chain**.
- **Poor scalability for large systems**.

Example:

```text
Service A → Service B → Service C
                ↓
           Slow response
                ↓
          Service A waits
```

---

# 5. Synchronous Examples

Common examples include:

- **REST APIs** → Client waits for JSON response.
- **gRPC calls** → Client waits for response.
- **Database queries** → Application waits for query result.

Example:

```text
Client
  ↓
GET /users/123
  ↓
Server
  ↓
JSON Response
  ↓
Client continues
```

---

# 6. Asynchronous Communication

In asynchronous communication, the client sends a request/message but **does not wait for the operation to finish**.

### Flow

```text
Client → Request → Server
Client → Continues working
Server → Processes later
Server → Response / Callback / Event
```

### Steps

1. Client sends a request/message.
2. Client continues working.
3. Server processes the request in the background.
4. Result can be received later through a response, callback, event, or message.

---

# 7. Real-Life Example of Asynchronous Communication

Imagine ordering food through an online application.

```text
Place Order
     ↓
Order Confirmed
     ↓
You continue doing other work
     ↓
Food preparation happens
     ↓
Notification arrives later
```

You do not have to continuously wait for the food to be prepared.

---

# 8. Advantages of Asynchronous Communication

- **Better for non-blocking work**.
- Services can continue working independently.
- **Improves scalability** for large workloads.
- Useful when tasks take a long time.
- Useful for **background processing**.
- Helps handle large traffic bursts.

---

# 9. Disadvantages of Asynchronous Communication

- **Harder to debug** because processing happens later.
- **More complex** than synchronous communication.
- Requires mechanisms such as:
  - Message queues
  - Events
  - Callbacks
  - Background workers
- Results may arrive later.
- Systems can become **eventually consistent**, rather than immediately consistent.

---

# 10. Asynchronous Communication Examples

### Message Queues

Examples:
- RabbitMQ
- Kafka
- SQS

```text
Service A
   ↓
Message Queue
   ↓
Service B
```

Service A does not need to wait for Service B to finish immediately.

### Background Jobs

```text
Client
  ↓
Job Queue
  ↓
Worker
  ↓
Process Task
```

### Push Notifications

```text
Payment Completed
       ↓
Notification Event
       ↓
Push Notification
```

---

# 11. Synchronous vs Asynchronous

| Feature | Synchronous | Asynchronous |
|---|---|---|
| Waiting | Client waits | Client does not wait |
| Communication | Request → Response | Message/Event → Later processing |
| Complexity | Simpler | More complex |
| Debugging | Easier | Harder |
| Coupling | More tightly coupled | More loosely coupled |
| Long tasks | Less suitable | Well suited |
| Traffic bursts | Less suitable | Better suited |
| Background jobs | Not ideal | Excellent |
| Response | Usually immediate | Can arrive later |

---

# 12. When to Use Synchronous Communication

Use synchronous communication when:

- You **need an immediate response**.
- The next action depends on the current response.
- The operation is small.
- Low latency is required.

### Examples

```text
REST API → Client waits for JSON response
gRPC call → Client waits for response
Database query → Application waits for result
```

### Example Scenario

```text
Client
  ↓
Check Account Balance
  ↓
Wait for Result
  ↓
Display Balance
```

The client needs the result immediately, so synchronous communication fits.

---

# 13. When to Use Asynchronous Communication

Use asynchronous communication when:

- You have **background or delayed tasks**.
- The client does not need an immediate result.
- You need to handle large traffic bursts.
- You want services to work independently.
- The task may take a long time.

### Examples

```text
Order Placed
    ↓
Event
    ↓
Email Service
```

or:

```text
Upload File
    ↓
Background Job
    ↓
Process File
    ↓
Notification
```

---

# 14. Simple Decision Rule

### Use Synchronous When:

**"I need the answer now."**

```text
Request → Wait → Response
```

### Use Asynchronous When:

**"Start the work now; I can get the result later."**

```text
Request → Continue
             ↓
        Result later
```

---

# 15. Important Terms

| Term | Meaning |
|---|---|
| **Synchronous** | Wait for the response before continuing |
| **Asynchronous** | Continue without waiting for completion |
| **Blocking** | Caller waits for an operation to finish |
| **Non-blocking** | Caller can continue while work happens |
| **Message Queue** | Stores messages for asynchronous processing |
| **Event** | Notification that something happened |
| **Background Job** | Task processed separately from the main request |
| **Callback** | Mechanism for receiving a result later |
| **RabbitMQ** | Message broker used for asynchronous communication |
| **Kafka** | Distributed event/message streaming platform |
| **SQS** | Managed message queue service |
| **Eventual Consistency** | Data may become consistent after some delay |
