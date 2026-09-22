# Monolithic vs Microservices — Short Notes

## 1. Monolithic Architecture

A **monolithic architecture** keeps the application's code together in a **single unit**.

### Basic Structure
`Client → Server → DB`

All major features can be part of the same application:
- Login & Signup
- Product Catalog
- Cart
- Payments
- Orders
- Search
- Notifications

The complete application can be packaged and deployed as one unit, such as a **JAR, WAR, EXE, or Docker image**.

### Key Idea
**Everything is deployed and managed together.**

---

# 2. Monolith — Pros

- **Simple to start** — good for small teams.
- **Easy deployment** — one application is deployed.
- **Fast performance** — function calls can happen directly inside the same application.
- **Easy development** — everything is in one codebase.
- **Easy debugging** — easier to trace problems.
- **Simple transactions** — operations can use the same database.
- **Easy local development** — fewer distributed components.

---

# 3. Monolith — Cons

- **Tightly coupled** — changes in one area can affect others.
- **Hard to scale independently** — the whole application may need scaling.
- **Large codebase** — becomes difficult to manage as the system grows.
- **Deployment risk** — a small change may require deploying the whole application.
- **Technology constraints** — components cannot easily use different technologies independently.
- **Large blast radius** — a problem in one part can affect the complete application.

---

# 4. Microservices Architecture

**Microservices architecture** breaks an application into **small, independent services**.

Each service:
- Handles a specific business capability.
- Can be developed independently.
- Can be deployed independently.
- Can be scaled independently.

### Example Services
- Login Service
- Product Catalog Service
- Cart Service
- Payment Service
- Order Service

Each service can communicate with other services.

### Basic Structure
`Client → Multiple Independent Services`

Each service may have its own database.

---

# 5. Monolith vs Microservices

| Feature | Monolithic | Microservices |
|---|---|---|
| Structure | Single application | Multiple independent services |
| Deployment | Together | Independently |
| Scaling | Usually whole application | Individual services |
| Codebase | One large codebase | Multiple smaller codebases |
| Database | Common database | Services can have separate DBs |
| Communication | Internal function calls | Network/API communication |
| Complexity | Lower initially | Higher |
| Debugging | Generally simpler | More distributed |
| Technology | More unified | Services can evolve independently |

---

# 6. How to Break a Monolith into Microservices

Do **not split the application randomly**.

Use **Domain-Driven Design (DDD)** and split based on **business capabilities**, not technical layers.

### Example

Instead of splitting into:
- Controller Service
- Database Service
- Authentication Service

Split by business capability:
- **User Service** → authentication, profiles
- **Order Service** → cart, checkout, order history
- **Payment Service** → payment processing
- **Product Service** → product catalog
- **Notification Service** → email, SMS, push

### Steps

1. **Don't split randomly — use Domain-Driven Design.**
2. Identify business capabilities.
3. Start with **one service at a time**.
4. **Expose APIs** between services.
5. Add an **API Gateway**.
6. Add **observability**.
7. **Repeat for other services.**

### Key Idea
**Break by business capability, not by technical layer.**

---

# 7. API Gateway

An **API Gateway** can act as a single entry point for clients when using microservices.

### Example
`Client → API Gateway → Service A`
`                    → Service B`
`                    → Service C`

It can help with:
- Routing requests
- Authentication
- Request handling
- Providing a common entry point

---

# 8. Why Separate Databases?

In microservices, services can have their **own databases**.

### Example
`Service A → DB A`
`Service B → DB B`
`Service C → DB C`

### Benefits

- **Data ownership** — each service owns its data.
- **Fault isolation** — problems in one database are less likely to directly affect another.
- **Independent scaling** — databases can be scaled according to service needs.
- **Independent technology choices** — services can potentially use different database technologies.

---

# 9. Shared Database Problem

Sometimes multiple services share the same database.

`Service A ─┐`
`Service B ─┼→ Shared DB`
`Service C ─┘`

Problems:
- Services become tightly coupled through the database.
- One service may depend on another service's tables.
- Database schema changes can affect multiple services.
- Data ownership becomes unclear.

### Key Idea
**Shared DB can create coupling at the data layer.**

---

# 10. Problems with Completely Separate Databases

Separate databases also introduce challenges.

Multiple databases can make it difficult to maintain **data consistency**, especially when one business operation involves multiple services.

Example:
`Order Service → Order DB`
`Payment Service → Payment DB`

If one operation needs both services to succeed, coordinating data across separate databases can be difficult.

---

# 11. Hybrid Approach

A **hybrid approach** is common in real-world systems.

Not every service has to follow exactly the same database or architecture pattern.

Some services may have:
- Their own databases
- Shared data where appropriate
- Different storage solutions based on requirements

### Key Idea
**Choose the architecture based on the business and technical requirements.**
