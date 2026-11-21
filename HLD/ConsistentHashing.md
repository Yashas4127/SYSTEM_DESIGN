# Consistent Hashing — Complete Notes
*Based on Chapter 5 of **System Design Interview by Alex Xu**, with an additional section on the Celebrity / Hot Key Problem.*

## 1. Why Do We Need Consistent Hashing?

In a distributed system, data is stored across multiple servers. A common way to decide where data belongs is to hash a key and use the result to select a server.

A basic approach is:

```text
server = hash(key) % number_of_servers
```

**Problem:** When a server is added or removed, the number of servers changes. The modulo result for many keys changes, so a large amount of data must be moved.

Consistent hashing reduces the number of keys that need to be remapped when the number of servers changes.

## 2. What Is Consistent Hashing?

**Consistent hashing** is a hashing technique in which servers and keys are mapped to a circular hash space called a **hash ring**.

When a key is mapped to the ring, move clockwise until the first server is found. That server owns the key.

**Main goal:** Minimize data movement when servers are added or removed.

## 3. Hash Space and Hash Ring

A hash function maps an input to a number in a fixed range.

For example, if a hash function produces values from `0` to `2^160 - 1`, that range is the **hash space**.

By connecting the two ends of the range, we get a circle called the **hash ring**.

```text
       0 ----------------------> 2^160 - 1
        \                       /
         \_____ HASH RING _____/
```

The ring is circular: moving clockwise past the maximum value wraps around to the minimum value.

## 4. Mapping Servers to the Ring

Use the same hash function to map each server onto the ring. A server can be hashed using its name, IP address, or another stable identifier.

```text
                 Server A
                    ●
             .-------------.
          ● /               \ ●
     Server D                 Server B
            \               /
             '------●------'
                 Server C
```

Each server gets a position on the ring.

## 5. Mapping Keys to the Ring

Keys are also hashed using the same hash function.

Example keys:
- `user_101`
- `user_202`
- `post_101`

Each key gets a position on the ring, just like a server.

**Important:** The key and server identifiers are hashed into the same hash space.

## 6. How Is a Server Selected?

After hashing a key:
1. Find the key's position on the ring.
2. Move clockwise.
3. Select the first server encountered.
4. Store or retrieve the key from that server.

```text
Key position ──clockwise──> First server = owner of key
```

This is often called the **clockwise successor rule**.

## 7. Adding a Server

When a new server is added, only keys in the range that now belongs to the new server need to move.

Example:
- Before: `key_1` belongs to Server A.
- Add Server D between the key and Server A in clockwise order.
- `key_1` now belongs to Server D.
- Keys assigned to other servers remain where they are.

**Benefit:** Only a fraction of keys are redistributed, rather than remapping almost every key.

## 8. Removing a Server

When a server is removed:
- Its keys move to the next server clockwise.
- Keys owned by other servers remain unchanged.

Example:
- Server B is removed.
- Keys previously owned by Server B move to its clockwise successor.
- Other keys do not need to move.

## 9. Problems with Basic Consistent Hashing

The basic approach has two important issues.

### A. Unequal partition sizes

A **partition** is the section of the hash ring between two neighboring server positions.

If server positions are unevenly distributed, some servers may own large sections while others own small sections.

Result:
- Some servers store much more data.
- Some servers store very little data.
- Server load becomes unbalanced.

### B. Non-uniform key distribution

Even if server positions look reasonable, hashed keys may cluster in certain ring regions.

Result:
- One server may receive most of the keys.
- Other servers may receive very few or no keys.

## 10. Virtual Nodes

A **virtual node ( vnode )** is a logical position on the ring that represents a physical server.

Instead of placing each physical server at only one position, place it at multiple positions.

Example:

```text
Physical Server A → A_0, A_1, A_2, A_3
Physical Server B → B_0, B_1, B_2, B_3
Physical Server C → C_0, C_1, C_2, C_3
```

All virtual nodes belonging to Server A point to the same physical machine.

### How lookup works with virtual nodes

1. Hash the key onto the ring.
2. Move clockwise to the first virtual node.
3. Find which physical server owns that virtual node.
4. Send the request to that physical server.

### Why do virtual nodes help?

- Each physical server owns multiple smaller ring partitions.
- The partitions are spread around the ring.
- Key distribution becomes more balanced.
- Adding or removing a physical server affects its virtual-node ranges.

### Virtual nodes and standard deviation

As the number of virtual nodes increases, the distribution of keys generally becomes more balanced and the standard deviation of load tends to decrease.

**Standard deviation** measures how far values are spread from their average.

- Lower standard deviation → server loads are closer to the average.
- Higher standard deviation → server loads vary more.

**Trade-off:** More virtual nodes improve balance but require more metadata and management.

### Heterogeneous servers

Servers may have different capacities. A more powerful server can be assigned more virtual nodes, allowing it to own a larger share of the ring.

## 11. Finding Affected Keys

### When a server is added

The new server takes over the key range between its position and the previous server encountered when moving anticlockwise (equivalently, the range whose clockwise successor changes to the new server).

Only keys in that range need to move.

### When a server is removed

The removed server's range is assigned to its clockwise successor. Only keys in the removed server's range need to move.

**Core idea:** Identify the changed ring interval; do not redistribute all keys.

## 12. Consistent Hashing and Load Balancing

Consistent hashing can be used to distribute keys or requests across servers while limiting remapping during membership changes.

However, consistent hashing does not guarantee perfectly equal load. Virtual nodes improve distribution, and capacity-aware vnode counts can help when servers have different capabilities.

## 13. Celebrity Problem / Hot Key Problem — Extra Notes

A **hot key** is a particular key that receives far more requests than other keys.

Example:
- Virat Kohli uploads a post.
- `post_101` receives 10 million read requests.
- Consistent hashing maps `post_101` to Server 2.
- Every request for that same key is routed to Server 2.
- Server 2 becomes overloaded even if the other servers are mostly idle.

```text
10 million requests
        |
     post_101
        |
 Consistent Hashing
        |
     Server 2  ← hot / overloaded
```

### Solution: Replication of popular data

For a read-heavy hot key, create multiple copies (replicas) of the popular data on different servers. Maintain a replica mapping and route reads across those copies.

```text
                  post_101
                      |
              Multiple replicas
                      |
          +-----------+-----------+
          |           |           |
       Server 1    Server 2    Server 3
          |           |           |
         Copy        Copy        Copy
          |           |           |
        Reads       Reads       Reads
```

If 10 million requests are shared approximately evenly, each of three replicas may handle around 3.3 million requests. Actual distribution depends on capacity and routing.

### Step-by-step

1. Detect that a key is receiving unusually high traffic.
2. Create replicas of the popular data on multiple servers.
3. Maintain a mapping of which servers contain the replicas.
4. Distribute incoming read requests across the replicas.
5. Reduce the number of replicas when traffic falls.

### Virtual nodes vs replication vs load balancing

| Technique | Main purpose |
|---|---|
| Virtual nodes | Reduce uneven distribution of keys/partitions across servers |
| Replication | Keep multiple copies of popular data; helps relieve a hot key |
| Load balancing / request routing | Spread requests across the available replicas |

**Important distinction:** More virtual nodes alone do not split one hot key across physical servers. A given key still maps to one logical location under the ordinary ring lookup. To spread requests for that same popular item, add replication and a routing strategy that selects among its replicas.

### Read/write consistency

Replication is especially useful for read-heavy celebrity content. When the original post is edited or deleted, replicas must be updated or invalidated so that stale data is not served indefinitely.

Possible approaches include:
- Synchronous replica updates
- Asynchronous replica updates
- Cache invalidation
- Versioning / timestamps

More replicas increase read capacity but add coordination and consistency work.

### A related shard-level approach

A system may isolate a celebrity's data on a dedicated shard, or partition that celebrity's data further, so that the celebrity's traffic does not overload a shard shared with unrelated users. This is a different mitigation from copying one hot item to multiple read replicas.

## 14. Advantages of Consistent Hashing

1. **Less data movement:** Adding or removing a server remaps only a portion of keys.
2. **Horizontal scalability:** Servers can be added to increase capacity.
3. **Better load distribution:** Virtual nodes improve balance across servers.
4. **Flexible capacity allocation:** More capable servers can receive more virtual nodes.
5. **Useful for distributed systems:** Helps partition data and distribute cache ownership.

## 15. Limitations and Trade-offs

- Basic consistent hashing can still produce uneven partitions.
- Virtual nodes require extra metadata and management.
- More virtual nodes improve balance but increase overhead.
- Consistent hashing alone does not solve every hot-key problem.
- Replication introduces consistency and update coordination challenges.

## 16. Where Is Consistent Hashing Used?

Examples discussed in system design literature include:
- Amazon Dynamo — data partitioning
- Apache Cassandra — partitioning across a cluster
- Discord — distributed infrastructure
- Akamai — content delivery network
- Maglev — network load balancing

## 17. Key Terms

| Term | Meaning |
|---|---|
| Hash function | Maps an input to a number |
| Hash space | Range of possible hash values |
| Hash ring | Circular representation of the hash space |
| Node / server | Physical machine in the distributed system |
| Key | Identifier for the data being stored or requested |
| Partition | Ring range between neighboring server positions |
| Successor | First server found moving clockwise |
| Virtual node | Logical ring position representing a physical server |
| Standard deviation | Measure of spread around the average |
| Hot key | Key receiving unusually high traffic |
| Replica | Additional copy of data |
| Load balancing | Distributing requests across servers |

## 18. Quick Concept Comparison

| Concept | Remember |
|---|---|
| Traditional hashing with modulo | Adding/removing a server can remap many keys |
| Consistent hashing | Only a fraction of keys need to move |
| Hash ring | Circular hash space where keys and servers are placed |
| Clockwise lookup | First server clockwise owns the key |
| Virtual nodes | Multiple logical positions per physical server |
| More virtual nodes | Usually lower load variation, but more overhead |
| Replication | Multiple copies of data |
| Hot key handling | Replicate popular data and route reads across replicas |

## 19. Final Core Idea

Consistent hashing distributes keys across servers while minimizing data movement when servers join or leave. Virtual nodes improve the balance of key distribution. For a single extremely popular key, use replication and request routing to spread its read traffic across multiple copies.
