# Object Storage — Short Notes

## 1. What Are BLOBs?

**BLOB = Binary Large Object.** A BLOB is raw binary data, for example:
- Images (JPEG, PNG)
- Videos (MP4)
- Audio (MP3)
- PDFs
- ZIP files

Databases can store BLOBs using types such as `BLOB`, `BYTEA`, and `VARBINARY`.

```sql
CREATE TABLE users (
    id INT,
    profile_pic BLOB
);
```

Storing a user's data and image together may look convenient, but it can become inefficient at scale.

---

# 2. Why Is Storing BLOBs in a Database a Bad Idea?

Databases are optimized for structured data: rows, columns, indexes, queries, joins, and transactions. They are not optimized for large binary payloads.

### Main Problems

**Performance:** If an image is 5 MB and a user row is 1 KB, reading the row may also retrieve a large BLOB even when the image is not needed. This can increase latency, memory use, and query time.

**Backup and restore:** Backup size grows, restores take longer, and replication traffic increases.

**Scaling:** Large binary data increases storage cost, I/O, and replication traffic. Scaling databases vertically or through sharding can become expensive or complex.

**Tight coupling:** Databases are closely coupled to applications and are not intended to act as public media-delivery systems.

### Correct Pattern

```text
Image / Video → Object Storage
Metadata      → Database
```

---

# 3. Why Do We Need Object Storage?

We want storage that is:
- Cheap
- Scalable
- Durable
- Accessible over HTTP
- CDN-friendly
- Not tied to database constraints

**Object Storage is designed for these requirements.**

---

# 4. What Is Object Storage?

**Object Storage** stores data as objects. Each object contains:

1. **Data** — actual file/binary content
2. **Metadata** — key-value information about the object
3. **Object ID / Key** — identifier used to access it

Objects live inside **buckets** and are accessed using HTTP APIs.

Example:

```text
Bucket: user-uploads
Key: profile-pics/user.png
```

Examples:
- Amazon S3
- Google Cloud Storage
- Azure Blob Storage

---

# 5. Important Object Storage Terms

| Term | Meaning |
|---|---|
| **Object** | File data plus metadata and an identifier |
| **Bucket** | Container that holds objects |
| **Object Key** | Name/identifier used to locate an object |
| **Metadata** | Information describing an object |
| **Object ID** | Identifier for an object |

---

# 6. Upload Flow (PUT Object)

A typical upload flow:

```text
Client
  ↓
API Layer
(Auth + Validation)
  ↓
Metadata Service
  ↓
Object Storage
```

### Steps

**Step 1: Request comes in**

The client sends an upload request, for example:

```http
PUT /user-uploads/profile-pics/user.png
```

The API layer checks:
- Authentication (IAM / token)
- Whether the bucket exists
- Permissions
- File-size limits

**Step 2: Metadata entry is created**

A metadata record is first created with status `PENDING`, because the upload has not finished.

| Key | Size | Status | Locations |
|---|---:|---|---|
| img1 | 5 MB | PENDING | NULL |

**Step 3: Object is chunked**

A large object can be divided into chunks:

```text
20 MB Object
     ↓
 C1 | C2 | C3 | C4
```

Chunking supports parallel upload, retries on failure, and large-object uploads.

**Step 4:** Chunks are distributed to data nodes.

**Step 5:** Durability is applied using replication or erasure coding.

**Step 6:** Metadata is updated to `COMMITTED` / ready after the upload completes.

Only then does the client receive `HTTP 200 OK`.

---

# 7. Read Flow (GET Object)

```text
Client
  ↓
API Layer (Auth + Validation)
  ↓
Metadata Service
  ↓
Data Nodes
  ↓
Stream object to client
```

### Steps

1. Client sends a GET request.
2. Metadata service looks up the object key and data locations.
3. Required chunks are fetched, potentially in parallel.
4. Chunks are reassembled into the complete object.
5. The object is streamed to the client.

```text
C1 + C2 + C3 + C4
          ↓
     Full Object
          ↓
        Client
```

---

# 8. Delete Flow

Deleting an object can happen asynchronously.

```text
Client
  ↓ DELETE object
API Layer
  ↓
Metadata Service
  ↓
Status = DELETED
```

The actual data deletion may happen later. A **Garbage Collector** can remove stored chunks after the object has been marked deleted.

**Logical deletion can happen first; physical cleanup can happen later.**

---

# 9. Metadata Is the Brain

The **Metadata Service** keeps important information about each object:
- Object key
- Size
- Version
- ACL (Access Control List)
- Checksums
- Data locations

If metadata is unavailable, the system may be unable to locate or access an object even if its data still exists.

**Metadata tells the system what an object is and where its data is stored.**

---

# 10. Versioning

Versioning keeps track of different versions of an object.

```text
PUT img.png → v1
PUT img.png → v2
```

Metadata can identify:

```text
img.png → v2 (active)
             v1 (older)
```

Deleting an object may add a **delete marker**, depending on the storage system's versioning behavior.

### Why Versioning?
- Keep older versions.
- Recover from accidental overwrites.
- Help recover deleted content.

---

# 11. Multipart Upload (Large Files)

For large files, object storage can support **multipart upload**.

```text
Initiate Upload
      ↓
Upload Part 1
Upload Part 2
Upload Part 3
      ↓
Complete Upload
```

Different parts may be uploaded in parallel.

### Benefits
- **Resume:** retry or continue failed parts.
- **Parallelism:** upload multiple parts at the same time.
- **Reliability:** a failed part does not necessarily require restarting the entire file.

---

# 12. Core Characteristics of Object Storage

## 1. Immutability

Objects are generally treated as immutable. To change content, you typically upload a new object or a new version rather than editing bytes in place.

## 2. Flat Namespace

Object storage uses a flat namespace rather than real nested folders.

```text
/photos/2025/img.png
```

This can simply be a string key. Folders shown in a user interface are often a visual representation created using key prefixes.

## 3. Globally Unique Object Key

An object is identified using its bucket and object key.

```text
s3://user-uploads/profile-pics/user.png
```

---

# 13. Durability — Why Data Is Protected

Object storage aims to provide high durability using techniques such as:

## Replication

Multiple full copies of data are stored.

```text
Original Object
   ├── Copy 1
   ├── Copy 2
   └── Copy 3
```

This is simple but can be costly because it stores multiple full copies.

## Erasure Coding

Data is split into data blocks and additional parity blocks.

```text
Data → k data blocks + m parity blocks
```

Even if some blocks are lost, the original data can still be recovered, depending on the coding scheme.

The notes mention that Amazon S3 uses durability mechanisms designed for extremely high durability.

---

# 14. Security in Object Storage

## Encryption
- **At rest:** protects stored data (example: AES-256).
- **In transit:** protects data sent over the network (TLS).

## Access Control

Access can be controlled using:
- Bucket policies
- Object ACLs
- IAM roles

## Pre-Signed URLs

A **Pre-Signed URL** grants temporary access to upload or download an object without exposing long-term credentials.

---

# 15. Performance Characteristics

### Object Storage Is Not Best For
- Low-latency random writes
- Frequent updates to small pieces of data
- Database workloads

### Object Storage Is Good For
- Large objects
- Streaming
- Read-heavy workloads
- CDN integration

---

# 16. Common Use Cases

- Media hosting (video, audio, images)
- Backups and snapshots
- Logs and analytics
- Data lakes
- Machine-learning training datasets

---

# 17. Correct Database + Object Storage Design

Do not store large files directly in a database when object storage is more suitable. Split responsibilities:

```text
Client ↔ Server
           ├── Object Storage → File bytes
           └── Database       → Metadata
```

### Object Storage Stores
- Images
- Videos
- PDFs
- Large binary files

### Database Stores Metadata

Example:

```text
post
- id
- creatorId
- text
- linkToObjectStore
```

`linkToObjectStore` can be an S3 object key or a URL.

---

# 18. Direct Upload Using Pre-Signed URLs

A common design lets the client upload directly to object storage.

### Flow

1. Client asks the server to upload a file.
2. Server authenticates the user and requests a pre-signed URL from object storage.
3. Server sends the URL back to the client.
4. Client uploads the file directly to object storage.

```text
Client ── request ──> Server
Client <── signed URL ─ Server

Client ── file upload ──> Object Storage
```

### Why Is This Better?

The application server does not need to receive and forward all file bytes.

This reduces:
- Server bandwidth bottlenecks
- High memory usage
- Poor scalability

The notes mention Instagram, Netflix uploads, and Google Drive as examples of services using direct-to-storage upload patterns. With Amazon S3, the server can create a pre-signed URL and the browser can upload directly to S3.

---

# 19. Why Upload Files in Chunks?

Large files (for example, 5–10 GB) may not upload reliably in one request because network failures can occur.

Multipart upload:
1. Accepts the file in multiple parts.
2. Stores parts independently.
3. Combines the parts when the upload is completed.

This supports retries and more reliable large-file uploads.

---

# 20. Important Terms

| Term | Meaning |
|---|---|
| **BLOB** | Binary Large Object; raw binary data |
| **Object Storage** | Stores files as objects |
| **Object** | Data + metadata + object identifier |
| **Bucket** | Container for objects |
| **Object Key** | Name/identifier used to access an object |
| **Metadata** | Information about an object |
| **PUT Object** | Upload an object |
| **GET Object** | Retrieve an object |
| **Multipart Upload** | Upload a large object in separate parts |
| **Replication** | Store multiple copies of data |
| **Erasure Coding** | Use data and parity blocks to recover lost data |
| **Immutability** | Object content is not modified in place |
| **Flat Namespace** | Objects are identified by keys, not real nested folders |
| **Versioning** | Keep multiple versions of an object |
| **Delete Marker** | Marker indicating an object is deleted in a versioned system |
| **Garbage Collector** | Removes data that is no longer needed |
| **Pre-Signed URL** | Temporary URL granting permission to access an object |
| **IAM** | Identity and Access Management |
| **ACL** | Access Control List |
| **CDN** | Content Delivery Network |
