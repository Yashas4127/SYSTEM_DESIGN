# Consistent Hashing — Celebrity / Hot Key Problem

## 1. What Is the Celebrity (Hot Key) Problem?

A **hot key** is a key that receives an unusually large number of requests compared with other keys.

Example:
- Virat Kohli uploads a post.
- `post_101` receives 10 million read requests.
- Consistent hashing maps `post_101` to Server 2.
- All requests for that key are directed to the same server, which can become overloaded.

```text
10 million requests
        |
     post_101
        |
 Consistent Hashing
        |
     Server 2  ← overloaded
```

**Problem:** Even if requests for other keys are distributed evenly, one extremely popular key can overload the server responsible for it.

## 2. Solution: Replication

Instead of storing only one copy of the popular data, create multiple replicas on different servers.

```text
                 post_101
                     |
              Multiple Replicas
                     |
          +----------+----------+
          |          |          |
       Server 1   Server 2   Server 3
          |          |          |
         Copy       Copy       Copy
          |          |          |
         3.3M       3.3M       3.4M
```

A load balancer or request-routing layer distributes reads across the replicas.

For example, 10 million requests can be divided approximately as:
- Server 1: 3.3 million
- Server 2: 3.3 million
- Server 3: 3.4 million

The distribution is illustrative; actual traffic depends on the routing strategy and server capacity.

## 3. What Happens Step by Step?

1. **Detect:** Identify that `post_101` is receiving unusually high traffic.
2. **Replicate:** Create copies of the popular post on multiple servers.
3. **Track replicas:** Maintain a mapping of which servers hold the copies.
4. **Route reads:** Distribute incoming read requests across the available replicas.
5. **Scale down:** When traffic drops, remove unnecessary replicas.

## 4. Virtual Nodes vs Replication vs Load Balancing

| Technique | What it solves |
|---|---|
| Virtual nodes | Uneven distribution of keys across servers |
| Replication | Hot key problem — multiple copies of popular data |
| Load balancing | Distributes incoming requests across servers/replicas |

### Important Difference

- **Virtual nodes:** Help make the distribution of many different keys more balanced.
- **Replication:** Creates multiple copies of one popular key so that it does not depend on a single server.
- **Load balancing:** Decides how requests are shared among the servers that can serve the key.

**Remember:** Increasing virtual nodes does not, by itself, solve a hot key. All requests for the same key still map to the same logical key location unless the system adds a separate replication and request-routing strategy.

## 5. Important Consideration: Read and Write Consistency

Replication is especially useful for **read-heavy** content, such as celebrity posts.

However, when the original data changes, replicas may need to be updated or invalidated.

For example:
- A user edits or deletes `post_101`.
- The system must make sure stale copies are not served for too long.
- Replication strategies may use synchronous updates, asynchronous updates, cache invalidation, or versioning.

More replicas improve read capacity, but they introduce additional work to keep copies sufficiently up to date.

## 6. Core Idea

Instead of forcing millions of users to access one server for the same celebrity post, store copies of that post on multiple servers and distribute read requests among those copies.

**One-line revision:**
- Virtual nodes solve uneven key distribution.
- Replication solves hot keys.
- Load balancing distributes requests across replicas.
