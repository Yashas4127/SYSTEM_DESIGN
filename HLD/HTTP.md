# HTTP — Short Notes

## 1. HTTP Overview

**HTTP = Hypertext Transfer Protocol**

HTTP is the protocol that powers communication across the web.

A simple way to understand HTTP versions:

- **HTTP/1.0** → 1 truck per item
- **HTTP/1.1** → 1 truck, sequential
- **HTTP/2** → 1 truck, parallel
- **HTTP/3** → independent drones

Each newer version improves how multiple resources are transferred between client and server.

---

# 2. HTTP/1.0 (1996)

HTTP/1.0 is a **text-based protocol**.

### Main Features

- Headers and body are sent as plain text.
- Each request creates a **new TCP connection**.
- No persistent connections.
- Every resource such as an image, CSS file, or JavaScript file may require a new connection.

### Problem

Creating a new TCP connection for every resource is inefficient.

### Real-Life Analogy

Imagine asking a friend one question, ending the conversation, and then calling again for every next question.

### Use Case

- Early static websites
- Mostly HTML with a few images

---

# 3. HTTP/1.1 (1997)

HTTP/1.1 improved HTTP/1.0 by introducing **persistent connections**.

### Main Features

- **Persistent connections** using `Connection: keep-alive`
- Multiple requests can use the **same TCP connection**.
- Better efficiency than HTTP/1.0.

### Problem: Head-of-Line (HOL) Blocking

Requests are handled sequentially.

If one request is delayed, requests behind it may have to wait.

### Real-Life Analogy

Keep one phone call open and ask multiple questions. If your friend delays one answer, the other questions have to wait.

### Use Case

- Dynamic websites
- Websites with multiple resources
- Example mentioned in the notes: product pages and blogs

---

# 4. HTTP/1.0 vs HTTP/1.1

| Feature | HTTP/1.0 | HTTP/1.1 |
|---|---|---|
| Connection | New connection per request | Persistent connection |
| TCP reuse | No | Yes |
| Multiple requests | Less efficient | Same TCP connection |
| Main problem | Connection overhead | HOL blocking |
| Performance | Slower | Better than 1.0 |

---

# 5. HTTP/2 (2015)

HTTP/2 was a **major upgrade**.

### Main Features

## A. Binary Protocol

HTTP/2 uses a **binary protocol** instead of the text-based approach of HTTP/1.x.

## B. Multiplexing

Multiple requests and responses can be sent **simultaneously over one TCP connection**.

```text
Request A ─┐
Request B ─┼→ One TCP Connection
Request C ─┘
```

## C. Header Compression

HTTP/2 reduces repeated header data using:
- Header tables
- Huffman encoding

## D. Server Push

The server can send resources to the client **before the client explicitly requests them**.

## E. Remaining Problem

HTTP/2 still uses **TCP**.

If a TCP packet is lost, affected streams can pause until retransmission. Therefore, TCP-level head-of-line blocking can still occur.

### Use Case

Modern websites with many:
- Images
- Scripts
- Styles
- Other resources

---

# 6. HTTP/1.1 vs HTTP/2

| Feature | HTTP/1.1 | HTTP/2 |
|---|---|---|
| Format | Text-based | Binary |
| Connection | Persistent TCP | Persistent TCP |
| Multiplexing | Limited | Yes |
| Header compression | Limited | Yes |
| Server Push | No | Yes |
| HOL blocking | Request-level | TCP-level still possible |

---

# 7. HTTP/3 (2020s — QUIC/UDP)

HTTP/3 is based on **QUIC over UDP**.

It is designed to avoid TCP's head-of-line blocking problems.

### Main Features

## A. QUIC over UDP

HTTP/3 uses **QUIC**, which runs over UDP instead of TCP.

## B. Independent Streams

Different streams are independent.

If one packet is lost, it does not necessarily block the other streams.

```text
Stream 1 → Request A
Stream 2 → Request B
Stream 3 → Request C
```

## C. Built-in TLS 1.3

TLS 1.3 is built into QUIC.

## D. 0-RTT Connection Setup

HTTP/3 can use **0-RTT** to resume sessions quickly.

## E. Good for Changing Networks

The notes highlight mobile networks where:
- Packet loss can occur.
- Devices may switch between Wi-Fi and mobile data.

### Use Cases

- Video streaming
- YouTube
- Netflix
- Online gaming
- Video conferencing
- Mobile browsing

---

# 8. HTTP/2 vs HTTP/3

| Feature | HTTP/2 | HTTP/3 |
|---|---|---|
| Transport | TCP | QUIC over UDP |
| Multiplexing | Yes | Yes |
| Stream independence | Limited by TCP | Yes |
| TCP HOL blocking | Possible | Avoided |
| Encryption | TLS | TLS 1.3 built into QUIC |
| Fast connection setup | Normal | Supports 0-RTT |
| Mobile network changes | Less optimized | Better suited |

---

# 9. HTTP Evolution

## HTTP/1.0

`New TCP connection → Request → Response → Close`

**Main problem:** Connection overhead.

## HTTP/1.1

`One TCP connection → Request A → Request B → Request C`

**Main improvement:** Persistent connections.

**Problem:** Head-of-Line blocking.

## HTTP/2

`One TCP connection → A + B + C in parallel`

**Main improvement:** Multiplexing.

**Problem:** TCP-level HOL blocking can still happen.

## HTTP/3

`QUIC/UDP → Independent streams`

**Main improvement:** Avoids TCP-level HOL blocking and improves performance on changing networks.

---

# 10. REST vs gRPC

The notes highlight:

**REST is an architectural style.**

If a REST API and client support HTTP/2 or HTTP/3, they can benefit from features such as:
- Multiplexing
- Header compression
- Lower latency

However, the notes point out that REST was designed in the HTTP/1.1 era, so many REST implementations may not fully use all advanced HTTP/2 features.

**gRPC** was built from the beginning on **HTTP/2**.

### Key Idea

- REST can use HTTP/2 or HTTP/3.
- gRPC is designed around HTTP/2.
- The notes present gRPC as often better suited for **high-performance microservices and APIs**.

---

# 11. HTTP Version Comparison

| Feature | HTTP/1.0 | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|---|
| Transport | TCP | TCP | TCP | QUIC/UDP |
| Persistent connection | No | Yes | Yes | Yes |
| Multiplexing | No | No | Yes | Yes |
| Binary protocol | No | No | Yes | Yes |
| Header compression | No | No | Yes | Yes |
| HOL blocking | Connection overhead | Yes | TCP-level | Avoids TCP-level |
| TLS | Separate | Separate | Separate TLS layer | TLS 1.3 in QUIC |
| 0-RTT | No | No | No | Yes |
| Main focus | Basic web | Persistent connections | Parallel requests | Independent streams + fast setup |

---

# 12. Important Terms

| Term | Meaning |
|---|---|
| **HTTP** | Protocol used for web communication |
| **TCP** | Reliable transport protocol used by HTTP/1.x and HTTP/2 |
| **UDP** | Connectionless transport used by QUIC |
| **QUIC** | Transport protocol used by HTTP/3 |
| **Persistent Connection** | Reuse one connection for multiple requests |
| **Multiplexing** | Multiple requests/responses over one connection |
| **HOL Blocking** | One blocked/lost request or packet delays other data |
| **Header Compression** | Reduces repeated header data |
| **Server Push** | Server sends resources before explicit request |
| **0-RTT** | Faster connection/session resumption |
| **Stream** | Independent sequence of data in HTTP/2 or HTTP/3 |
