# GraphQL — Short Notes

## 1. What is GraphQL?

**GraphQL** is a **query language for APIs**.

- Developed by **Facebook in 2012**.
- Unlike REST, where the server defines the response structure, GraphQL allows the **client to specify exactly what data it wants**.
- GraphQL commonly uses a **single endpoint** instead of many REST endpoints.

### Simple Idea

**REST:** Server decides what data you get.

**GraphQL:** Client asks for exactly the data it needs.

Think of:
- REST → fixed meal
- GraphQL → choose exactly what you want

---

# 2. Why GraphQL Was Created

The notes identify some problems with REST.

## A. Over-fetching

REST may return **more data than required**.

Example:

`GET /users/123`

may return the complete user profile, while a mobile app only needs:
- Name
- Photo

**Problem:** Unnecessary data is transferred.

---

## B. Under-fetching

Sometimes one REST API call does not provide enough data.

Example:

To show a user's profile and recent posts, the client may need:

`GET /users/123`

and

`GET /users/123/posts`

This can require **multiple round trips**.

**Problem:** More requests can make the application slower.

---

## C. Versioning

REST APIs may use versions such as:

`/v1/users`

`/v2/users`

As APIs evolve, maintaining multiple versions can become difficult.

GraphQL uses flexible queries, which can reduce the need for traditional API versioning.

---

# 3. GraphQL Operations

GraphQL has **3 main operations**:

## 1. Query

Used to **read data**.

Similar to:
`GET` in REST.

---

## 2. Mutation

Used to:
- Create data
- Update data
- Delete data

Similar to:
`POST / PUT / DELETE` in REST.

---

## 3. Subscription

Used for **real-time updates**.

Similar to the real-time communication idea provided by **WebSockets**.

Example:

`Preparing → Out for delivery → Delivered`

The client can receive updates automatically instead of repeatedly polling.

---

# 4. GraphQL Query — Read Data

The client specifies exactly which fields it needs.

Example:

```graphql
query {
  user(id: "123") {
    name
    email
    orders {
      id
      total
      items {
        product {
          name
          price
        }
        quantity
      }
    }
  }
}
```

The response contains only the requested fields.

### Key Idea

**Client controls the shape of the response.**

---

# 5. GraphQL Mutation

A **mutation** changes data.

Example:

```graphql
mutation {
  createOrder(
    userId: "123",
    items: [{productId: "X1", quantity: 2}]
  ) {
    id
    total
    status
  }
}
```

The server processes the operation and returns the requested fields.

---

# 6. GraphQL Subscription

A **subscription** provides real-time updates.

Example:

```graphql
subscription {
  orderStatusChanged(orderId: "999") {
    status
    estimatedDeliveryTime
  }
}
```

When the order status changes, the server pushes new data to the client.

**Key idea:** Subscription = Real-time updates

---

# 7. Schema & Types

GraphQL uses a **Schema Definition Language (SDL)**.

GraphQL is **strongly typed**.

This means every field has a defined type.

Example:

```graphql
type User {
  id: ID!
  name: String
  email: String
  orders: [Order!]
}

type Order {
  id: ID!
  total: Float!
  status: String
}
```

The client sends a query → server checks it against the schema → valid data is returned.

**Key idea:** Schema defines what data and operations are available.

---

# 8. Benefits of GraphQL

## 1. Client Controls Data

The client requests exactly what it needs.

This helps reduce:
- Over-fetching
- Under-fetching

## 2. One Endpoint

GraphQL commonly exposes a **single endpoint**.

## 3. Strong Typing

The schema defines the available fields and their types.

## 4. Versioning Is Often Reduced

APIs can evolve through schema changes and field deprecation rather than maintaining many URL versions.

## 5. Good for Mobile Applications

Clients can request only the required fields.

## 6. Flexible Queries

Clients can ask for different shapes of data without requiring a separate endpoint for every response shape.

---

# 9. Challenges of GraphQL

## A. Server-Side Complexity

The server needs **resolvers** for fields and operations.

## B. Caching Can Be Harder

REST URLs can be easy to cache:

`/users/123`

GraphQL queries can vary while using the same endpoint, making caching more complex.

## C. Performance Risk

Clients can request **large or deeply nested data**.

Poorly designed queries can create high server load.

## D. Learning Curve

GraphQL introduces:
- Queries
- Mutations
- Subscriptions
- Schema
- Types
- Resolvers

---

# 10. When to Use GraphQL

The notes suggest GraphQL for:

## 1. Complex Systems

Useful for systems with many relationships:
- Social networks
- E-commerce
- Dashboards

## 2. Different Client Data Requirements

Useful when web and mobile clients need **different shapes of data**.

## 3. Real-Time Applications

Useful for applications such as:
- Chat
- Live tracking

---

# 11. When to Stick to REST

The notes suggest REST when:

## 1. Simple CRUD APIs

For simple:
- Create
- Read
- Update
- Delete

REST can be sufficient.

## 2. Heavy Caching Is Needed

CDNs generally work well with REST-style URL-based resources.

## 3. Small Team + Simple Requirements

REST may avoid unnecessary complexity when requirements are straightforward.

---

# 12. GraphQL vs REST

| Feature | REST | GraphQL |
|---|---|---|
| Endpoints | Usually multiple | Commonly single endpoint |
| Data selection | Server defines response | Client selects fields |
| Over-fetching | More likely | Reduced |
| Under-fetching | More likely | Reduced |
| Versioning | Often `/v1`, `/v2` | Often schema evolution |
| Read operation | GET | Query |
| Write operation | POST/PUT/DELETE | Mutation |
| Real-time | Usually separate mechanism | Subscription |
| Caching | Generally simpler | More complex |
| Query flexibility | Lower | Higher |
| Server complexity | Generally simpler | Higher |

---

# 13. Important Terms

| Term | Meaning |
|---|---|
| **GraphQL** | Query language for APIs |
| **Query** | Read data |
| **Mutation** | Create/update/delete data |
| **Subscription** | Real-time updates |
| **Schema** | Defines available data and operations |
| **SDL** | Schema Definition Language |
| **Strongly Typed** | Fields have defined types |
| **Resolver** | Server-side logic that provides field data |
| **Over-fetching** | Getting more data than needed |
| **Under-fetching** | Not getting enough data in one request |
