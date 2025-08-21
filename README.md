# System Design Basics Notes
## Functional vs Non-Functional Requirements
## 🔹 Functional Requirements
### ✅ Definition:
Specify **what** a system should do — its core **features and behavior**.
### 🎯 Purpose:
Define **functions**, **interactions**, and **rules** of the system.
### 📌 Examples:
- User can log in using email and password
- Admin can delete a user account
- The system sends a confirmation email after registration
- Search functionality filters products by category
### 🔁 Characteristics:
- Use cases or user stories
- Typically documented using **"shall"** statements
- Derived from business requirements
## 🔸 Non-Functional Requirements
### ✅ Definition:
Specify **how** the system performs — the **qualities or constraints**.
### 🎯 Purpose:
Ensure performance, usability, reliability, scalability, and compliance.
### 📌 Examples:
- The system should respond within 200ms
- The application must handle 10,000 concurrent users
- Availability of 99.99% uptime
- Data must be encrypted at rest and in transit
### 🔁 Characteristics:
- Also known as **Quality Attributes**
- Often measurable
- Not related to specific business functions
## 🔄 Comparison Table
| Aspect                  | Functional Requirements                | Non-Functional Requirements               |
|------------------------|----------------------------------------|-------------------------------------------|
| **Focus**              | Features and capabilities              | Performance, quality, constraints         |
| **Defines**            | What the system should do              | How the system should behave              |
| **Validation**         | Verified through test cases/use cases  | Verified via metrics and benchmarks       |
| **Examples**           | Login, Registration, Search            | Latency, Security, Availability           |
| **Driven by**          | Business needs                         | Technical and operational needs           |
## 📝 Summary
- Functional = *What it does*
- Non-Functional = *How it does it*
Both are **critical** for successful system design and delivery.
---
## Key Characteristics of Distributed Systems
### 1. Scalability
**Definition**: Ability of a system to handle growing workload gracefully.
#### Types:
- **Horizontal Scaling**: Add more servers.
  - *Example*: Cassandra, MongoDB
- **Vertical Scaling**: Add resources (CPU/RAM) to existing server.
  - *Example*: MySQL
> **Tip**: Horizontal scaling supports dynamic growth better.
### 2. Reliability
**Definition**: The probability that a system fails within a given time.
- Ensures system continues functioning even when some components fail.
- Achieved through **redundancy**.
**Example**: In an e-commerce platform like Amazon, a user’s cart must persist even if a server crashes.
### 3. Availability
**Definition**: System uptime during a specific period.
- High availability ≠ High reliability.
- Achievable via rapid failover and repair mechanisms.
**Example**: An aircraft with low downtime is *available*. One that survives all weather conditions is *reliable*.
> A system can be available but not reliable.
### 4. Efficiency
**Measures**:
- **Latency**: Time to receive first item.
- **Throughput**: Items delivered per second.
> Consider:
- Network load
- Hardware/software heterogeneity
- Topology variation
### 5. Serviceability / Manageability
**Definition**: Ease of diagnosing, maintaining, and repairing the system.
- Early fault detection reduces downtime.
- Auto-diagnosing systems improve maintainability.
**Example**: A server automatically contacting support on fault detection.
---
## Load Balancing
### What is Load Balancing?
Distributes traffic across multiple servers to improve **availability** and **responsiveness**.
### Placement:
- Between user and web servers
- Between web servers and app/cache layers
- Between internal layers and DBs
### Benefits
- Faster service and lower latency
- High throughput and reduced downtime
- Predictive insights for bottlenecks
- Reduced stress per server
### Load Balancing Algorithms
#### 1. Least Connection
- Chooses server with fewest active connections.
- *Use case*: Long-lived connections.
#### 2. Least Response Time
- Picks server with fewest connections *and* lowest response time.
#### 3. Least Bandwidth
- Chooses server with lowest current traffic (Mbps).
#### 4. Round Robin
- Cycles through servers evenly.
- *Use case*: Equal server specs, short-lived requests.
#### 5. Weighted Round Robin
- Weights servers by capacity.
- Higher weight = more requests.
#### 6. IP Hash
- Uses hash of client IP to pick server.
- Ensures client always hits the same server.
  
### Redundant Load Balancers
> A single LB is a potential **SPOF** (Single Point of Failure).
- Use **dual LBs** in a cluster to monitor and replace each other on failure.
---
## Caching
### What is Caching?
Caches store frequently accessed data in a fast-access layer to reduce latency and load on backend systems.
> **Principle**: Locality of reference — recently accessed data is likely to be accessed again.
### Why Use Caching?
- Reduces response time
- Offloads backend services
- Makes high-demand requirements feasible
### Application Server Cache
- Cache is local to the request-handling node.
- Stored in **memory** (fastest) or **disk** (slower, but faster than network).
- Works best in single-node environments.
-  **Challenge** in multi-node: cache inconsistency due to random request routing.
### Global / Distributed Cache
Used to overcome cache misses in distributed setups:
- Shared cache among nodes (e.g., Redis, Memcached)
### Content Delivery Network (CDN)
Used for static media (images, videos, etc.).
**How it works**:
1. Request hits CDN.
2. If content available → served.
3. Else → fetch from origin, cache, and serve.
> **Tip**: Serve media from `static.domain.com` for future CDN migration ease.
### Cache Invalidation Strategies
| Strategy            | Description                                                                 | Pros                       | Cons                            |
|---------------------|-----------------------------------------------------------------------------|-----------------------------|----------------------------------|
| **Write-through**   | Write to cache **and** DB at same time                                       | Strong consistency          | Higher write latency             |
| **Write-around**    | Write directly to DB, bypass cache                                           | Avoids cache pollution      | Potential read misses            |
| **Write-back**      | Write to cache first, write to DB asynchronously                             | Fast writes                 | Risk of data loss on crash       |
### Cache Eviction Policies
| Policy       | Description                                              |
|--------------|----------------------------------------------------------|
| FIFO         | Removes oldest added items first                         |
| LIFO         | Removes most recent items first                          |
| LRU          | Removes **least recently used** items                    |
| MRU          | Removes **most recently used** items                     |
| LFU          | Removes **least frequently used** items                  |
| RR           | Removes random items                                     |
---
## Data Partitioning (Sharding)
### What is Sharding?
Splitting a large DB into smaller pieces across servers to improve:
- Performance
- Manageability
- Load balancing
- Availability
> **Why?** Horizontal scaling is cheaper and more flexible than vertical scaling.
### Partitioning Methods
#### 1. Horizontal Partitioning
- Split by rows (e.g., ZIP codes `< 10000` vs `>= 10000`)
- Also called **range-based** sharding
>  Unbalanced partitions if data is skewed
#### 2. Vertical Partitioning
- Split by feature or table (e.g., users, photos, followers in separate DBs)
>  May still need horizontal partitioning later
#### 3. Directory-based Partitioning
- Use a **lookup service** to determine data location
>  Easy to rebalance  
>  Adds complexity and a new **SPOF** (single point of failure)
### Partitioning Criteria

| Type                  | Description                                                            |
|-----------------------|------------------------------------------------------------------------|
| **Hash-based**        | `hash(ID) % N` determines server                                       |
| **List-based**        | Explicit value-to-shard mapping (e.g., Nordic countries in 1 shard)    |
| **Round-robin**       | `i-th` record → `(i mod N)-th` partition                               |
| **Composite**         | Combine methods (e.g., List + Hash)                                    |
### Challenges in Sharding

#### a. Joins and Denormalization
- Joins across shards are slow
- Solution: Denormalization (but leads to data duplication/inconsistency)

#### b. Referential Integrity
- Foreign keys across shards not supported
- Enforce integrity in **application logic**

#### c. Rebalancing
Reasons to rebalance:
- Skewed data (hotspots)
- Uneven load

>  Downtime likely unless using **directory-based partitioning**
### Summary

| Strategy            | Strength                         | Weakness                         |
|---------------------|----------------------------------|----------------------------------|
| Horizontal          | Scalable, simple to implement     | Risk of skewed data              |
| Vertical            | Clean separation by feature       | Not scalable in the long term    |
| Directory-based     | Easy rebalancing                 | Extra layer of complexity        |

---
## 📡 Bandwidth vs ThroughPut
> *📶 **Bandwidth** is the maximum amount of data that can be transferred over a network in a given time. Higher bandwidth means faster data transfer.*
> *📥 **Ingress** refers to data **coming into** the system (e.g., user uploads).*
> *📤 **Egress** refers to data **leaving** the system (e.g., user downloads or reads).*

Throughput is the amount of data or number of operations a system can process in a given period
Bandwidth is the maximum capacity of data transfer in a network (like a highway’s width).
Throughput is the actual amount of data successfully transferred over time (like cars actually moving on the highway).

---

# RESTful Conventions
## 🌐 What is REST?
**REST (Representational State Transfer)** is an architectural style for building scalable web services using HTTP. RESTful APIs follow conventions to make services predictable, uniform, and stateless.
## ✅ RESTful API Core Principles
1. **Statelessness**  
   - Each request from client to server must contain all the information needed.
   - Server does **not store** session state.

2. **Resource-Based**  
   - Everything is treated as a resource (e.g., user, product).
   - Resources are identified by URIs.

3. **Use of HTTP Methods (CRUD Mapping)**

| HTTP Method | Action   | Description                          | Example                 |
|-------------|----------|--------------------------------------|-------------------------|
| GET         | Read     | Fetch resource(s)                    | `GET /users/123`        |
| POST        | Create   | Add new resource                     | `POST /users`           |
| PUT         | Update   | Replace resource                     | `PUT /users/123`        |
| PATCH       | Update   | Partially update resource            | `PATCH /users/123`      |
| DELETE      | Delete   | Remove resource                      | `DELETE /users/123`     |
## 📁 URI Naming Conventions

- Use **nouns**, not verbs:  
  ❌ `GET /getUser` → ✅ `GET /users/123`

- Use **plural nouns** for collections:  
  ✅ `GET /products`, `POST /orders`

- Nest resources for hierarchy:  
  ✅ `GET /users/123/orders`

- Use query parameters for filtering/search:  
  ✅ `GET /products?category=shoes&sort=price`

## 🧩 Response Conventions

- Return standard **HTTP status codes**:
  
| Code | Meaning                 | Use Case                      |
|------|-------------------------|-------------------------------|
| 200  | OK                      | Successful GET, PUT, DELETE   |
| 201  | Created                 | Resource created (POST)       |
| 204  | No Content              | Successful, no return body    |
| 400  | Bad Request             | Malformed request             |
| 401  | Unauthorized            | Auth required                 |
| 403  | Forbidden               | No permission                 |
| 404  | Not Found               | Resource doesn't exist        |
| 500  | Internal Server Error   | Unexpected server issue       |

## 🧠 Best Practices

- Use **JSON** as the default response format.
- Include **HATEOAS** (Hypermedia as the Engine of Application State) if needed for discoverability.
- Support **pagination** for large result sets (e.g., `?page=2&limit=10`).
- Use **consistent error messages** with error codes and descriptions.
  
## 📦 Example: RESTful API for a Blog

```http
GET     /posts                 // list all posts
POST    /posts                 // create a new post
GET     /posts/42              // get post with ID 42
PUT     /posts/42              // update post with ID 42
DELETE  /posts/42              // delete post with ID 42
GET     /posts/42/comments     // get comments for post 42
POST    /posts/42/comments     // add comment to post 42


```
 **example-based comparison** of SOAP(Simple Object Access Protocol) vs REST APIs to help you see the difference:

---

## 📦 Use Case: **Get user details by ID**

### ✅ REST API Example (JSON over HTTP)

**URL:**

```
GET https://api.example.com/users/123
```

**Request:**
(No body needed for a GET request)

**Response (JSON):**

```json
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com"
}
```

✔ Simple
✔ Uses standard HTTP verbs
✔ Lightweight (JSON)
✔ Easily testable in browser or Postman

### 🧼 SOAP API Example (XML over HTTP)

**Endpoint:**

```
POST https://api.example.com/userService
```

**Request (XML):**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:usr="http://example.com/user">
   <soapenv:Header/>
   <soapenv:Body>
      <usr:GetUserDetails>
         <usr:UserId>123</usr:UserId>
      </usr:GetUserDetails>
   </soapenv:Body>
</soapenv:Envelope>
```

**Response (XML):**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:usr="http://example.com/user">
   <soapenv:Body>
      <usr:GetUserDetailsResponse>
         <usr:User>
            <usr:Id>123</usr:Id>
            <usr:Name>Alice</usr:Name>
            <usr:Email>alice@example.com</usr:Email>
         </usr:User>
      </usr:GetUserDetailsResponse>
   </soapenv:Body>
</soapenv:Envelope>
```

✖ Verbose
✖ Requires parsing nested XML
✔ Built-in standards for security, reliability, and WSDL documentation
### 🆚 Summary

| Feature            | REST                    | SOAP                      |
| ------------------ | ----------------------- | ------------------------- |
| Request Format     | Simple URL + HTTP       | Enclosed XML in POST body |
| Response Format    | JSON (or XML, optional) | XML only                  |
| Verbosity          | Lightweight             | Heavy                     |
| Browser-Friendly   | Yes                     | No                        |
| Contract (WSDL)    | Optional                | Required                  |
| Security Standards | Needs custom handling   | Built-in WS-Security      |

---

## 🏷️ What is Metadata?
- **Definition**:  
  *"Metadata is data about data."*
### 📌 Examples
- In a **Pastebin** system:
  - Actual data → Paste content (e.g., code, text)
  - **Metadata** → Info about the paste:
    - `CreatedAt`
    - `ExpiresAt`
    - `UserID`
    - `Views`
    - `PasteName`
    - `CustomAlias`
- In an **image**:
  - Actual data → The image pixels
  - Metadata → Resolution, file size, date taken, camera model
### ✅ Summary
- Helps manage, search, sort, and process real data  
- Usually small in size compared to actual content  
- Stored in database for quick access

---
## 📚 What is Indexing?
- **Index** = Data structure (like a shortcut)  
- Speeds up **searching** in a database table  
- Works like an **index in a book** → Jump directly to the needed info
### 🧠 Why Index?
- Without index → DB scans every row (slow)
- With index → DB finds data faster (like using a map)
### 💡 Example:
- Table: 10 million pastes
- Query: Find paste with `URLHash = 'abc123'`
- Without index: Check all 10M rows  
- With index: Jump directly to the match (very fast)
### 🔍 Commonly Indexed Fields:
- Unique IDs like `URLHash`
- Timestamps like `ExpiresAt`
- User IDs, email, etc.
> ✅ Indexing = Faster queries  
> ❌ Too many indexes = Slower writes (trade-off)
---
## 🧱 Storage Options (with Examples)
### 📁 1. HDFS (Hadoop Distributed File System)
> **Example:** Instagram stores millions of images. Instead of keeping them on one server, HDFS splits and distributes them across many machines.

- Distributed file system used to store **huge files** reliably
- Splits large files into **blocks**, stores them across nodes
- **Fault-tolerant**: if one node dies, data is still available from others
- Best for **batch processing** and **big data workloads**
>Batch processing is a way of processing large volumes of data all at once, typically at scheduled times or in groups ("batches"), rather than processing each data item individually and immediately.
>Imagine Instagram wants to:
Generate weekly analytics reports on the most liked photos
Process 100,000 uploaded photos at night to organize them by category
Instead of doing this in real-time (every second), Instagram runs a batch job at 2 AM that processes all the data together.
>
> Fun Fact: Why is it called Hadoop? Hadoop is an Open-source framework for big data processing. 

### ☁️ 2. Amazon S3 (Simple Storage Service)
> **Example:** When a user uploads a photo, it’s saved in a storage bucket on AWS S3 and can be downloaded via a link.

- Cloud-based **object storage** service by Amazon
- Stores **any kind of file** (image, video, document)
- Each file stored as an **object** in a **bucket**
- Supports:
  - High durability and availability
  - Permissions, versioning, encryption
- Can **scale infinitely**
  
```yaml
Bucket: instagram-user-uploads
Object:
  - File: sunset.jpg
  - Metadata:
      - uploaded_by: Agniva_9383
      - location: Goa
      - timestamp: 2025-07-27 18:35
```
### 🧩 3. Cassandra (Wide-Column NoSQL Database)
> **Example:** To get all photos posted by a user or all followers of a user, we store data like:
> - Key = `UserID`
> - Value = list of `PhotoIDs` or `FollowedUserIDs`

- NoSQL database designed for **high write throughput**
- Data organized as **rows with flexible columns** (not fixed schema)
- Excellent for use cases with **heavy reads/writes**, like:
  - Social graphs
  - Time-series data
- **Replicates data** across nodes to prevent data loss
- Deletes are **eventual** (soft-delete first, then removed)

#### Cassandra Example: Flexible Columns

Cassandra organizes data as **rows** in **tables**, but each row can have a different number of columns.

### 👤 Table: UserPhotos

| UserID    | PhotoID_1 | PhotoID_2 | PhotoID_3 |
|-----------|-----------|-----------|-----------|
| agniva938 | img101    | img102    | img103    |
| ria2025   | img201    | img202    |           |
| zed_x     | img301    |           |           |

- **UserID** is the row key.
- Each row stores photo IDs as **flexible columns**.
- Ria only has 2 photos, Zed has 1 — this is allowed.

> ✅ No need for every row to have the same number of columns. Cassandra handles sparse data efficiently.

### ✅ Summary

| Storage Type | Use Case Example                     | Best For                            |
|--------------|--------------------------------------|-------------------------------------|
| HDFS         | Large-scale distributed photo backup | Big data batch processing           |
| Amazon S3    | Storing individual user-uploaded pics| Durable, scalable file storage      |
| Cassandra    | Relationships & metadata             | High-speed access to user data/maps |

###  4. Hbase
### Why HBase is the Best Option for Chat Messages

Chat messages are like **millions of tiny post-it notes**.  
We need to:
1. Stick them somewhere very fast (**write**).
2. Find a bunch of them in order later (**read**).

### Problem with MySQL/MongoDB
- Works like filing each post-it in a cabinet:
  - Open cabinet → Find folder → Put one post-it → Close cabinet.
- Doing this millions of times a second = **too slow** for chat scale.

### How HBase Works Differently
- Uses a **big whiteboard in memory** (**MemStore**) to collect messages quickly.
- Once the board is full, it **takes a photo** (batch save) and stores it on disk (HDFS).
- This avoids slow, one-by-one writes.

### Example: Chat App Flow
1. **User-1 → "Hello" → User-2**
   - Server creates key: `User2:2025-08-13T10:45`
   - Stores in memory:
     ```
     msg:body   = "Hello"
     msg:sender = "User-1"
     ```
2. Many messages go into the **memory basket**.
3. When full, **HBase writes them all at once** to disk.
4. When User-2 checks history:
   - HBase fetches all rows starting with `User2:`
   - Reads them **in order** using fast sequential scans.

### Why This is Perfect for Chat
✅ **Fast writes**: Messages go to memory first.  
✅ **Fast reads**: Can grab a range (e.g., last 50 messages) quickly.  
✅ **Scalable**: Handles millions of chats simultaneously.  
✅ **Flexible**: Store text, images, or metadata in separate columns.

---

## 🤳 What is Long Polling? — An Instagram Tale

Meet **Agniva**, an avid Instagram user. Every time his favorite creator *“_KajuKatli_Queen_420_”* posts, Agniva *must* see it within 3 seconds. “No delay,” he says, “or the vibe is gone.” ⚡
### 🛑 The Problem: Too Many Checks

At first, Agniva wrote a little bot (don’t tell Insta) that **kept asking the server every second**:

> “Hey Instagram, did she post?”  
> “Nope.”  
> “Hey Instagram, now?”  
> “Still nope.”  
> “How about now?”  
> “NO, Agniva. PLEASE.”

This is called **polling** — the client *keeps checking* for updates, like an annoying kid in a car:  
> “Are we there yet?” x1000

### ✅ Enter Long Polling: Chill, Agniva

Now, Agniva uses **long polling**.

Instead of asking repeatedly, he sends **one polite request**:

> 🧘 “Hey Insta, let me know when *KajuKatli_Queen_420_* posts something. I’ll wait.”

Instagram says:

> “Sure. I’ll hold on to your request. You just relax.”

⏳ 15 seconds later...  
BOOM 💥 — a new post about Kaju Samosas.

Instagram responds:

> “Here it is, Agniva!”  
> 🥳 Agniva rejoices, likes, and comments "first 🔥"

And right after, he sends another request to wait for the **next** update. This cycle continues peacefully.

### ⚙️ Summary

| Term            | Meaning |
|-----------------|---------|
| **Client**      | Agniva’s app/browser |
| **Long Poll**   | Server *waits* to reply until there’s something to send |
| **Regular Poll**| Repeatedly asks “anything new?” |
| **Benefit**     | Real-time-ish updates without hammering the server

So with **long polling**, Agniva’s feed stays fresh, the servers stay calm, and *KajuKatli_Queen_420_* gets her rightful likes.

Everybody wins. 🍬

---
## 🧪 Understanding ACID with Dropbox (and Drama)

ACID = Atomicity, Consistency, Isolation, Durability  
These are the **rules of the universe** for data operations in systems like Dropbox. Let’s meet our fictional user: **Agniva**, a chaotic human.
### 🅰️ Atomicity – "All or Nothing!"

> Agniva tries to upload his 500-page **"Thesis on Why Cats Should Rule the World"**.

- ✅ If the entire file uploads, great.
- ❌ If his Wi-Fi dies mid-upload, Dropbox **doesn’t save a half-file**.
- Instead, it **rolls back** like nothing ever happened.

**Moral**: No partial files. Either the file is saved completely or not at all.
### 🅲 Consistency – "Keep the Universe in Balance"

> Dropbox has a rule: **no duplicate file names** in the same folder.

- Agniva accidentally tries to upload **“cat_rules.pdf”** twice.
- Dropbox politely declines: "Bruh, it's already here."

**Moral**: System always moves from one valid state to another — no broken rules.
### 🅸 Isolation – "Don’t Peek While I’m Working"

> Agniva and his cat **Mr. Meowington** are *simultaneously* editing the same file:  
> `"Top_10_Meow_Memes.txt"`

- While Agniva is typing, Dropbox ensures Mr. Meowington doesn’t see half-written nonsense like:
When the cat says "meeeee
- Each user works in isolation until their changes are complete.

**Moral**: Simultaneous actions won’t mess each other up.
### 🅳 Durability – "Even If the World Ends..."

> Agniva finally uploads his life’s work: `"Ultimate_Cat_Manifesto.pdf"`  
> Just as he hits save, a power cut hits his city. 💡⚡

- But fear not! Dropbox already **replicated his file across servers in 3 countries**.

**Moral**: Once Agniva sees “Upload Complete”, the file is safe — even if his laptop explodes.
### ✅ TL;DR: ACID Makes Sure That...

- **Atomicity**: Uploads aren’t half-done.
- **Consistency**: Dropbox always obeys the rules.
- **Isolation**: No messy overlaps when people edit at the same time.
- **Durability**: Once saved, always saved.

🧠 Without ACID, your files could become soup. With ACID, your "Cat Manifesto" is safe forever.

---

# Data Deduplication (with DropBox Example)

Data Deduplication helps eliminate **duplicate data** to save **storage** and **bandwidth**.

🧠 **Idea**: Instead of storing/uploading the same chunk multiple times, store it once and reuse it via references.

### 📘 Example:
Agniva and his friend(hypothetical, he has no friends) both upload the same file: `project_notes.pdf`.  
Instead of storing **two copies**, Dropbox calculates a **hash** for each chunk.

If a chunk already exists (based on hash match), it just **reuses the chunk**.

## 🛠️ Two Deduplication Methods:

### a. 🕒 Post-process Deduplication
- Chunks are stored **first**, deduplicated **later**.
- ✅ Fast upload for clients.
- ❌ Temporary duplicates use extra **storage** and **bandwidth**.

📌 *Example*: Agniva uploads a chunk → gets stored → backend checks later and removes duplicate.

### b. ⚡ In-line Deduplication
- Chunks are **checked immediately** before uploading/storing.
- ✅ Saves **network + storage** instantly.
- ❌ Slight delay for hash comparison during upload.

📌 *Example*: Agniva uploads a chunk → system calculates hash → already exists → only metadata is updated.

## 🧠 Summary Table

| Method        | When it deduplicates     | Pros                         | Cons                          |
|---------------|---------------------------|------------------------------|-------------------------------|
| Post-process  | After storing chunks      | Fast upload for user         | Temp duplicates waste space   |
| In-line       | Before storing/uploading  | Optimal storage & bandwidth  | Slightly slower uploads       |

---

## Reed-Solomon Encoding

> **Reed-Solomon Encoding** is an error-correcting code used to ensure data reliability.  
> It splits data into `k` parts and adds `r` redundant parts, creating `n = k + r` total parts.  
> Any `k` parts can reconstruct the original data, even if up to `r` parts are lost.

**Example**:  
- Original data: `D1, D2, D3`  
- Redundant parts: `R1, R2`  
- Total parts: `D1, D2, D3, R1, R2`  
- If `D2` and `R1` are lost, the original data can still be reconstructed using `D1, D3, R2`.  

> **Use Case**: Ensures data recovery in distributed systems when some servers fail.

---
