# Authentication & Authorization — Short Notes

## 1. Authentication

**Authentication = Proving who you are.**

The system checks your identity using credentials such as:
- Username + password
- OTP
- Fingerprint
- API key

If the credentials are valid → **you are authenticated**.

### Real-Life Example

At an airport:
- Security checks your ID card + ticket.
- If the details match, you are allowed inside.

**Memory trick:**  
**Authentication = WHO are you?**

---

# 2. Authorization

**Authorization = What you are allowed to do.**

After authentication, the system checks:
- Roles
- Permissions
- Resources you can access
- Actions you can perform

### Real-Life Example

If you have an economy ticket, you cannot access the business class area.

**Memory trick:**  
**Authorization = WHAT can you do?**

---

# 3. Authentication in APIs

API authentication checks **who is calling the API**.

The client provides credentials such as:
- API key
- Username/password
- OAuth token

If credentials are valid:
→ API issues a session/token.

If authentication fails:
→ **401 Unauthorized**

---

# 4. Common Authentication Methods

## 1. API Key Authentication

A unique key is generated for each developer/user.

The key is usually sent with every request through:
- Header
- Query string

### Example

```text
GET /weather
X-API-Key: abc123xyz
```

### Pros
- Easy to use
- Widely supported

### Cons
- Not very secure if leaked
- Can be exposed through logs or URLs
- Less suitable for user-facing APIs

---

## 2. Basic Authentication

Uses:
- Username
- Password

The credentials are encoded in **Base64** and sent in the `Authorization` header.

### Example

```text
GET /profile
Authorization: Basic <encoded-credentials>
```

### Pros
- Simple to implement

### Cons
- Should be used with **HTTPS**
- Credentials are sent with every request

---

## 3. Session-Based Authentication (Cookies)

A classic web authentication approach.

### Flow

1. User logs in.
2. Server creates a session.
3. Server stores the session.
4. Server sends a cookie to the client.
5. Browser sends the cookie with every request.

### Example

```text
Cookie: sessionId=abc123xyz
```

### Pros
- Common for traditional websites

### Cons
- Less common for modern stateless APIs.
- REST APIs often prefer tokens.

---

## 4. Bearer Token / JWT

**JWT = JSON Web Token**

### Flow

1. User logs in.
2. Server issues a token.
3. Client sends the token with every request.
4. Server verifies the token.

### Example

```text
GET /orders
Authorization: Bearer <token>
```

A JWT can contain information such as:
- User information
- Roles
- Expiry time

### Pros
- Supports stateless APIs
- No need to store a traditional server session

### Cons
- If the token is stolen, an attacker can use it until it expires.

---

# 5. OAuth 2.0

**OAuth = Open Authorization**

OAuth is a **delegated authorization framework**.

It allows a user to grant **limited access** to their resources without sharing their password.

### Example: Login with Google

You log into an application using **Google**.

The application does not receive your Google password.

Instead, it receives a token that gives limited access to allowed information, such as:
- Email
- Profile

### Pros
- Useful for third-party applications
- Allows limited access

### Cons
- More complex to implement

---

# 6. Mutual TLS (mTLS)

**mTLS = Mutual Transport Layer Security**

In normal TLS, the server proves its identity to the client.

With **mTLS**:
- Client presents a certificate.
- Server presents a certificate.
- Both sides verify each other.

### Use cases
Often used in:
- Banking
- Financial systems
- Enterprise APIs

**Key idea:**  
**Both client and server are verified.**

---

# 7. Authorization in APIs

API authorization checks **what the authenticated caller can do**.

After authentication, the API checks:
- Roles
- Permissions
- Requested resource

### Example

User with:

`role=user`

→ Can access:

`/profile`

User with:

`role=admin`

→ Can access:

`/admin/reports`

If a user tries to access something outside their permissions:

→ **403 Forbidden**

---

# 8. Authentication vs Authorization

| Feature | Authentication | Authorization |
|---|---|---|
| Main question | Who are you? | What can you do? |
| Happens | First | After authentication |
| Checks | Identity/credentials | Roles/permissions |
| Example | Login | Access admin page |
| Failure | **401 Unauthorized** | **403 Forbidden** |

### Easy Memory Trick

**AuthN → Identity**

**AuthZ → Permissions**

---

# 9. Practical API Authentication Flow

## Step 1: Authentication / Login

Client sends login credentials:

```text
POST /login
Content-Type: application/json

{
  "username": "aniket",
  "password": "mypassword"
}
```

Server returns a token:

```json
{
  "token": "eyJhbGciOiJIUzI1Ni..."
}
```

Now the user is **authenticated**.

---

## Step 2: Authorization / Access Control

Client uses the token to request protected data:

```text
GET /admin/reports
Authorization: Bearer <token>
```

If token has:

`role=admin`

→ Request succeeds.

If token has:

`role=user`

→ Request is **forbidden**.

---

# 10. Important Error Codes

## 401 Unauthorized

Means the caller is **not authenticated**.

Possible reasons:
- Missing credentials
- Invalid credentials
- Invalid/expired authentication token

**Memory:**  
**401 = Who are you?**

---

## 403 Forbidden

Means the caller **is authenticated but does not have permission**.

Example:

A normal user tries to access an admin resource.

**Memory:**  
**403 = I know who you are, but you can't do this.**

---

# 11. API Headers

**Headers** are key-value pairs sent with an API request or response.

They provide metadata about the request/response.

Without headers, the server/client may not know important details such as:
- Authentication information
- Data format
- Expected response format
- Caching rules

---

# 12. Types of API Headers

## A. General Headers

Can apply to both requests and responses.

### Examples

**Date**

Tells when the message was sent.

```text
Date: Sat, 13 Sep 2025 12:00:00 GMT
```

**Cache-Control**

Provides caching instructions.

```text
Cache-Control: no-cache
```

---

## B. Request Headers

Sent from **client → server**.

They provide details about what the client wants.

### Authorization

Carries credentials or tokens.

```text
Authorization: Bearer <token>
```

### Content-Type

Tells the server the format of the request body.

```text
Content-Type: application/json
```

### Accept

Tells the server the response format expected by the client.

```text
Accept: application/json
```

### User-Agent

Provides information about the client, such as browser/app and version.

```text
User-Agent: Mozilla/5.0
```

### Host

Specifies the server/domain being requested.

```text
Host: api.example.com
```

---

# 13. Response Headers

Sent from **server → client**.

They provide metadata about the response.

### Content-Type

Tells the client the format of the response body.

```text
Content-Type: application/json
```

### Set-Cookie

Server sets a cookie on the client.

```text
Set-Cookie: sessionId=xyz123; HttpOnly
```

### Access-Control-Allow-Origin

Used in **CORS (Cross-Origin Resource Sharing)**.

```text
Access-Control-Allow-Origin: *
```

### Content-Length

Tells the size of the response body in bytes.

```text
Content-Length: 340
```

---

# 14. Common API Headers

| Header | Purpose |
|---|---|
| **Authorization** | Auth token / API key |
| **Content-Type** | Format of request/response body |
| **Accept** | Expected response format |
| **Cache-Control** | Caching rules |
| **User-Agent** | Information about the client |

---

# 15. Quick Comparison

| Concept | Simple Meaning |
|---|---|
| **Authentication** | Prove who you are |
| **Authorization** | Check what you can do |
| **API Key** | Unique key used to identify/access an API |
| **Basic Auth** | Username + password |
| **Session/Cookie** | Server-side session identified by a cookie |
| **JWT** | Token-based authentication |
| **OAuth 2.0** | Delegated/limited access |
| **mTLS** | Both client and server verify certificates |
| **401** | Not authenticated |
| **403** | Authenticated but not permitted |
| **API Header** | Metadata sent with request/response |
