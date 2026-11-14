# Scaling Systems & Load Balancing — Short Notes

## 1. Introduction

**Scaling** means increasing the capacity of a system so it can handle more users, requests, or data.

A system may need scaling when:
- Number of users increases
- Requests increase
- Response time becomes high
- One server cannot handle the workload

Two main approaches:
- **Vertical Scaling**
- **Horizontal Scaling**

---

# 2. Vertical Scaling

**Vertical scaling = Increase the power of one server.**

Instead of adding more servers, we improve the existing server.

### Example

Before:
`Server → 4 GB RAM + 2 CPU`

After:
`Server → 16 GB RAM + 8 CPU`

### Advantages
- Simple to implement
- Usually requires fewer changes to the application
- Easy to manage

### Disadvantages
- Hardware has a maximum limit
- Can become expensive
- The single server can become a **single point of failure**

**Memory trick:**  
**Vertical = Make one machine stronger**

---

# 3. Horizontal Scaling

**Horizontal scaling = Add more servers.**

Instead of making one server more powerful, we add multiple servers to handle the workload.

### Example

`Client → Server 1`
`       → Server 2`
`       → Server 3`

A load balancer can distribute requests among these servers.

### Advantages
- Can handle large traffic
- Easier to expand by adding servers
- Better availability when designed correctly
- Workload can be distributed

### Disadvantages
- More complex architecture
- Requires request distribution
- Data/session management can become more complicated

**Memory trick:**  
**Horizontal = Add more machines**

---

# 4. Vertical vs Horizontal Scaling

| Feature | Vertical Scaling | Horizontal Scaling |
|---|---|---|
| Basic idea | Increase server power | Add more servers |
| Number of servers | Usually one | Multiple |
| Expansion | Upgrade hardware | Add machines |
| Complexity | Lower | Higher |
| Maximum capacity | Limited by machine | Can scale by adding machines |
| Failure handling | Single server can be a risk | Multiple servers can improve availability |

---

# 5. Load Balancing

**Load balancing** means distributing incoming requests across multiple servers.

### Basic Flow

`Clients → Load Balancer → Server 1`
`                       → Server 2`
`                       → Server 3`

### Why use a Load Balancer?

- Distributes traffic
- Prevents one server from becoming overloaded
- Improves availability
- Helps horizontal scaling
- Can route traffic away from unhealthy servers

### Simple Example

Suppose 300 requests arrive:

- Server 1 → 100 requests
- Server 2 → 100 requests
- Server 3 → 100 requests

Instead of sending all 300 requests to one server, the load balancer distributes them.

---

# 6. Client-Side Load Balancing

In **client-side load balancing**, the client knows about multiple available servers and selects one to send the request to.

### Flow

`Client → Server 1 / Server 2 / Server 3`

The client-side component decides which server should receive the request.

### Key idea

**The client participates in choosing the server.**

### Advantages
- Can avoid a separate centralized load-balancer layer
- Client can make routing decisions

### Disadvantages
- Client needs server information
- More responsibility is placed on the client
- Service discovery and server health information may be required

---

# 7. Server-Side Load Balancing

In **server-side load balancing**, the client sends the request to a load balancer.

The load balancer chooses the backend server.

### Flow

`Client → Load Balancer → Server 1`
`                     → Server 2`
`                     → Server 3`

### Key idea

**The load balancer chooses the server.**

### Advantages
- Client remains simple
- Centralized traffic management
- Can perform health checks
- Can remove unhealthy servers from routing

### Disadvantage
- The load balancer itself needs high availability so it does not become a single point of failure.

---

# 8. Client-Side vs Server-Side Load Balancing

| Feature | Client-Side | Server-Side |
|---|---|---|
| Server selection | Client | Load balancer |
| Extra LB layer | Not necessarily | Yes |
| Client complexity | Higher | Lower |
| Traffic control | Distributed | Centralized |
| Health/routing logic | Often on client | On load balancer |

---

# 9. Load Balancing Algorithms

A **load balancing algorithm** decides **which server should receive the next request**.

Common algorithms include:

## A. Round Robin

Requests are distributed in sequence.

Example:

`Request 1 → Server 1`
`Request 2 → Server 2`
`Request 3 → Server 3`
`Request 4 → Server 1`

**Idea:** Take turns.

---

## B. Weighted Round Robin

Servers receive requests according to assigned weights.

Example:

- Server 1 → Weight 3
- Server 2 → Weight 2
- Server 3 → Weight 1

A more powerful server can receive more requests.

**Idea:** More capacity → Higher weight.

---

## C. Least Connections

Send the new request to the server having the **fewest active connections**.

Example:

- Server 1 → 10 connections
- Server 2 → 4 connections
- Server 3 → 7 connections

New request → **Server 2**

**Idea:** Choose the least busy server.

---

## D. IP Hash

The client's IP address is used to determine the server.

The same client can therefore be directed to the same server when the mapping remains unchanged.

**Idea:** Client IP → Server selection.

---

# 10. Layer 4 Load Balancer

A **Layer 4 (Transport Layer) load balancer** works using transport-level information.

It commonly uses:
- IP address
- Port
- TCP
- UDP

It generally does not need to understand the application's HTTP request details.

### Example

`Client → L4 Load Balancer → TCP/UDP Server`

**Memory trick:**  
**L4 = Transport information**

---

# 11. Layer 7 Load Balancer

A **Layer 7 (Application Layer) load balancer** understands application-level information.

For web traffic, it can use information such as:
- HTTP method
- URL/path
- Host/domain
- Headers

### Example

`/users → User Service`
`/orders → Order Service`

**Memory trick:**  
**L7 = Application/HTTP information**

---

# 12. Layer 4 vs Layer 7 Load Balancer

| Feature | Layer 4 | Layer 7 |
|---|---|---|
| OSI layer | Transport | Application |
| Common protocols | TCP/UDP | HTTP/HTTPS |
| Routing information | IP + Port | URL, Host, Headers, etc. |
| Application awareness | Lower | Higher |
| Routing flexibility | Lower | Higher |

---

# 13. SPOF in Load Balancer

**SPOF = Single Point of Failure**

A component is a single point of failure if its failure can make the whole system unavailable.

### Problem

`Clients → One Load Balancer → Servers`

If the only load balancer fails, clients may not be able to reach the servers.

### How to avoid SPOF?

Use multiple load balancers and provide **high availability**.

Example:

`Clients → LB 1`
`       → LB 2`
`              ↓`
`        Multiple Servers`

If one load balancer fails, another can continue handling traffic.

**Key idea:**  
**Never depend on only one critical load-balancing component.**

---

# 14. Active and Passive Load Balancer

## Active Load Balancer

The active load balancer **actively handles traffic**.

Example:

`Clients → Active LB → Servers`

It performs request distribution during normal operation.

---

## Passive Load Balancer

The passive load balancer is kept as a **backup**.

It can take over when the active load balancer fails.

### Basic idea

`Clients → Active LB → Servers`
`              ↓ failure`
`          Passive LB takes over`

This provides failover and improves availability.

---

# 15. Active vs Passive

| Feature | Active | Passive |
|---|---|---|
| Handles normal traffic | Yes | Usually no |
| Main purpose | Traffic distribution | Backup/failover |
| During failure | Continues if healthy | Takes over |
| Availability role | Primary | Standby |

---

# 16. Important Terms

| Term | Meaning |
|---|---|
| **Scaling** | Increasing system capacity |
| **Vertical Scaling** | Make one server more powerful |
| **Horizontal Scaling** | Add more servers |
| **Load Balancing** | Distribute requests across servers |
| **Client-Side LB** | Client selects backend server |
| **Server-Side LB** | Load balancer selects backend server |
| **Round Robin** | Servers are selected in sequence |
| **Weighted Round Robin** | Selection depends on server weight |
| **Least Connections** | Select server with fewest connections |
| **IP Hash** | Use client IP for server selection |
| **L4 LB** | Works at transport level |
| **L7 LB** | Works at application level |
| **SPOF** | Single component whose failure can affect the system |
| **Active LB** | Handles traffic normally |
| **Passive LB** | Backup that can take over |
