# API & REST APIs — Short Notes

## 1. What is an API?

**API = Application Programming Interface**

An API allows one **software/system to communicate with another** without exposing its internal implementation.

### Simple Example

`Customer → Waiter → Kitchen`

The customer does not directly enter the kitchen. The waiter carries the request and brings back the food.

Similarly:

`Client → API → Service`

The API acts as the communication interface between systems.

### Examples

**Swiggy / Zomato**
- App searches for "Burger".
- App calls an API.
- API returns available items.

**Flight Booking**
- Airline APIs provide flight schedules and prices.
- APIs can also be used to book tickets.

**Weather App**
- The app calls a weather API.
- The API provides weather data.

---

# 2. Types of APIs

Common API types mentioned:
- **REST**
- **GraphQL**
- **gRPC**
- **SOAP**

This topic mainly focuses on **REST APIs**.

---

# 3. REST API

**REST = Representational State Transfer**

REST is an **architectural style**, not a programming library.

REST commonly uses **HTTP** for communication between client and server.

### Basic Idea

`Client ↔ Server`

The client communicates with the server using **URLs + HTTP methods**.

---

# 4. Key Principles of REST

## A. Client-Server Separation

The **client** and **server** are independent.

Examples of clients:
- Web frontend
- Mobile app

The backend/server handles application logic and data.

**Key idea:** Client and server can evolve independently.

---

## B. Statelessness

Each API request is **independent**.

The server does not depend on remembering previous requests.

**Memory trick:**  
**Stateless = Server doesn't remember previous requests.**

---

## C. Uniform Interface

REST APIs use a consistent way of communicating.

| HTTP Method | Purpose |
|---|---|
| **GET** | Fetch/read data |
| **POST** | Create data |
| **PUT/PATCH** | Update data |
| **DELETE** | Remove data |

---

## D. Resource-Based URLs

In REST, everything is treated as a **resource** with a URL.

Examples:

`GET /users/123`

→ Get the user with ID `123`.

`POST /orders`

→ Create a new order.

The URL identifies the resource, while the HTTP method describes the operation.

---

# 5. JSON / XML

REST commonly exchanges data using **JSON** because it is lightweight.

Example:

```json
{
  "id": 123,
  "name": "Aniket",
  "city": "Delhi"
}
```

The server sends this data back to the client as part of the response.

---

# 6. Data Returned by an API

An API response contains:
- **Response body** — actual data.
- **Status code** — tells whether the request succeeded or failed.

---

# 7. Path Parameter

A **path parameter** is a value included directly in the URL path.

It identifies a specific resource.

### Example

`GET /users/123`

Here `123` is the path parameter.

**Meaning:** Get the user whose ID is 123.

---

# 8. Query Parameter

A **query parameter** is extra data added to the URL after `?`.

Used for:
- Filtering
- Sorting
- Searching
- Pagination

### Example

`GET /users?page=2&city=Delhi`

Here `page=2` and `city=Delhi` are query parameters.

### Easy Difference

**Path parameter → identifies a specific resource**

**Query parameter → filters/modifies the request**

---

# 9. Response Body

The **response body** contains the actual data returned by the server.

Example:

```json
{
  "id": 123,
  "name": "Aniket",
  "city": "Delhi"
}
```

Think of it as the **actual "food" returned from the server**.

---

# 10. HTTP Status Codes

A **status code** tells the client whether the request worked or failed.

| Code | Meaning |
|---|---|
| **200 OK** | Request succeeded |
| **201 Created** | New resource created |
| **400 Bad Request** | Client sent invalid input |
| **401 Unauthorized** | Authentication is required/failed |
| **404 Not Found** | Resource does not exist |
| **500 Internal Server Error** | Server encountered an error |

---

# 11. When Should We Use REST APIs?

## 1. Client-Server Applications

Useful for communication between:
- Mobile app
- Backend

Example:

`Mobile App → GET /restaurants?city=Delhi`

---

## 2. Web Applications

Frontend frameworks such as:
- React
- Angular
- Vue

can communicate with backends such as:
- Node.js
- Spring Boot
- Django

using REST APIs.

`React Frontend ↔ REST API ↔ Backend`

---

## 3. Public APIs

REST can be used when third-party developers need access to a service.

Examples mentioned:
- Stripe Payments API
- Google Maps API
- Twitter API

REST is easy to understand and widely supported.

---

## 4. Microservices Communication

Different services can communicate using REST.

Example:

`Order Service → REST API → Inventory Service`

`GET /inventory/123`

---

## 5. CRUD Applications

**CRUD = Create, Read, Update, Delete**

| CRUD Operation | HTTP Method |
|---|---|
| Create | **POST** |
| Read | **GET** |
| Update | **PUT/PATCH** |
| Delete | **DELETE** |

Useful for resources such as users, posts, and orders.

---

# 12. When NOT to Use REST

## A. Real-Time Communication

For applications requiring real-time updates, REST may not be the best fit.

Examples:
- Chat applications
- Live updates

Better options mentioned:
- **WebSockets**
- **gRPC**

---

## B. Over-Fetching / Under-Fetching

REST can return a fixed response structure.

Sometimes the client may receive:
- **Too much data** → Over-fetching
- **Too little data** → Under-fetching

Example:

A mobile app needs only a user's name and photo, but REST returns the full user profile.

The notes suggest **GraphQL** as an alternative when this is a major concern.

---

## C. High-Performance Service-to-Service Communication

REST commonly uses text-based formats such as **JSON**.

For very high-performance service-to-service communication, the notes suggest **gRPC**, which uses a binary format and can be faster.

---

# 13. REST — Quick Comparison

| Concept | Meaning |
|---|---|
| **API** | Interface for communication between software systems |
| **REST** | Architectural style commonly using HTTP |
| **GET** | Read/fetch |
| **POST** | Create |
| **PUT/PATCH** | Update |
| **DELETE** | Delete |
| **Path Parameter** | Identifies a resource |
| **Query Parameter** | Filters/modifies a request |
| **Response Body** | Data returned by server |
| **Status Code** | Result of the request |
| **JSON** | Common lightweight data format |
