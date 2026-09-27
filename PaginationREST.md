# REST API Pagination — Short Notes

## 1. What is Pagination?

APIs often return large datasets such as all orders, products, or posts.

Returning everything in one response can cause:
- **Slow responses** — more time to fetch.
- **High bandwidth usage** — too much data is transferred.
- **Higher server cost** — more server load.

**Pagination = Splitting large results into smaller, manageable chunks (pages).**

Example:

`Page 1 → 20 products`

`Page 2 → 20 products`

`Page 3 → 20 products`

---

# 2. Why Pagination Matters

### Performance
- Faster responses
- Less memory usage

### User Experience
- Users are not overloaded with thousands of results at once.

### Cost
- Saves network bandwidth and API costs.

### Control
- Lets clients choose how much data to fetch.

---

# 3. Pagination in REST

The notes cover four approaches:

1. **Offset + Limit (Page-Based Pagination)**
2. **Cursor-Based Pagination (Opaque Token)**
3. **Keyset / Seek Pagination**
4. **Time-Window Pagination**

---

# 4. Offset + Limit Pagination

This is common, especially in SQL-based systems.

- `limit` → number of records to return
- `offset` → number of records to skip

### Example

```text
GET /products?limit=20&offset=40
```

This means:
- Skip 40 products.
- Return the next 20 products.

### Page-Based Style

```text
GET /products?page=3&limit=20
```

Formula:

```text
offset = (page - 1) × limit
       = (3 - 1) × 20
       = 40
```

So:

`Page 3 → offset 40 → limit 20`

---

# 5. SQL Query for Offset Pagination

```sql
SELECT id, name, price
FROM products
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 40;
```

The query:
- Sorts products.
- Skips the first 40.
- Returns the next 20.

---

# 6. Pagination Response

A useful REST response can include data and pagination information.

Example:

```json
{
  "data": [
    {"id": 1, "name": "Product 1"},
    {"id": 2, "name": "Product 2"}
  ],
  "pagination": {
    "type": "offset",
    "limit": 20,
    "offset": 40,
    "total_items": 1243,
    "total_pages": 63,
    "next": "/products?page=4&limit=20",
    "prev": "/products?page=2&limit=20"
  }
}
```

The client can easily request the next or previous page.

---

# 7. Offset Pagination — Pros

- Easy to implement and debug.
- Supports jump-to-page UI such as **"page 17"**.
- Uses familiar SQL `LIMIT` and `OFFSET`.
- Good for small or mostly-static lists.

---

# 8. Offset Pagination — Cons

### A. Deep Offsets Can Be Slow

Large offsets can be inefficient because the database may still scan/skip many rows.

Example:

```text
OFFSET 100000
```

### B. Inconsistent Results

If rows are inserted or deleted while paging, results can:
- Shift
- Duplicate
- Be missed

### C. Expensive Total Counts

Exact total counts can be expensive on huge tables.

---

# 9. Best Practices for Offset Pagination

- Keep page sizes small, around **20–100**.
- Avoid deep offsets.
- After around **10K+** rows, consider cursor-based pagination.
- Create indexes for `LIMIT`/`OFFSET` queries.
- Cache total counts or return **`has_more`** instead of an exact total when appropriate.

---

# 10. Cursor-Based Pagination

Cursor-based pagination uses a **cursor/token** instead of an offset.

The cursor represents a position in the dataset.

### Example

First request:

```text
GET /products?limit=20
```

Response contains:

```text
next_cursor=eyJjcmVhdGVkX2F0Ijox...
```

Next request:

```text
GET /products?limit=20&after=<cursor>
```

The server continues from the cursor position.

---

# 11. Opaque Cursor

A cursor should generally be treated as an **opaque token**.

The client should:
- Store it.
- Send it back.
- Not modify it.
- Not depend on its internal structure.

The server can encode information such as:
- Last `created_at`
- Last `id`
- Other pagination state

---

# 12. Cursor Pagination Best Practices

- Use **opaque/signed (HMAC) cursors** so clients cannot forge cursors.
- Include a **query hash** so the cursor remains tied to the same filters/sort order.
- Use **stable sorting**, such as `created_at` and `id`.

Stable sorting helps prevent:
- Duplicates
- Skipped records
- Inconsistent results

---

# 13. Cursor Pagination — Pros

- Works well with **concurrent inserts/deletes**.
- Fast when using an indexed position.
- Good for large or continuously changing datasets.
- Useful for infinite scrolling.

---

# 14. Cursor Pagination — Cons

- Requires encoding/decoding and signing.
- Harder to support **jump-to-page**.
- Cursor tokens are usually tied to a particular sort order and filter.
- Changing the query can make a cursor invalid.

---

# 15. Keyset / Seek Pagination

**Keyset pagination** uses actual column values as the position instead of an opaque cursor.

It is also called **seek pagination**.

Suppose data is sorted by:

```text
created_at DESC, id DESC
```

To get the next page, use the last record's values.

```sql
SELECT id, name, price, created_at
FROM products
WHERE (created_at, id) < (:last_created_at, :last_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

### Key Idea

Instead of:

**"Skip 1000 rows."**

Use:

**"Start after this specific record."**

---

# 16. Keyset vs Offset

| Feature | Offset | Keyset / Seek |
|---|---|---|
| Uses | `LIMIT + OFFSET` | Last-seen key values |
| Deep pages | Can become slow | Usually efficient |
| Inserts/deletes | Can cause shifts | More stable |
| Jump to page | Easy | Difficult |
| Large datasets | Less suitable | Well suited |
| Query complexity | Simpler | More complex |

---

# 17. Time-Window Pagination

Time-window pagination divides data using **time ranges**.

Useful for:
- Logs
- Analytics
- Time-based events

### Example

```text
GET /events?start=2025-09-01T00:00:00Z
&end=2025-09-01T01:00:00Z
&limit=1000
```

This requests events within a specific time window.

**Key idea:** Time-window pagination = Fetch data by time ranges.

---

# 18. Pagination Approach Comparison

| Approach | Main Idea | Best For |
|---|---|---|
| **Offset + Limit** | Skip rows and take a fixed number | Small/medium, mostly-static lists |
| **Cursor-Based** | Continue from an opaque token | Large/changing datasets |
| **Keyset / Seek** | Continue after last key values | Large indexed datasets |
| **Time-Window** | Divide results by time range | Logs and analytics |

---

# 19. Important Terms

| Term | Meaning |
|---|---|
| **Pagination** | Splitting large results into smaller pages |
| **Page** | One chunk of results |
| **Limit** | Number of records returned |
| **Offset** | Number of records skipped |
| **Cursor** | Token representing a position in the dataset |
| **Opaque Cursor** | Cursor whose internal value should not be modified by the client |
| **Keyset Pagination** | Pagination using last-seen column values |
| **Seek Pagination** | Another name for keyset pagination |
| **Time-Window Pagination** | Pagination using time ranges |
| **Deep Offset** | Very large offset that can become inefficient |
| **Stable Sort** | Consistent ordering using fields such as `created_at` and `id` |
