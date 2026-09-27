# gRPC Pagination — Short Notes

## 1. Introduction to Pagination in gRPC

When a gRPC service has a large number of records, returning everything in one response can cause:

- Large response size
- High memory usage
- More network usage
- Slow responses
- Higher server/client processing

**Pagination = Returning large data in smaller portions.**

In gRPC, pagination can commonly be implemented using:

1. **Offset-Based Pagination**
2. **Cursor-Based Pagination**
3. **Streaming-Based Pagination**

---

# 2. Offset-Based Pagination in gRPC

Offset-based pagination uses:

- `offset` → Number of records to skip
- `limit` → Number of records to return

### Example

```text
offset = 20
limit  = 10
```

Meaning:

**Skip 20 records and return the next 10.**

### gRPC Request

In gRPC, these values are normally defined in a `.proto` message.

```proto
message ListProductsRequest {
  int32 limit = 1;
  int32 offset = 2;
}
```

The RPC can be:

```proto
service ProductService {
  rpc ListProducts(ListProductsRequest)
      returns (ListProductsResponse);
}
```

---

# 3. Offset-Based Pagination — Response

The response can contain:

- Current data
- Offset
- Limit
- Whether more records exist

Example:

```proto
message ListProductsResponse {
  repeated Product products = 1;
  int32 offset = 2;
  int32 limit = 3;
  bool has_more = 4;
}
```

### Flow

```text
Client
  ↓
ListProducts(offset=0, limit=10)
  ↓
Server
  ↓
Products 1–10
  ↓
Client
  ↓
ListProducts(offset=10, limit=10)
  ↓
Products 11–20
```

---

# 4. Offset-Based Pagination — Server Logic

Suppose the client sends:

```text
offset = 20
limit = 10
```

The server can execute a database query such as:

```sql
SELECT *
FROM products
ORDER BY id
LIMIT 10 OFFSET 20;
```

The server then places the result into the gRPC response.

---

# 5. Offset-Based Pagination — Pros

- Simple to understand.
- Easy to implement in a gRPC service.
- Easy to support page numbers.
- Works well for small or mostly-static datasets.

---

# 6. Offset-Based Pagination — Cons

### A. Deep Offset

Large offsets can become inefficient.

```text
offset = 100000
```

The database may need to skip many rows.

### B. Data Changes

If records are inserted/deleted while clients are paging:

- Records may be duplicated.
- Records may be skipped.

### C. Large Datasets

For very large and frequently changing datasets, cursor-based pagination can be more suitable.

---

# 7. Hands-On Offset-Based Pagination

### `.proto`

```proto
message ListProductsRequest {
  int32 limit = 1;
  int32 offset = 2;
}

message ListProductsResponse {
  repeated Product products = 1;
  bool has_more = 2;
}

service ProductService {
  rpc ListProducts(ListProductsRequest)
      returns (ListProductsResponse);
}
```

### Client Request

Conceptually:

```text
limit = 5
offset = 10
```

### Server

```sql
SELECT *
FROM products
ORDER BY id
LIMIT 5 OFFSET 10;
```

### Result

The server returns products **11–15**.

---

# 8. Cursor-Based Pagination in gRPC

Cursor-based pagination uses a **cursor/token** instead of an offset.

The cursor represents the position from which the next page should continue.

### Basic Idea

Instead of:

```text
offset = 100
```

use:

```text
cursor = "abc123"
```

The client sends the cursor to the server.

---

# 9. gRPC Cursor Request

The cursor can be defined in the `.proto` request.

```proto
message ListProductsRequest {
  int32 limit = 1;
  string cursor = 2;
}
```

The RPC remains:

```proto
rpc ListProducts(ListProductsRequest)
    returns (ListProductsResponse);
```

---

# 10. gRPC Cursor Response

The server can return:

```proto
message ListProductsResponse {
  repeated Product products = 1;
  string next_cursor = 2;
  bool has_more = 3;
}
```

### Flow

```text
Client
  ↓
ListProducts(limit=10)
  ↓
Server
  ↓
Products 1–10
next_cursor = ABC
  ↓
Client
  ↓
ListProducts(limit=10, cursor=ABC)
  ↓
Products 11–20
next_cursor = XYZ
```

---

# 11. What is a Cursor?

A cursor is an **opaque token** representing a position in the dataset.

The client should normally:

- Store it.
- Send it back.
- Not modify it.
- Not depend on its internal structure.

The server can internally encode values such as:

```text
last_id
created_at
sort information
```

---

# 12. Cursor + Keyset Pagination

A cursor is commonly used to remember the last position in a sorted dataset.

Suppose products are sorted by:

```text
id ASC
```

The cursor may represent:

```text
last_id = 10
```

The server can then query:

```sql
SELECT *
FROM products
WHERE id > 10
ORDER BY id ASC
LIMIT 10;
```

This avoids a large `OFFSET`.

### Key Idea

**Offset → Skip rows**

**Cursor → Continue from a known position**

---

# 13. Cursor-Based Pagination — Pros

- Efficient for large datasets.
- Better for frequently changing data.
- Avoids deep-offset problems.
- Good for infinite scrolling.
- Works well with indexed database queries.

---

# 14. Cursor-Based Pagination — Cons

- More complex than offset pagination.
- Difficult to jump directly to page 100.
- Cursor generation and validation are required.
- Cursor can depend on sorting/filtering.

---

# 15. Hands-On Cursor-Based Pagination

### `.proto`

```proto
message ListProductsRequest {
  int32 limit = 1;
  string cursor = 2;
}

message ListProductsResponse {
  repeated Product products = 1;
  string next_cursor = 2;
  bool has_more = 3;
}
```

### First Request

```text
limit = 5
cursor = ""
```

Server returns:

```text
Products 1–5
next_cursor = cursor-5
has_more = true
```

### Second Request

```text
limit = 5
cursor = cursor-5
```

Server returns:

```text
Products 6–10
next_cursor = cursor-10
```

---

# 16. Streaming-Based Pagination in gRPC

gRPC has a major feature that is useful for large result sets:

**Server Streaming RPC**

Instead of returning one large response, the server can send **multiple messages over one RPC call**.

### `.proto`

```proto
rpc StreamProducts(StreamProductsRequest)
    returns (stream Product);
```

The important part is:

```proto
returns (stream Product)
```

This means the server can send many `Product` messages.

---

# 17. How gRPC Server Streaming Works

### Normal Unary RPC

```text
Client → Request
Client ← One Response
```

### Server Streaming RPC

```text
Client → One Request

Server → Product 1
Server → Product 2
Server → Product 3
Server → Product 4
...
```

The client receives messages as they arrive.

---

# 18. Why Use gRPC Streaming?

Streaming is useful when:

- Dataset is very large.
- The client can process data incrementally.
- You don't want one huge response.
- Results should start arriving before the complete dataset is ready.
- Large exports or long-running operations are involved.

### Main Benefit

**The client does not need to wait for the entire dataset before processing it.**

---

# 19. Hands-On Streaming-Based Pagination

### `.proto`

```proto
message StreamProductsRequest {
  int32 batch_size = 1;
}

service ProductService {
  rpc StreamProducts(StreamProductsRequest)
      returns (stream Product);
}
```

### Request

```text
batch_size = 100
```

The server can read records in batches and send them one by one:

```text
Client → StreamProducts(batch_size=100)

Server → Product 1
Server → Product 2
...
Server → Product 100
Server → Product 101
...
```

The client processes each message as it arrives.

---

# 20. Streaming vs Offset/Cursor

### Offset/Cursor

```text
Request Page 1
      ↓
Response Page 1

Request Page 2
      ↓
Response Page 2

Request Page 3
      ↓
Response Page 3
```

### gRPC Streaming

```text
One Request
      ↓
Message 1
Message 2
Message 3
Message 4
...
```

The server keeps sending messages until the stream finishes.

---

# 21. Offset vs Cursor vs Streaming in gRPC

| Feature | Offset | Cursor | Server Streaming |
|---|---|---|---|
| Main idea | Skip + take | Continue from token | Send multiple messages |
| gRPC request | `offset`, `limit` | `cursor`, `limit` | Streaming request |
| Response | One message | One message | Multiple messages |
| Page numbers | Easy | Difficult | Not the main goal |
| Large datasets | Less suitable | Good | Excellent |
| Changing data | Less stable | More stable | Depends on implementation |
| Infinite scroll | Good | Excellent | Possible |
| Large exports | Not ideal | Good | Excellent |
| Implementation | Easy | Medium/complex | Medium/complex |

---

# 22. When to Use Each in gRPC

## Use Offset-Based Pagination

When:
- Dataset is small/medium.
- Page numbers are required.
- Users need jump-to-page.
- Data does not change frequently.

Example:

```text
Admin Dashboard
Page 1 → Page 2 → Page 3
```

---

## Use Cursor-Based Pagination

When:
- Dataset is large.
- Data changes frequently.
- Deep pagination is required.
- Infinite scrolling is needed.

Example:

```text
Mobile Feed
↓
Load More
↓
Load More
```

---

## Use gRPC Server Streaming

When:
- Very large amounts of data need to be transferred.
- Data can be processed incrementally.
- You want one RPC call to deliver many results.
- Large exports or long-running operations are involved.

Example:

```text
Generate Report
      ↓
gRPC Stream
      ↓
Records arrive continuously
      ↓
Client processes records
```

---

# 23. Important gRPC Pagination Terms

| Term | Meaning |
|---|---|
| **Pagination** | Splitting large results into smaller portions |
| **Offset** | Number of records to skip |
| **Limit** | Number of records to return |
| **Cursor** | Token representing a position |
| **Keyset Pagination** | Continue from the last known key |
| **Opaque Cursor** | Cursor whose internal value is hidden from the client |
| **Unary RPC** | One request → one response |
| **Server Streaming** | One request → multiple server responses |
| **Stream** | Continuous sequence of gRPC messages |
| **`.proto`** | Defines the gRPC service and message contract |
| **`has_more`** | Indicates whether more records exist |
| **`next_cursor`** | Cursor used to request the next page |
