# Bloom Filter — Short Notes

## 1. What is a Bloom Filter?

A **Bloom Filter** is a **probabilistic data structure** used to quickly check whether an item may exist in a set.

It gives two possible answers:

- **Definitely NOT present**
- **Maybe present**

### Important

A standard Bloom Filter can have **false positives**, but it does **not have false negatives** when used correctly.

```text
Query
  ↓
Bloom Filter
 ↙          ↘
NO          MAYBE
Definitely  Could be
not present present
```

---

# 2. Why Do We Need Bloom Filters?

Suppose a system has millions of URLs or database records.

Without a Bloom Filter:

```text
Request → Database lookup → Result
```

With a Bloom Filter:

```text
Request
   ↓
Bloom Filter
 ↙        ↘
NO       MAYBE
 ↓          ↓
Stop     Database
```

If the Bloom Filter says **definitely not present**, we can avoid an expensive database lookup.

### Main Benefits

- Fast membership checking
- Very low memory usage
- Reduces unnecessary database/cache lookups

---

# 3. Main Components

A Bloom Filter mainly contains:

1. **Bit Array**
2. **Hash Functions**

```text
             Hash 1 ─┐
Item ──────── Hash 2 ─┼→ Bit Array
             Hash 3 ─┘
```

---

# 4. Bit Array

A Bloom Filter uses an array of bits.

Initially:

```text
0 0 0 0 0 0 0 0 0 0
```

When an item is inserted, selected positions are changed to `1`.

Example:

```text
0 0 1 0 0 1 0 0 1 0
```

### Important

Insertion only changes:

```text
0 → 1
```

---

# 5. Hash Functions

A hash function converts an item into a bit position.

Example:

```text
Hash("apple") → 2
Hash("apple") → 5
Hash("apple") → 8
```

With 3 hash functions:

```text
apple
  ↓
2, 5, 8
```

The Bloom Filter sets:

```text
bit[2] = 1
bit[5] = 1
bit[8] = 1
```

---

# 6. Inserting an Item

Suppose the initial array is:

```text
0 0 0 0 0 0 0 0 0 0
```

Insert:

```text
apple
```

Assume hashes give:

```text
2, 5, 8
```

After insertion:

```text
0 0 1 0 0 1 0 0 1 0
```

---

# 7. Checking an Item

To check an item:

1. Apply the same hash functions.
2. Find the required bit positions.
3. Check those bits.

### If any required bit is `0`

```text
Definitely NOT present
```

### If all required bits are `1`

```text
Maybe present
```

---

# 8. Why "Maybe Present"?

Another item may have already set the same bits.

Example:

```text
apple → bits 2, 5, 8
mango → bits 2, 5, 8
```

If we query `mango`, all its bits are `1`.

The Bloom Filter says:

```text
Maybe present
```

But `mango` may never have been inserted.

This is a **false positive**.

---

# 9. False Positive

A **false positive** occurs when:

```text
Bloom Filter → Maybe present
Actual set   → Item is absent
```

### Example

```text
Actual data:
apple
banana

Query:
mango

Bloom Filter:
Maybe present

Actual result:
mango does not exist
```

---

# 10. False Negative

A **false negative** would mean:

```text
Bloom Filter → Definitely NOT
Actual set   → Item exists
```

A standard Bloom Filter does **not produce false negatives** if it is used correctly.

### Easy Memory Trick

```text
NO     → Definitely NO
MAYBE  → Maybe YES
```

---

# 11. Bloom Filter Operations

### Insert

```text
add(item)
```

### Query

```text
contains(item)
```

The query result is:

```text
Definitely NOT
OR
Maybe PRESENT
```

---

# 12. Time Complexity

Let:

- `k` = number of hash functions

### Insert

```text
O(k)
```

### Query

```text
O(k)
```

If `k` is a small constant:

```text
O(1)
```

So Bloom Filters provide very fast membership checks.

---

# 13. Space Efficiency

Bloom Filters use a **bit array**, so they use very little memory compared with storing complete objects.

### Key Idea

**Low memory + Fast lookup**

This is especially useful for very large datasets.

---

# 14. False Positive Rate

The false positive rate mainly depends on:

- Size of the bit array
- Number of inserted items
- Number of hash functions

If too many items are inserted:

```text
More items
    ↓
More bits become 1
    ↓
More collisions
    ↓
Higher false positive rate
```

---

# 15. Number of Hash Functions

The number of hash functions should be chosen carefully.

### Too Few

```text
Too few hashes
     ↓
More collisions
     ↓
Higher false positives
```

### Too Many

```text
More hash calculations
     ↓
More computation
```

There is an optimal number of hash functions for a given filter size and expected number of elements.

---

# 16. Bloom Filter Workflow

### Insert

```text
Item
 ↓
Multiple Hash Functions
 ↓
Set corresponding bits to 1
```

### Query

```text
Item
 ↓
Multiple Hash Functions
 ↓
Check corresponding bits
 ↙                 ↘
Any bit = 0       All bits = 1
 ↓                   ↓
Definitely NO     Maybe Present
```

---

# 17. Bloom Filter vs Hash Set

| Feature | Bloom Filter | Hash Set |
|---|---|---|
| Result | Maybe / Definitely No | Exact |
| False Positive | Possible | No |
| False Negative | No | No |
| Memory | Very low | Higher |
| Stores actual values | No | Yes |
| Lookup | Very fast | Very fast |
| Best use | Pre-check | Exact membership |

### Key Difference

**Bloom Filter saves memory but is probabilistic.**

---

# 18. Bloom Filter vs Database

| Feature | Bloom Filter | Database |
|---|---|---|
| Purpose | Fast membership pre-check | Store/retrieve actual data |
| Exact answer | No | Yes |
| False positives | Possible | Normally no |
| Storage | Very compact | Larger |
| Stores actual data | No | Yes |

### Important

**Bloom Filter does not replace the database.**

It helps reduce unnecessary database lookups.

---

# 19. Practical Architecture

```text
             Request
                ↓
          Bloom Filter
           ↙        ↘
      Definitely NO  Maybe
           ↓           ↓
         Reject     Database
                       ↓
                     Result
```

This is useful when database lookups are expensive.

---

# 20. Common Use Cases

Bloom Filters can be used for:

### Web Caching

Check whether data may exist in a cache.

### Databases

Avoid unnecessary disk/database lookups.

### Distributed Systems

Quick membership checks across large datasets.

### URL / Duplicate Detection

Check whether a URL or item may have been seen before.

### Large Blocklists

Quickly check whether an identifier may belong to a large set, followed by an exact lookup.

---

# 21. Limitations

- False positives are possible.
- Cannot return the actual stored value.
- Standard Bloom Filters do not support normal deletion.
- False positive rate increases as the filter becomes full.
- Filter size and hash count need to be selected appropriately.

---

# 22. Why Deletion is Difficult

Suppose:

```text
apple → bit 2
banana → bit 2
```

Both items use bit 2.

If we delete `apple` and set bit 2 to `0`:

```text
bit 2 → 0
```

we may incorrectly say that `banana` is absent.

Therefore, a standard Bloom Filter does not support safe deletion.

---

# 23. Counting Bloom Filter

A **Counting Bloom Filter** replaces bits with counters.

Normal Bloom Filter:

```text
0 1 0 1 1
```

Counting Bloom Filter:

```text
0 2 0 1 3
```

Insertion:

```text
counter++
```

Deletion:

```text
counter--
```

This allows deletion while tracking how many inserted items use a position.

---

# 24. Simple Example

Suppose:

```text
Bit Array = 10 bits
Hash Functions = 3
```

Insert:

```text
cat
```

Hashes:

```text
2, 4, 7
```

Array becomes:

```text
0 0 1 0 1 0 0 1 0 0
```

Now query:

```text
dog
```

Suppose hashes are:

```text
1, 4, 7
```

Check:

```text
bit[1] = 0
bit[4] = 1
bit[7] = 1
```

Because one required bit is `0`:

```text
dog → Definitely NOT present
```

---

# 25. Important Terms

| Term | Meaning |
|---|---|
| **Bloom Filter** | Probabilistic membership data structure |
| **Bit Array** | Array of bits used by the filter |
| **Hash Function** | Maps an item to a bit position |
| **False Positive** | Says "maybe present" when item is absent |
| **False Negative** | Incorrect "not present" result; standard Bloom Filters avoid this |
| **Membership Test** | Checking whether an item may belong to a set |
| **k** | Number of hash functions |
| **Counting Bloom Filter** | Variant that supports deletion |
| **Pre-check** | Fast check before an expensive operation |
| **Collision** | Different items map to overlapping bit positions |
