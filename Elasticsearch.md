# Elasticsearch — Short Notes

## 1. What is Elasticsearch?

**Elasticsearch** is a **distributed search and analytics engine**.

It is commonly used for:
- Fast text search
- Filtering
- Sorting
- Aggregations
- Log analysis
- Monitoring

### Simple Idea

```text
Application
     ↓
Elasticsearch
     ↓
Fast Search Results
```

### Common Use Cases

- Website search
- Product search
- Log analysis
- Monitoring
- Autocomplete

---

# 2. Why Do We Need Elasticsearch?

Suppose a system has millions of products.

```text
10 million products
        ↓
Search: "wireless headphones"
        ↓
Fast matching + ranking
```

Elasticsearch is designed for this type of search workload.

### Key Idea

**Database → Store/manage application data**

**Elasticsearch → Search and analyze data quickly**

Elasticsearch can work alongside a database.

---

# 3. Cluster

A **cluster** is a collection of Elasticsearch nodes working together.

```text
        Elasticsearch Cluster
       /          |          \
   Node 1      Node 2      Node 3
```

A cluster provides:
- Distributed storage
- Search across nodes
- High availability
- Scalability

---

# 4. Node

A **node** is a running Elasticsearch instance.

```text
Cluster
 ├── Node 1
 ├── Node 2
 └── Node 3
```

Each node can store data and participate in searches.

---

# 5. Index

An **Index** is a collection of related documents.

Example:

```text
products index
    ↓
Product 1
Product 2
Product 3
```

An application may have indexes such as:

```text
products
users
orders
```

---

# 6. Document

A **Document** is the basic unit of data stored in Elasticsearch.

Documents are represented as JSON.

```json
{
  "id": 101,
  "name": "Wireless Headphones",
  "price": 2499,
  "category": "electronics"
}
```

**Document = One JSON data object**

---

# 7. Field

A **field** is a property inside a document.

Example:

```json
{
  "name": "Laptop",
  "price": 50000
}
```

Fields:

```text
name
price
```

---

# 8. Cluster → Node → Index → Document → Field

```text
Cluster
   ↓
 Nodes
   ↓
Indexes
   ↓
Documents
   ↓
Fields
```

| Term | Meaning |
|---|---|
| **Cluster** | Group of Elasticsearch nodes |
| **Node** | Elasticsearch server/instance |
| **Index** | Collection of related documents |
| **Document** | JSON object |
| **Field** | Property inside a document |

---

# 9. Inverted Index

The **Inverted Index** is one of the most important Elasticsearch concepts.

It is a data structure designed for fast text search.

Suppose:

```text
Document 1 → "apple phone"
Document 2 → "apple laptop"
Document 3 → "gaming laptop"
```

Inverted index:

```text
apple  → Document 1, Document 2
phone  → Document 1
laptop → Document 2, Document 3
gaming → Document 3
```

### Key Idea

**Term → Documents containing the term**

---

# 10. How Search Works

```text
User Query
    ↓
Elasticsearch
    ↓
Analyze Query
    ↓
Search Inverted Index
    ↓
Find Matching Documents
    ↓
Rank Results
    ↓
Return Results
```

---

# 11. Full-Text Search

Elasticsearch supports **full-text search**.

Example:

```text
"wireless bluetooth headphones"
```

Elasticsearch can analyze the query and find relevant documents.

---

# 12. Relevance

When multiple documents match, Elasticsearch can assign a **relevance score**.

Example:

```text
Query: "wireless headphones"

Document A → Score 10.5
Document B → Score 7.2
Document C → Score 2.8
```

Higher score generally means a more relevant result.

---

# 13. Mapping

A **Mapping** defines how fields in documents are stored and indexed.

Common field types include:

- `text`
- `keyword`
- `integer`
- `long`
- `float`
- `boolean`
- `date`

Example:

```json
{
  "properties": {
    "name": {
      "type": "text"
    },
    "category": {
      "type": "keyword"
    },
    "price": {
      "type": "float"
    }
  }
}
```

**Mapping = Defines field types and indexing behavior**

---

# 14. Text vs Keyword

## `text`

Used for **full-text search**.

Example:

```text
"wireless headphones"
```

The value can be analyzed into searchable terms.

## `keyword`

Used for:
- Exact matching
- Filtering
- Sorting
- Aggregations

Example:

```text
category = "electronics"
```

### Easy Memory Trick

**text → Search inside text**

**keyword → Exact value**

---

# 15. Sharding

Large indexes can be divided into smaller pieces called **shards**.

```text
Index
  ↓
┌────────┬────────┬────────┐
│Shard 1 │Shard 2 │Shard 3 │
└────────┴────────┴────────┘
```

Each shard contains part of the index's data.

### Why Sharding?

- Distribute data across nodes
- Handle large datasets
- Improve scalability
- Allow parallel processing

---

# 16. Primary Shard

A **primary shard** contains the original copy of a portion of an index.

```text
Index
 ├── Primary Shard 1
 ├── Primary Shard 2
 └── Primary Shard 3
```

Documents are distributed among the primary shards.

---

# 17. Replica Shard

A **Replica** is a copy of a primary shard.

```text
Primary Shard 1
       ↓
Replica Shard 1
```

### Why Replicas?

- High availability
- Fault tolerance
- Additional search capacity

If a node containing a primary shard fails, a replica can help keep data available.

---

# 18. Primary vs Replica

| Feature | Primary Shard | Replica Shard |
|---|---|---|
| Main copy | Yes | Copy |
| Stores data | Yes | Yes |
| Fault tolerance | Original data | Backup/copy |
| Search | Yes | Yes |
| Main purpose | Store partition | Availability + extra search capacity |

---

# 19. Distributed Search

If an index has multiple shards, a search can work like:

```text
Client
  ↓
Elasticsearch
  ↓
┌─────────┬─────────┬─────────┐
│ Shard 1 │ Shard 2 │ Shard 3 │
└─────────┴─────────┴─────────┘
     ↓        ↓         ↓
   Results  Results   Results
      \        |        /
       \       |       /
        Combine Results
              ↓
           Response
```

Elasticsearch searches relevant shards and combines the results.

---

# 20. CRUD Operations

### Create / Index

```text
POST /products/_doc/1
```

```json
{
  "name": "Laptop",
  "price": 50000
}
```

### Read

```text
GET /products/_doc/1
```

### Update

```text
POST /products/_update/1
```

### Delete

```text
DELETE /products/_doc/1
```

---

# 21. Search Query

Example:

```text
GET /products/_search
```

With a query:

```json
{
  "query": {
    "match": {
      "name": "wireless headphones"
    }
  }
}
```

This searches the `name` field for relevant matches.

---

# 22. Filtering

Filtering is used when we need exact conditions.

Example:

```text
category = electronics
price < 50000
```

### Search vs Filter

**Search → Relevance matters**

**Filter → Exact condition matters**

---

# 23. Aggregations

**Aggregations** are used for analytics and summaries.

Example:

```text
Products by category
```

Result:

```text
Electronics → 1200
Clothing    → 850
Books       → 600
```

Other examples:
- Average price
- Maximum price
- Minimum price
- Count
- Group by category

### Key Idea

**Search → Find documents**

**Aggregation → Analyze documents**

---

# 24. Elasticsearch Use Cases

### E-commerce Search

```text
Search products
Filter by price/category
Sort by relevance
```

### Log Analytics

```text
Application Logs
      ↓
Elasticsearch
      ↓
Search + Analyze
```

### Autocomplete

```text
lap...
 ↓
Laptop
Laptop Stand
Laptop Bag
```

### Monitoring

Search and analyze application/system events.

---

# 25. Elasticsearch vs SQL Database

| Feature | Elasticsearch | SQL Database |
|---|---|---|
| Main purpose | Search & analytics | Transactions & structured data |
| Full-text search | Excellent | Available but different |
| Data format | JSON documents | Tables/rows |
| Schema | Mapping | Schema |
| Scaling | Distributed | Depends on database |
| Transactions | Not its main strength | Strong |
| Joins | Limited compared with SQL | Strong |
| Best for | Search, logs, analytics | Business transactions |

### Important

Elasticsearch is **not a direct replacement for a transactional SQL database**.

Many systems use both:

```text
Application
   ├── SQL Database → Source of truth
   └── Elasticsearch → Search
```

---

# 26. Simple Architecture

```text
                Application
                 /       \
                /         \
               ↓           ↓
        SQL Database   Elasticsearch
        Source of      Search &
         truth         Analytics
```

The database can store authoritative application data while Elasticsearch provides fast search and analytics.

---

# 27. Important Terms

| Term | Meaning |
|---|---|
| **Elasticsearch** | Distributed search and analytics engine |
| **Cluster** | Group of Elasticsearch nodes |
| **Node** | Running Elasticsearch instance |
| **Index** | Collection of documents |
| **Document** | JSON object stored in an index |
| **Field** | Property inside a document |
| **Mapping** | Defines field types and indexing behavior |
| **Inverted Index** | Data structure for fast text search |
| **Shard** | Partition of an index |
| **Primary Shard** | Main shard containing a portion of data |
| **Replica** | Copy of a primary shard |
| **Text** | Field type for analyzed full-text search |
| **Keyword** | Field type for exact values |
| **Relevance Score** | Score representing how well a document matches |
| **Filter** | Exact condition used to restrict results |
| **Aggregation** | Operation used to summarize/analyze data |
