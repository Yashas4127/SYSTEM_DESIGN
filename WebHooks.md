# Webhooks — Short Notes

## 1. What is a Webhook?

A **Webhook** is an **HTTP-based callback** used by one system to notify another system when an event happens.

### Simple Meaning

Instead of continuously asking:

```text
"Did something happen?"
```

the receiving system gives the sender a URL:

```text
"When something happens, call this URL."
```

### Basic Flow

```text
Event Happens
     ↓
Webhook Sender
     ↓
HTTP POST
     ↓
Receiver Endpoint
```

### Key Idea

**Webhook = HTTP-based callback**

Webhooks are usually sent as **HTTP POST requests**.

---

# 2. Callback Concept

A callback is a pattern where one system says:

```text
"When you finish, call me back."
```

It is an **asynchronous communication mechanism**.

### Simple Example

Imagine ordering food:

```text
You → Restaurant → Place Order
          ↓
       You continue
          ↓
Restaurant → Calls you when food is ready
```

You don't wait on the original call.

---

# 3. Webhook vs Polling

### Polling

The client repeatedly asks:

```text
"Is the payment complete?"
       ↓
"Is the payment complete?"
       ↓
"Is the payment complete?"
```

This creates unnecessary requests.

### Webhook

The provider automatically notifies the client:

```text
Payment Complete
      ↓
Webhook
      ↓
Your Server
```

### Key Difference

**Polling → Client keeps asking**

**Webhook → Server/provider tells client when event happens**

---

# 4. Webhook Example — Payment

Suppose your application uses a payment gateway.

### Step 1: Send Payment Request

```text
POST /api/payments
```

Example:

```json
{
  "amount": 500,
  "callback_url": "https://yourapp.com/payment-status"
}
```

### Step 2: Payment Provider Processes Payment

The provider processes the payment asynchronously.

### Step 3: Provider Sends Webhook

When payment is complete:

```text
POST https://yourapp.com/payment-status
```

Example payload:

```json
{
  "orderId": "123",
  "status": "SUCCESS"
}
```

### Important

You:

- Did not continuously poll.
- Did not wait synchronously.
- Received a notification when the event happened.

---

# 5. Core Components of a Webhook System

| Component | Meaning |
|---|---|
| **Publisher / Event Source** | System that triggers the webhook |
| **Subscriber / Receiver** | System that receives and handles the webhook |
| **Event** | Something that happened |
| **Endpoint / Callback URL** | Public HTTP endpoint receiving the webhook |
| **Payload** | Data describing the event |
| **Delivery Attempt** | One HTTP request made to the receiver |
| **Retry Mechanism** | Re-attempt after a failed delivery |

### Example

```text
Stripe
  ↓
Payment Success Event
  ↓
Webhook Endpoint
  ↓
Your Backend
```

---

# 6. Real-World Webhook Examples

| Platform | Event | Purpose |
|---|---|---|
| **GitHub** | New commit pushed | Notify another system |
| **Stripe / Razorpay** | Payment success | Update order/payment status |
| **Slack** | Message event | Notify another service/bot |
| **Shopify** | Order created | Notify ERP/warehouse |

### Key Idea

Webhooks are useful when one system needs to **automatically notify another system about events**.

---

# 7. Webhook Lifecycle

A typical webhook lifecycle:

```text
1. Register Webhook Endpoint
          ↓
2. Event Happens
          ↓
3. Webhook Sent
          ↓
4. Receiver Acknowledges
          ↓
5. Process Payload
          ↓
6. Retry if Delivery Failed
```

---

# 8. Step 1 — Register Webhook Endpoint

The subscriber provides a callback URL to the external service.

Example:

```text
https://yourapp.com/webhooks/payment
```

The external system stores this endpoint.

---

# 9. Step 2 — Event Happens

Something triggers the webhook.

Examples:

```text
Payment completed
New commit pushed
Order created
Message received
```

---

# 10. Step 3 — Webhook is Sent

The publisher sends an HTTP POST request.

```text
External Service
       ↓
HTTP POST
       ↓
/webhooks/payment
```

The request usually contains a JSON payload.

---

# 11. Step 4 — Acknowledge Delivery

The receiver should respond quickly, commonly with:

```text
HTTP 200 OK
```

This tells the sender:

```text
"Webhook received successfully."
```

### Important

Do not perform heavy business logic before sending the acknowledgment if the provider has a short timeout.

---

# 12. Step 5 — Process Payload

After receiving the webhook, your application can:

- Update the database
- Update order status
- Send notifications
- Start background jobs
- Trigger another service

Example:

```text
Webhook
   ↓
Update Payment Status
   ↓
Update Order
   ↓
Send Notification
```

---

# 13. Advantages of Webhooks

### No Continuous Polling

You don't need to repeatedly ask whether something happened.

### Doesn't Block the Flow

The sender can perform work asynchronously.

### Event-Based

The webhook is sent only when the relevant event occurs.

### Simple Integration

Webhooks use standard HTTP, making them easy to integrate.

### Efficient

No unnecessary repeated requests are required.

---

# 14. Webhook Challenges

Webhooks are useful, but production systems need to handle several problems.

### Main Challenges

- Unreliable delivery
- Duplicate deliveries
- Ordering issues
- Security risks
- Receiver downtime
- Timeouts

---

# 15. Unreliable Delivery

The receiver may be:

- Down
- Overloaded
- Temporarily unreachable

The webhook may fail.

### Solution

Use:

```text
Retry
   ↓
Exponential Backoff
```

The sender should retry failed deliveries.

---

# 16. Duplicate Deliveries

A webhook provider may send the same event more than once.

Example:

```text
Webhook Event 123
      ↓
Receiver
      ↓
Timeout
      ↓
Provider retries
      ↓
Event 123 again
```

Now the receiver gets the same event twice.

### Solution

Use **Idempotency**.

The system should safely process the same event multiple times without producing an incorrect final result.

---

# 17. Idempotency

**Idempotency** means processing the same event multiple times produces the same final result.

A common approach is to store an:

```text
event_id
```

Example:

```text
Event ID = 12345
```

First delivery:

```text
12345 → Process → Store event ID
```

Duplicate delivery:

```text
12345 → Already processed → Ignore
```

### Key Idea

**Idempotency is critical for reliable webhooks.**

---

# 18. Ordering Issues

Events may arrive out of order.

Example:

```text
Event 2 arrives
      ↓
Event 1 arrives later
```

This can cause incorrect state if the application assumes strict ordering.

### Possible Solutions

Use:

- Event IDs
- Timestamps
- Sequence numbers
- Version numbers

The receiver can use these values to determine the correct order.

---

# 19. Security Risks

Anyone who knows the webhook URL may try to send fake requests.

Example:

```text
Attacker
   ↓
Fake HTTP POST
   ↓
Webhook Endpoint
```

Therefore, webhook authenticity must be verified.

---

# 20. Securing Webhooks — HMAC

A common approach is **HMAC signature verification**.

### Flow

```text
Publisher
   ↓
Sign Payload + Shared Secret
   ↓
Webhook + Signature
   ↓
Receiver
   ↓
Recalculate Signature
   ↓
Compare
```

If signatures match:

```text
Request is trusted
```

If they don't:

```text
Reject request
```

Some services provide a signature in an HTTP header.

---

# 21. HTTPS

Webhooks should use:

```text
HTTPS
```

instead of plain HTTP.

Example:

```text
https://yourapp.com/webhook
```

HTTPS protects data while it travels over the network.

---

# 22. IP Whitelisting

The receiver can restrict webhook requests to trusted IP ranges.

```text
Trusted IP → Allow
Unknown IP → Reject
```

This can add another security layer.

However, IP allowlists should be maintained carefully because provider IP ranges can change.

---

# 23. Acknowledge Fast

A webhook endpoint should respond quickly.

Recommended pattern:

```text
Webhook Received
      ↓
Validate
      ↓
Push to Queue
      ↓
HTTP 200 OK
```

Then:

```text
Queue
  ↓
Worker
  ↓
Business Logic
```

### Key Idea

**ACK first, process asynchronously.**

---

# 24. Retry Mechanism

If webhook delivery fails:

```text
Webhook
   ↓
Delivery Failed
   ↓
Retry
   ↓
Failed?
   ↓
Retry Again
```

A common strategy is **exponential backoff**.

Example:

```text
1 sec
2 sec
4 sec
8 sec
...
```

This prevents repeatedly hitting an unavailable receiver.

---

# 25. Dead Letter / Failed Webhooks

If a webhook keeps failing after several retries, it can be moved to a separate failed-message queue.

```text
Webhook
   ↓
Retry
   ↓
Retry
   ↓
Retry limit reached
   ↓
Dead Letter Queue
```

This allows developers to inspect and recover failed events later.

---

# 26. Reliable Webhook Architecture

In production, avoid putting heavy business logic directly inside the webhook endpoint.

Use:

```text
External Service
       ↓
Webhook Endpoint
       ↓
Message Queue
       ↓
Worker Service
       ↓
Process Event
       ↓
Update Database
```

### Webhook Endpoint

Its main job is:

```text
Receive
Validate
Queue
ACK
```

### Worker

Its job is:

```text
Read Event
Process Business Logic
Update DB
```

---

# 27. Why Use a Queue?

A queue improves reliability.

If the worker is temporarily unavailable:

```text
Webhook Endpoint
       ↓
Message Queue
       ↓
Worker unavailable
```

The message can remain in the queue until the worker is ready.

This prevents the webhook endpoint from depending directly on the worker's availability.

---

# 28. Webhook Architecture — Complete Flow

```text
External Service
       ↓
Webhook POST
       ↓
Webhook Endpoint
       ↓
Validate Signature
       ↓
Message Queue
       ↓
HTTP 200 OK
       ↓
Worker
       ↓
Process Event
       ↓
Update Database
       ↓
Send Notifications
```

Retries, failed-message handling, and monitoring can be managed separately.

---

# 29. Webhook vs API Request

| Feature | Normal API Request | Webhook |
|---|---|---|
| Who starts communication? | Client | Event source |
| Main purpose | Request data/action | Notify about an event |
| Communication | Request → Response | Event → Callback |
| Timing | Usually immediate | Usually asynchronous |
| Polling needed | Sometimes | No |
| Example | Get payment status | Payment completed notification |

---

# 30. Webhook vs Polling

| Feature | Polling | Webhook |
|---|---|---|
| Client asks repeatedly | Yes | No |
| Event notification | Delayed by polling interval | Near event time |
| Unnecessary requests | More | Fewer |
| Implementation | Simple | Requires endpoint |
| Server pushes event | No | Yes |
| Best for | When webhook is unavailable | Event-driven integrations |

---

# 31. Practical System Design Example

Suppose:

```text
Service A = Your Application
Service B = Payment Provider
```

### Flow

```text
Service A
    ↓
Payment API
    ↓
Service B
    ↓
Payment Processing
    ↓
Webhook
    ↓
Service A Webhook Endpoint
    ↓
Queue
    ↓
Worker
    ↓
Update Database
```

The payment provider performs the asynchronous work and later calls Service A through the webhook.

---

# 32. Important Terms

| Term | Meaning |
|---|---|
| **Webhook** | HTTP-based callback used for event notification |
| **Callback** | Call made later when an operation/event completes |
| **Publisher** | System that sends the webhook |
| **Subscriber** | System that receives the webhook |
| **Event** | Something that happened |
| **Endpoint** | URL that receives the webhook |
| **Payload** | Data sent in the webhook |
| **Delivery Attempt** | One attempt to deliver a webhook |
| **ACK** | Successful acknowledgment, commonly HTTP 200 |
| **Retry** | Re-attempt after delivery failure |
| **Idempotency** | Safe handling of duplicate events |
| **HMAC** | Signature mechanism used to verify authenticity |
| **Exponential Backoff** | Increasing delay between retries |
| **Dead Letter Queue** | Stores repeatedly failed events |
| **Message Queue** | Temporarily stores events for asynchronous processing |
