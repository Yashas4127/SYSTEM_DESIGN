# GraphQL Pagination — Short Notes

## 1. What is GraphQL Pagination?

GraphQL APIs can also return large datasets, so they need **pagination**.

GraphQL pagination is not completely different from REST.

**GraphQL is the query language; pagination rules are defined in the GraphQL schema using types and arguments.**

The notes cover two common styles:

1. **Simple List — Offset/Page-Based**
2. **Relay Connection — Cursor-Based**

---

# 2. Why Use Pagination?

Without pagination, an API may return too many records at once.

This can cause:
- Slow responses
- More memory usage
- More bandwidth usage
- Higher server load

Pagination breaks a large dataset into smaller pages.

---

# 3. Simple List Pagination

The simple-list approach uses **page/offset + limit**.

It is:
- Familiar
- Easy to implement
- Good for dashboards and similar use cases

### Example Schema

```graphql
type Product {
  id: ID!
  name: String!
  createdAt: String!
}

type ProductPage {
  items: [Product!]!
  page: Int!
  limit: Int!
  hasMore: Boolean!
  totalItems: Int
  totalPages: Int
  nextPage: Int
  prevPage: Int
}

type Query {
  productsOffset(page: Int = 1, limit: Int = 20): ProductPage!
}
```

---

# 4. Simple List Query

Example:

```graphql
query {
  productsOffset(page: 2, limit: 5) {
    items {
      id
      name
      createdAt
    }
    page
    limit
    hasMore
    nextPage
    prevPage
  }
}
```

The client specifies:
- Page number
- Number of items per page
- Fields it wants

---

# 5. SQL for Simple Pagination

Example:

```sql
SELECT *
FROM products
ORDER BY created_at DESC, id DESC
LIMIT :limit
OFFSET (page - 1) * :limit;
```

### Example

For:

```text
page = 2
limit = 5
```

The database skips:

```text
(2 - 1) × 5 = 5
```

and returns the next 5 products.

---

# 6. `hasMore`

Instead of always calculating the exact total, the API can determine whether another page exists.

A common technique is:

```sql
LIMIT (limit + 1)
```

### Example

If the client asks for:

```text
limit = 5
```

the server queries:

```text
LIMIT 6
```

- If 6 records are returned → `hasMore = true`
- Return only the first 5 records.
- If 5 or fewer are returned → `hasMore = false`

### Key Idea

**Fetch one extra record to know whether another page exists.**

---

# 7. Simple List Pagination — Pros

- Easy to implement and understand.
- Easy to debug.
- Supports **jump-to-page**.
- Familiar for dashboards and admin tables.

---

# 8. Simple List Pagination — Cons

- Deep offsets can become slow.
- Inserts/deletes can cause duplicates or skipped records.
- Exact total counts can be expensive.
- Results may shift when data changes between page requests.

---

# 9. Relay Connection Pattern

The **Relay Connection pattern** is a common cursor-based pagination pattern in GraphQL.

Instead of returning a plain array, the API returns a **Connection object**.

A Connection contains:

- **Edges**
- **PageInfo**

### Basic Structure

```text
Connection
 ├── edges
 │    ├── node
 │    └── cursor
 │
 └── pageInfo
      ├── hasNextPage
      ├── hasPreviousPage
      ├── startCursor
      └── endCursor
```

---

# 10. Node

A **node** is the actual data item.

Example:

```graphql
type Product {
  id: ID!
  name: String!
  createdAt: String!
}
```

If the connection contains products, each `node` is a **Product**.

### Key Idea

**Node = Actual item**

---

# 11. Edge

An **edge** wraps a node and provides information about that node's position in the list.

Example:

```graphql
type ProductEdge {
  node: Product!
  cursor: String!
}
```

An edge contains:
- `node` → actual product
- `cursor` → cursor for that product

You can also add custom edge fields such as:
- Rank
- Score
- Relationship type

### Key Idea

**Edge = Node + Cursor + optional edge metadata**

---

# 12. PageInfo

`PageInfo` contains pagination metadata.

Example:

```graphql
type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

### Meaning

| Field | Meaning |
|---|---|
| `hasNextPage` | More records exist after the current page |
| `hasPreviousPage` | More records exist before the current page |
| `startCursor` | Cursor of the first item |
| `endCursor` | Cursor of the last item |

---

# 13. Product Connection

A complete connection can be defined as:

```graphql
type ProductConnection {
  edges: [ProductEdge!]!
  pageInfo: PageInfo!
  totalCount: Int
}
```

### Important

- **Node** = actual item
- **Edge** = node + cursor
- **Connection** = entire paginated list
- **PageInfo** = pagination metadata

`totalCount` can be included as an optional field.

---

# 14. Forward Cursor Pagination

To get the next page, use:

- `first` → number of items to fetch
- `after` → cursor after which to continue

### Example

```graphql
query {
  products(first: 3, after: "cursorToken") {
    edges {
      node {
        id
        name
        createdAt
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### Basic Flow

```text
First request
     ↓
Get products + endCursor
     ↓
Send endCursor as `after`
     ↓
Get next page
```

---

# 15. Backward Cursor Pagination

To move backward through pages, use:

- `last` → number of items to fetch
- `before` → cursor before which to fetch

### Example

```graphql
query {
  products(
    last: 5,
    before: "cursorFromStartOfCurrentPage"
  ) {
    edges {
      node {
        id
        name
      }
      cursor
    }
    pageInfo {
      hasPreviousPage
      startCursor
    }
  }
}
```

---

# 16. What is a Cursor?

A cursor is a **compact, opaque token**.

It usually contains encoded information about:
- Last seen key(s)
- Sorting
- Sometimes filters

Example payload before encoding:

```json
{
  "createdAt": 1695808800000,
  "id": 122,
  "sort": "createdAt.desc,id.desc",
  "filters": {
    "q": "phone"
  }
}
```

The client should treat the cursor as an opaque value.

---

# 17. Cursor → Keyset Query

The server can:

1. Decode the cursor.
2. Validate it.
3. Extract the last-seen values.
4. Use them in a keyset query.

For example:

```text
(created_at, id) < (cursor.createdAt, cursor.id)
```

This allows the database to continue from the previous position instead of skipping a large number of rows.

---

# 18. Relay Connection — Pros

- More consistent when data changes.
- **Index-friendly** when implemented with keyset pagination.
- Ideal for **infinite scrolling**.
- `edges` allow extra metadata such as rank or score.
- Supports forward and backward pagination.

---

# 19. Relay Connection — Cons

- No natural **jump-to-page**.
- Cursors must be encoded and validated.
- Arguments need guardrails.
- More complex than simple page-based pagination.

---

# 20. Simple List vs Relay Connection

| Feature | Simple List | Relay Connection |
|---|---|---|
| Pagination style | Offset/page-based | Cursor-based |
| Main arguments | `page`, `limit` | `first`, `after`, `last`, `before` |
| Response | Items + page info | Edges + PageInfo |
| Jump to page | Easy | Not naturally supported |
| Large/changing data | Less suitable | Better suited |
| Infinite scroll | Possible | Excellent fit |
| Implementation | Simpler | More complex |
| Cursor | No | Yes |

---

# 21. Important Terms

| Term | Meaning |
|---|---|
| **GraphQL Pagination** | Splitting GraphQL results into smaller pages |
| **Simple List** | Page/offset + limit pagination |
| **Connection** | Complete cursor-based paginated list |
| **Edge** | Wrapper containing a node and cursor |
| **Node** | Actual data item |
| **Cursor** | Opaque token representing a position |
| **PageInfo** | Pagination metadata |
| **`first`** | Number of items to fetch forward |
| **`after`** | Cursor after which to fetch |
| **`last`** | Number of items to fetch backward |
| **`before`** | Cursor before which to fetch |
| **`hasNextPage`** | Whether more items exist after current page |
| **`hasPreviousPage`** | Whether more items exist before current page |
| **`startCursor`** | Cursor of first item |
| **`endCursor`** | Cursor of last item |
