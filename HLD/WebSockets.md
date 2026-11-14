# WebSockets — Short Notes

## 1. Why Were WebSockets Introduced?

The original HTTP model follows a **request → response** pattern.

```text
Client  ── Request ──→ Server
Client  ←─ Response ── Server
```

Basic HTTP flow:
1. Client sends a request.
2. Server sends a response.
3. In HTTP/1.1, the connection is commonly closed or reused for another request; it is not a server-push channel by itself.

### Main Problem

What if the server has new data, but the client did not ask for it?

**Normal HTTP request/response does not let the server independently push an update to the client.**

---

# 2. Applications That Need Real-Time Communication

Examples:
- Chat applications — WhatsApp, Slack
- Stock price updates
- Notifications
- Multiplayer games
- Live dashboards
- Online collaboration — Google Docs

HTTP polling can support updates, but it may be inefficient for continuous real-time communication.

---

# 3. Approaches Used Before WebSockets

The notes cover three approaches:

1. Polling
2. Long Polling
3. Server-Sent Events (SSE)

---

# 4. Polling

In **Polling**, the client repeatedly sends requests to the server after a fixed time interval.

```text
Client                 Server
   |---- Any update? ---->|
   |<------- No ----------|
   |   Request ends       |
   |                      |
   |---- Any update? ---->|
   |<------- No ----------|
   |                      |
   |---- Any update? ---->|
   |<------ Yes, data ----|
```

Example: the client asks every 5 seconds:

```text
"Any update?"
```

### Problems
- Many unnecessary requests
- Updates may be delayed until the next poll
- Wasted CPU, bandwidth, and money

Example from the notes:

```text
10 million users × poll every 5 seconds
= 120 million requests/minute
```

---

# 5. Long Polling

**Long Polling** improves on normal polling.

Instead of responding immediately when no new data exists, the server waits until data is available or a timeout occurs.

### Flow

```text
Client                 Server
   |---- Any update? ---->|
   |                      | Waiting...
   |<------ Data ---------|
   |                      |
   |---- Any update? ---->|
   |                      | Waiting...
   |<------ Timeout ------|
```

### Steps
1. Client sends a request.
2. Server waits until data is available.
3. Server responds when data arrives or the request times out.
4. Client immediately sends another request.

### Problems
- HTTP overhead remains: headers are sent repeatedly.
- Connections/requests are continually created and completed.
- Hard to scale for many clients.

---

# 6. Server-Sent Events (SSE)

**Server-Sent Events (SSE)** allow the server to push updates to the client over one HTTP connection.

```text
Client                 Server
   |------ Connection -->|
   |<------- Event ------|
   |<------- Event ------|
   |<------- Event ------|
   |<------- Event ------|
```

### Limitations Mentioned in the Notes
- One-way communication only: server → client
- No binary data
- Not suitable for some two-way applications such as chat and games

**SSE is useful when the server sends updates to the client, but two-way communication is not required.**

---

# 7. Why WebSockets Were Invented

The goal was a protocol that:
- Keeps one persistent connection
- Allows full-duplex communication
- Has low overhead
- Works well with browsers
- Works with existing HTTP infrastructure

**Full-duplex** means both client and server can send data at any time.

---

# 8. What Exactly Is a WebSocket?

A **WebSocket** is a persistent, full-duplex communication protocol over a single TCP connection.

### Key Properties
- One persistent connection
- Client and server can both send data at any time
- Very low latency
- Minimal overhead after the connection is established

```text
Client                 Server
   |--- Initiate ------->|
   |<-- Connection ready-|
   |<==== Data flow ====>|
   |--- Close ---------->|
```

---

# 9. WebSocket vs HTTP

| Feature | Normal HTTP | WebSocket |
|---|---|---|
| Communication | Request → Response | Full-duplex |
| Connection | Request/response based | Persistent |
| Server push | Not directly in normal request/response | Yes |
| Client can send anytime | Sends requests | Yes |
| Overhead | Repeated request/response overhead | Lower after handshake |
| Common use | CRUD APIs, documents | Chat, live updates, games |

---

# 10. How a WebSocket Connection Is Established

The connection begins with a TCP connection and an HTTP **Upgrade handshake**.

## Step 0 — TCP Connection
- Client establishes a TCP connection to the server.
- `ws://` normally uses port 80.
- `wss://` normally uses port 443 and TLS.

This is normal TCP/HTTPS connection setup.

## Step 1 — Client Sends an HTTP Upgrade Request

The client starts the WebSocket handshake with a normal HTTP request containing special headers.

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: random_key
Sec-WebSocket-Version: 13
```

### Meaning of the Headers

| Header | Meaning |
|---|---|
| `Upgrade: websocket` | Requests a switch to the WebSocket protocol |
| `Connection: Upgrade` | Indicates that the connection should be upgraded |
| `Sec-WebSocket-Key` | Random value used in the handshake |
| `Sec-WebSocket-Version: 13` | WebSocket protocol version |

**At this point, communication is still HTTP. WebSocket frames have not started yet.**

## Step 2 — Server Validates the Request

The server checks:
- Is the Upgrade header present?
- Is WebSocket version 13 supported?
- Is this endpoint allowed to upgrade?

If a check fails, the server responds with a normal HTTP error.

## Step 3 — Server Sends HTTP 101

If the request is accepted, the server responds:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: hashed_key
```

`101 Switching Protocols` means the protocol switch has been accepted.

## Step 4 — Protocol Switches to WebSocket

After the `101` response:
- The WebSocket protocol starts.
- The HTTP request/response model is no longer used for application messages.
- Communication uses WebSocket frames.

```text
Client                         Server
  |--- HTTP Upgrade Request --->|
  |<-- HTTP 101 Switching -------|
  |       Protocols              |
  |<==== WebSocket Frames ======>|
```

---

# 11. WebSocket Communication

After the handshake, both sides can send different types of frames.

The notes list:
- **Text frames**
- **Binary frames**
- **Ping frames**
- **Pong frames**
- **Close frames**

### Full-Duplex Communication

```text
Client  <==== messages ====>  Server
```

The server can send data without waiting for a client request, and the client can also send data at any time.

---

# 12. How a WebSocket Connection Closes

A WebSocket connection closes through a graceful handshake using **Close frames**, rather than simply terminating TCP immediately.

## Step 0 — Connection Is Active
- TCP connection is open.
- WebSocket frames may be flowing.
- Ping/Pong may be happening.

## Step 1 — One Side Decides to Close

Either side can start closing.

Examples:
- Client: browser refresh or tab close
- Server: shutdown, idle timeout, or deployment

## Step 2 — Initiator Sends a Close Frame

The initiator sends a WebSocket Close frame.

It can contain:
- Close code (optional, but recommended)
- Reason string (optional)

Example:

```text
Opcode: CLOSE
Code: 1000
Reason: Normal closure
```

After sending the Close frame, the initiator does not send more normal data frames.

## Step 3 — Receiver Gets the Close Frame

The other side must:
1. Stop sending normal data.
2. Send its own Close frame back.

This completes the graceful shutdown handshake.

## Step 4 — TCP Connection Closes

After both sides exchange Close frames:
- TCP connection is closed.
- File descriptors are released.
- Memory is cleaned up.
- WebSocket is fully closed.

```text
Client  ---- CLOSE ----> Server
Client  <--- CLOSE ----- Server
          TCP closes
```

---

# 13. Scaling WebSockets

WebSocket connections are **stateful**: a connection stays associated with a server.

### Problem

Suppose a load balancer sends users to different servers:

```text
User A → Server A
User B → Server B
```

A WebSocket connection is tied to the server that accepted it. The notes mention that the load balancer may need **sticky sessions** so a client's connection continues to use the same server.

### Scaling Solutions
1. **Sticky load balancing** — keep a client connected to the same server.
2. **Shared Pub/Sub** — use Redis, Kafka, etc. to distribute messages between servers.
3. **Connection registries** — track which server owns each connection.
4. **Horizontal scaling with event fan-out** — distribute an event to servers that need to deliver it to connected clients.

```text
User A ── WSS ── Server A
                    \
                     Message Broker
                    /
User B ── WSS ── Server B
```

A shared broker helps servers exchange messages even when users are connected to different servers.

---

# 14. Failure Handling in WebSockets

Connections can fail.

### Common Failures
- Network drops → connection is lost.
- Server crashes → clients disconnect.

### Strategies Mentioned in the Notes
- **Heartbeats (Ping/Pong)** — detect whether a connection is still alive.
- **Automatic reconnect on client** — reconnect after disconnection.
- **Resume sessions** — continue a previous session where supported.
- **Message acknowledgements (application-level)** — track whether important messages were processed.

**WebSocket itself does not guarantee message delivery.**

If reliable delivery is required, the application must implement suitable acknowledgements and recovery logic.

---

# 15. When Should You Use WebSockets?

Use WebSockets when continuous, two-way real-time communication is required.

Examples:
- Chat systems
- Live notifications
- Real-time dashboards
- Online games
- Collaborative editors
- Live tracking applications

---

# 16. When Should You NOT Use WebSockets?

WebSockets may be unnecessary for:
- Simple CRUD APIs
- Rare updates
- Stateless REST services
- Applications where SSE is enough for one-way server-to-client updates

Use a simpler communication method when real-time, two-way communication is not needed.

---

# 17. Polling vs Long Polling vs SSE vs WebSockets

| Feature | Polling | Long Polling | SSE | WebSockets |
|---|---|---|---|---|
| Connection behavior | Repeated requests | Server holds request until data/timeout | Persistent HTTP connection | Persistent TCP connection |
| Server → client | In response to poll | When data becomes available | Yes | Yes |
| Client → server | Through requests | Through requests | Not over the SSE stream | Yes |
| Two-way on same connection | No | No | No | Yes |
| Main limitation | Many requests and delay | Repeated HTTP overhead | One-way | More connection/scaling management |

---

# 18. Important Terms

| Term | Meaning |
|---|---|
| **WebSocket** | Persistent, full-duplex protocol over TCP |
| **HTTP** | Request/response protocol |
| **Polling** | Client repeatedly asks for updates |
| **Long Polling** | Server holds a request until data or timeout |
| **SSE** | Server pushes events to client over HTTP |
| **Full-Duplex** | Both sides can send data at any time |
| **Handshake** | Initial process that upgrades HTTP to WebSocket |
| **Upgrade** | HTTP header requesting a protocol switch |
| **HTTP 101** | Response indicating protocol switching was accepted |
| **WebSocket Frame** | Unit of WebSocket communication |
| **Text Frame** | Carries text data |
| **Binary Frame** | Carries binary data |
| **Ping/Pong** | Control frames used for connection health checks |
| **Close Frame** | Starts or acknowledges graceful WebSocket closure |
| **`ws://`** | WebSocket connection without TLS |
| **`wss://`** | WebSocket connection protected by TLS |
| **Sticky Session** | Keeps a client associated with the same server |
| **Pub/Sub** | Messaging pattern used to distribute events to subscribers |
| **Heartbeat** | Periodic signal used to check whether a connection is alive |
