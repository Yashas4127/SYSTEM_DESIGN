# Client-Server Architecture — Short Notes

## 1. Basic Client-Server Architecture

A **client** requests a service/data, and a **server** processes the request and sends back a response.

### Basic Flow
`User → Client (App/Browser) → Server → Response → Client`

Example: When you type `abc.com` in a browser:
1. Browser acts as the **client**.
2. Client sends a **request** to the server.
3. Server processes the request.
4. Server sends a **response** back.

---

# 2. IP Address

**IP = Internet Protocol**

An **IP address** is an address used to identify a device on a network.

## Types of IP Addresses

### IPv4

- Internet Protocol version 4
- **32-bit** address
- Written as **4 numbers separated by dots**
- Example: `192.168.1.1`
- Each number (octet) ranges from **0 to 255**
- Around **4.3 billion** possible addresses

### IPv6

- Internet Protocol version 6
- **128-bit** address
- Written as **8 groups of hexadecimal numbers separated by colons**
- Example: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Provides a much larger address space than IPv4

### Why IPv6?

The number of connected devices is increasing and IPv4 addresses are limited. IPv6 provides a much larger address space.

---

# 3. Public and Private IP Addresses

IP addresses can be categorized as:

### Public IP
- Used for communication over the **Internet**.
- Can identify a network/device to the outside world.

### Private IP
- Used inside a **local/private network**.
- Devices such as laptops, phones, and TVs can use private IP addresses.
- A router commonly uses **NAT (Network Address Translation)** to connect private devices to the Internet using a public IP.

### Example from the diagram
`Internet → Router (uses NAT) → Laptop / Phone / TV`

---

# 4. DNS

**DNS = Domain Name System**

DNS converts a **domain name** into an **IP address**.

### Example

`abc.com → DNS → IP address`

Instead of remembering an IP address, users can type a domain name such as `abc.com`.

### Basic Flow

`Client → DNS → IP Address → Server`

The client uses the resolved IP address to communicate with the server.

---

# 5. OSI Model

The **OSI model has 7 layers**.

The notes specifically mention:

### Transport Layer
Responsible for transport-level communication.

Examples:
- **TCP**
- **UDP**

### Application Layer
Provides network services to applications.

Examples mentioned:
- HTTP / HTTPS
- WebSockets
- WebRTC

---

# 6. TCP

**TCP = Transmission Control Protocol**

TCP provides **reliable, connection-oriented communication** between a client and server.

### TCP Connection

Before data transfer, TCP establishes a connection using a **handshake**.

The diagram shows:
- `SYN`
- `SYN-ACK`
- `ACK`

This establishes the connection.

### Key Features of TCP

- **Connection-oriented:** Establishes a connection before transferring data.
- **Reliable:** Retransmits lost packets.
- **Ordered:** Data packets arrive in the correct sequence.
- **Error checking:** Detects errors and handles retransmission.
- **Flow control:** Helps prevent the receiver from being overloaded.
- **Congestion control:** Adjusts sending rate to reduce network congestion.

### TCP Connection Closing

TCP also has a process for ending the connection after communication is complete.

---

# 7. UDP

**UDP = User Datagram Protocol**

UDP provides **connectionless communication**.

### Main Features

- **Connectionless:** No connection is established before sending data.
- **No guaranteed ordering:** Packets may arrive out of order.
- **Fast:** Less overhead than TCP.
- **No guaranteed delivery:** Lost packets are not automatically retransmitted.

### When is UDP useful?

UDP is useful when **speed is more important than reliability**.

### Examples
- Live streaming
- Online gaming

---

# 8. TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable | Not guaranteed |
| Ordering | Maintains order | No guaranteed order |
| Retransmission | Yes | No |
| Speed | Generally slower | Generally faster |
| Use case | Reliable data transfer | Real-time/fast communication |

### Easy Memory Trick

**TCP = Reliable**

**UDP = Fast**

---

# 9. Important Terms

| Term | Meaning |
|---|---|
| **Client** | Requests a service/data |
| **Server** | Processes requests and provides responses |
| **IP Address** | Identifies a device/network |
| **IPv4** | 32-bit IP addressing |
| **IPv6** | 128-bit IP addressing |
| **DNS** | Converts domain name to IP address |
| **TCP** | Reliable, connection-oriented protocol |
| **UDP** | Fast, connectionless protocol |
| **NAT** | Translates private IPs for Internet communication |
| **OSI Model** | 7-layer networking model |
