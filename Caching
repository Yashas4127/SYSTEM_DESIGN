# Caching — Short Notes for Revision

## 1. What is Caching?

**Caching** means storing frequently accessed data in a **temporary, fast storage area** so that it can be retrieved quickly instead of repeatedly fetching it from the database.

### Basic Flow
`Client → Server → Cache / Database`

- **Cache:** Temporary storage, fast, usually smaller and more expensive.
- **Database:** Permanent storage, generally disk-based and slower than cache.
- Common cache examples mentioned in the notes: **Redis, Memcached**.

### Why use caching?
- Faster response time
- Reduces database load
- Improves application performance

---

# 2. Cache Invalidation

**Cache invalidation** means deciding **how and when cached data should be updated or removed** when the original data changes.

Example: A user's profile picture is stored in cache. If the picture changes, the old cached value must eventually be replaced.

## A. TTL — Time-to-Live / Expiry-Based

A cached item is given an **expiry time**.

- Set an expiry time.
- After expiry, the cached data becomes stale/invalid.
- Simple to implement.
- Data can remain stale until the TTL expires.

**Use cases:** Product catalogs, profile pictures, and similar data where slightly old data is acceptable.

---

# 3. Cache Write Policies

Write policies decide **when data is written to the cache and database**.

## A. Write-Through

`Client → Server → Cache + Database`

- Data is written to **cache and database at the same time**.
- Cache stays consistent with the database.
- Writes are slower because both must be updated.
- Useful where consistency is important.

**Easy memory trick:**  
**Write-Through = Write to both immediately**

---

## B. Write-Around

`Client → Server → Database`

- Data is written to the **database only**.
- Cache is not updated during the write.
- Cache gets updated later when the data is read/needed.
- Helps avoid putting rarely-read data into cache.

**Easy memory trick:**  
**Write-Around = Write around the cache**

---

## C. Write-Back / Write-Behind

`Client → Server → Cache → Database (later)`

- Data is written to the **cache first**.
- Database is updated **asynchronously/later**.
- Very fast writes.
- Risk: if the cache fails before syncing, data may be lost.
- Useful for high-write systems and cases where fast writes are important.

**Easy memory trick:**  
**Write-Back = Write cache first, DB later**

---

# 4. Cache Eviction

**Cache eviction** means deciding **which data to remove when the cache becomes full**.

## A. FIFO — First-In, First-Out

- Removes the item that entered the cache **first**.
- Does not consider how frequently or recently the item was used.

**Example:**  
If items enter as `A → B → C → D`, then **A** is removed first.

**Memory trick:**  
**FIFO = First in, first out**

---

## B. LRU — Least Recently Used

- Removes the item that has **not been used for the longest time**.
- Considers the time of the last access.

**Example:**  
If A was last used 2 min ago and B was last used 20 min ago, **B** is removed first.

**Memory trick:**  
**LRU = Least recently used**

---

## C. LFU — Least Frequently Used

- Removes the item that has been accessed **the fewest number of times**.
- Considers access frequency.

**Example:**  
If A is used 10 times and B is used 2 times, **B** is removed first.

**Memory trick:**  
**LFU = Least frequently used**

---

# 5. FIFO vs LRU vs LFU

| Policy | Removes based on | Simple idea |
|---|---|---|
| **FIFO** | Entry order | Oldest inserted item |
| **LRU** | Recent usage | Least recently accessed |
| **LFU** | Usage count | Least frequently accessed |

### Quick Revision
- **FIFO → When did it enter?**
- **LRU → When was it last used?**
- **LFU → How many times was it used?**

---

# 6. CDN — Content Delivery Network

A **CDN** is a network of distributed servers that delivers content to users from a **nearby server**.

It can cache and deliver content such as:
- Images
- Videos
- CSS
- JavaScript
- API responses

### Basic idea

`User → Nearby CDN → Origin Server (if content is not in CDN)`

For example, a user in **India** can receive cached content from a nearby CDN location instead of requesting it directly from a server in the **USA**.

### Benefits
- Lower latency
- Faster content delivery
- Reduces load on the origin server
- Useful for globally distributed users

---

# 7. Client-Side Caching

**Client-side caching** means caching data on the **client device** rather than on the server.

Examples:
- Web browser / HTTP caching
- LocalStorage
- Mobile app cache

### Benefits
- Faster access
- Reduced network requests
- Can support offline functionality
- Reduces latency

### Examples

**Browser / HTTP Cache**
- Stores resources such as images, CSS, and JavaScript.

**LocalStorage**
- Stores data directly in the browser.

**App / Mobile Cache**
- Stores frequently needed data on a mobile/desktop application.

