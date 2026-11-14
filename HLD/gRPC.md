# gRPC — Short Notes

## 1. What is gRPC?

**gRPC = Google Remote Procedure Call**

gRPC is a modern **RPC framework developed by Google**.

It allows one service/program to call a function on another computer or remote server as if it were calling a local function.

### Basic Idea

`Your Code → Remote Server`

Example:

```text
call getBalance()
        ↓
    Remote Server
```

Your code calls `getBalance()`, but the actual work is performed by a remote server.

---

# 2. What is RPC?

**RPC = Remote Procedure Call**

RPC allows one program to call a **function/procedure on another computer**.

The goal is to hide network complexity such as:
- Sockets
- IP addresses
- Serialization

### Real-Life Example

Imagine a banking application.

Your frontend calls:

```text
getBalance(accountId)
```

Instead of calculating the balance locally:
1. The call goes to the bank's backend server.
2. The server executes the logic.
3. The server gets the result from the database.
4. The result is returned to your application.

For the caller, it feels similar to calling a normal function.

---

# 3. Older RPC Systems

Some older RPC technologies include:

- Java RMI
- XML-RPC
- SOAP
- Thrift

The notes highlight problems such as:
- Heavy
- Slow
- Complex

**gRPC was introduced as a modern RPC approach.**

---

# 4. gRPC — Main Technologies

gRPC uses:

### HTTP/2

Used as the transport layer.

It provides features such as:
- Fast communication
- Multiplexing
- Bidirectional communication

### Protocol Buffers (Protobuf)

Used for **data serialization**.

Protobuf is:
- Lightweight
- Binary
- Compact
- Faster than text-based JSON according to the notes

### Simple Architecture

```text
Client Service
      ↓
    gRPC
      ↓
HTTP/2
      ↓
Server Service
```

---

# 5. Why gRPC is Fast

The notes associate gRPC's performance with:

- **HTTP/2**
- **Binary Protobuf serialization**
- **Multiplexing**
- **Streaming support**

Compared with text-based JSON, Protobuf sends compact binary data.

### Simple Idea

**REST → JSON → Text-based**

**gRPC → Protobuf → Binary**

---

# 6. Protobuf

**Protobuf = Protocol Buffers**

It is a **binary data format developed by Google**.

It is similar to JSON or XML for representing data, but the data is:
- Smaller
- Binary
- Faster to serialize
- Strongly typed

### Example

A JSON request might look like:

```json
{
  "item": "Pizza",
  "quantity": 2
}
```

A Protobuf request is represented in a compact binary form.

It may look unreadable to humans, but computers can process it efficiently.

---

# 7. .proto File

A **`.proto` file** acts like a **contract or API specification**.

It defines:

- **Messages** → Data structures
- **Services** → RPC functions/APIs

### Example

```proto
syntax = "proto3";

package food;

service OrderService {
  rpc PlaceOrder(OrderRequest) returns (OrderResponse);
  rpc TrackDelivery(DeliveryRequest) returns (stream DeliveryLocation);
}

message OrderRequest {
  string item = 1;
  int32 quantity = 2;
}

message OrderResponse {
  string status = 1;
}

message DeliveryRequest {
  string orderId = 1;
}

message DeliveryLocation {
  double latitude = 1;
  double longitude = 2;
}
```

### Key Idea

The `.proto` file defines the **contract between client and server**.

---

# 8. Strongly Typed Contracts

gRPC uses strongly typed contracts.

The API is defined in the `.proto` file.

The `.proto` definition specifies:
- Data types
- Messages
- RPC methods
- Request types
- Response types

gRPC can then **auto-generate client and server code** from the contract.

### Benefit

Client and server follow the same API definition.

---

# 9. Cross-Language Support

gRPC supports communication between services written in different languages.

Example:

```text
Client → Python
Server → Java
Another Service → Go
```

These services can communicate using the same gRPC contract.

### Key Idea

**Different languages can communicate using the same `.proto` definition.**

---

# 10. Streaming Support

gRPC supports different communication patterns.

## 1. Unary RPC

**One request → One response**

Similar to a normal REST request.

```text
Client → Request → Server
Client ← Response ← Server
```

---

## 2. Server Streaming

**One request → Multiple responses**

The client sends one request and the server sends multiple responses.

Useful for:
- Live updates
- Continuous data

```text
Client → Request
Server → Response 1
Server → Response 2
Server → Response 3
```

---

## 3. Client Streaming

**Multiple requests → One response**

The client sends multiple messages and the server sends one final response.

```text
Client → Request 1
Client → Request 2
Client → Request 3
              ↓
        Server → Response
```

---

## 4. Bidirectional Streaming

**Both client and server can send multiple messages.**

Communication can happen in both directions in real time.

Example:

```text
Client ↔ Server
  ↕        ↕
Multiple messages
```

The notes use **chat** as an example.

### Easy Memory Trick

- **Unary** → 1 → 1
- **Server streaming** → 1 → many
- **Client streaming** → many → 1
- **Bidirectional** → many ↔ many

---

# 11. Where is gRPC Used?

The notes mention gRPC in systems where **performance matters**, including:

- Google
- Netflix
- Uber
- AWS
- IoT
- Gaming
- Finance

It is especially useful for communication between services.

---

# 12. gRPC in Microservices

Imagine a system like Netflix with separate services:

```text
Auth Service
     ↕
Video Streaming Service
     ↕
Billing Service
     ↕
Recommendation Service
```

These independent services need to communicate.

gRPC can provide fast service-to-service communication using:
- HTTP/2
- Protobuf
- Strong contracts
- Streaming

---

# 13. gRPC vs REST

| Feature | REST | gRPC |
|---|---|---|
| Data format | JSON (text) | Protobuf (binary) |
| Speed | Medium | High |
| Communication | Request → Response | Unary + Streaming |
| Contract | Optional (e.g. Swagger) | Strong `.proto` contract |
| Best for | Public APIs, easy use | Microservices & real-time apps |

---

# 14. gRPC vs GraphQL vs REST

| Feature | REST | GraphQL | gRPC |
|---|---|---|---|
| Data format | JSON (text, verbose) | JSON (client flexible) | Protobuf (binary, compact) |
| Speed | Medium | Medium | High |
| Communication | Request → Response | Request → Response | Unary + Streaming |
| Contract | Optional (Swagger) | Strong schema | Strong `.proto` contract |
| Best for | Public APIs | Frontend flexibility | Microservices & real-time apps |

---

# 15. When gRPC is Useful

gRPC is useful when you need:

- High-performance service-to-service communication
- Microservices communication
- Streaming
- Strong API contracts
- Cross-language communication
- Compact binary data
- Real-time communication patterns

---

# 16. Important Terms

| Term | Meaning |
|---|---|
| **gRPC** | Modern RPC framework developed by Google |
| **RPC** | Calling a function on a remote computer |
| **HTTP/2** | Transport used by gRPC |
| **Protobuf** | Compact binary serialization format |
| **.proto** | API/service contract file |
| **Message** | Data structure defined in `.proto` |
| **Service** | Collection of RPC methods |
| **Unary RPC** | One request → one response |
| **Server Streaming** | One request → multiple responses |
| **Client Streaming** | Multiple requests → one response |
| **Bidirectional Streaming** | Multiple messages in both directions |
| **Strong Contract** | Client and server follow a defined `.proto` schema |
| **Cross-language** | Different programming languages can communicate |

