# System Design Basics Notes


# Functional vs Non-Functional Requirements

---

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

---

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

---

## 🔄 Comparison Table

| Aspect                  | Functional Requirements                | Non-Functional Requirements               |
|------------------------|----------------------------------------|-------------------------------------------|
| **Focus**              | Features and capabilities              | Performance, quality, constraints         |
| **Defines**            | What the system should do              | How the system should behave              |
| **Validation**         | Verified through test cases/use cases  | Verified via metrics and benchmarks       |
| **Examples**           | Login, Registration, Search            | Latency, Security, Availability           |
| **Driven by**          | Business needs                         | Technical and operational needs           |

---

## 📝 Summary

- Functional = *What it does*
- Non-Functional = *How it does it*

Both are **critical** for successful system design and delivery.


## Overview

When designing scalable systems, consider:
1. **Architectural components**
2. **How components interact**
3. **Tradeoffs in usage and design**

Premature scaling is often inefficient. Instead, plan with scalability in mind to avoid costly reworks later.

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

---

### 2. Reliability
**Definition**: The probability that a system fails within a given time.

- Ensures system continues functioning even when some components fail.
- Achieved through **redundancy**.

**Example**: In an e-commerce platform like Amazon, a user’s cart must persist even if a server crashes.

---

### 3. Availability
**Definition**: System uptime during a specific period.

- High availability ≠ High reliability.
- Achievable via rapid failover and repair mechanisms.

**Example**: An aircraft with low downtime is *available*. One that survives all weather conditions is *reliable*.

> A system can be available but not reliable.

---

### 4. Efficiency
**Measures**:
- **Latency**: Time to receive first item.
- **Throughput**: Items delivered per second.

> Consider:
- Network load
- Hardware/software heterogeneity
- Topology variation

---

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

---

### Benefits
- Faster service and lower latency
- High throughput and reduced downtime
- Predictive insights for bottlenecks
- Reduced stress per server

---

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

---

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

---

### Application Server Cache
- Cache is local to the request-handling node.
- Stored in **memory** (fastest) or **disk** (slower, but faster than network).
- Works best in single-node environments.
-  **Challenge** in multi-node: cache inconsistency due to random request routing.

---

### Global / Distributed Cache
Used to overcome cache misses in distributed setups:
- Shared cache among nodes (e.g., Redis, Memcached)

---

### Content Delivery Network (CDN)
Used for static media (images, videos, etc.).

**How it works**:
1. Request hits CDN.
2. If content available → served.
3. Else → fetch from origin, cache, and serve.

> **Tip**: Serve media from `static.domain.com` for future CDN migration ease.

---

### Cache Invalidation Strategies

| Strategy            | Description                                                                 | Pros                       | Cons                            |
|---------------------|-----------------------------------------------------------------------------|-----------------------------|----------------------------------|
| **Write-through**   | Write to cache **and** DB at same time                                       | Strong consistency          | Higher write latency             |
| **Write-around**    | Write directly to DB, bypass cache                                           | Avoids cache pollution      | Potential read misses            |
| **Write-back**      | Write to cache first, write to DB asynchronously                             | Fast writes                 | Risk of data loss on crash       |

---

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

---

### Partitioning Methods

#### 1. Horizontal Partitioning
- Split by rows (e.g., ZIP codes `< 10000` vs `>= 10000`)
- Also called **range-based** sharding

>  Unbalanced partitions if data is skewed

---

#### 2. Vertical Partitioning
- Split by feature or table (e.g., users, photos, followers in separate DBs)

>  May still need horizontal partitioning later

---

#### 3. Directory-based Partitioning
- Use a **lookup service** to determine data location

>  Easy to rebalance  
>  Adds complexity and a new **SPOF** (single point of failure)

---

### Partitioning Criteria

| Type                  | Description                                                            |
|-----------------------|------------------------------------------------------------------------|
| **Hash-based**        | `hash(ID) % N` determines server                                       |
| **List-based**        | Explicit value-to-shard mapping (e.g., Nordic countries in 1 shard)    |
| **Round-robin**       | `i-th` record → `(i mod N)-th` partition                               |
| **Composite**         | Combine methods (e.g., List + Hash)                                    |

---

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

---

### Summary

| Strategy            | Strength                         | Weakness                         |
|---------------------|----------------------------------|----------------------------------|
| Horizontal          | Scalable, simple to implement     | Risk of skewed data              |
| Vertical            | Clean separation by feature       | Not scalable in the long term    |
| Directory-based     | Easy rebalancing                 | Extra layer of complexity        |

---

---

---


# RESTful Conventions

---

## 🌐 What is REST?

**REST (Representational State Transfer)** is an architectural style for building scalable web services using HTTP. RESTful APIs follow conventions to make services predictable, uniform, and stateless.

---

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

---

## 📁 URI Naming Conventions

- Use **nouns**, not verbs:  
  ❌ `GET /getUser` → ✅ `GET /users/123`

- Use **plural nouns** for collections:  
  ✅ `GET /products`, `POST /orders`

- Nest resources for hierarchy:  
  ✅ `GET /users/123/orders`

- Use query parameters for filtering/search:  
  ✅ `GET /products?category=shoes&sort=price`

---

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

---

## 🧠 Best Practices

- Use **JSON** as the default response format.
- Include **HATEOAS** (Hypermedia as the Engine of Application State) if needed for discoverability.
- Support **pagination** for large result sets (e.g., `?page=2&limit=10`).
- Use **consistent error messages** with error codes and descriptions.

---

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
# Designing a URL Shortening Service like TinyURL

---

# Designing a URL Shortening Service like TinyURL

---

## 🔍 Why URL Shortening?

URL shortening creates compact aliases ("short links") for long URLs to:

* Save space (e.g., in tweets or printed material)
* Reduce typing errors
* Optimize links across devices
* Track link analytics and performance
* Hide affiliate or tracking URLs

**Example:**
Original: `https://www.educative.io/collection/.../5668600916475904/`
Shortened: `http://tinyurl.com/jlg8zpc`

---

## ✅ Functional Requirements

1. Generate a **unique short link** for a given long URL.
2. Redirect a **short link** to the original URL.
3. Support **custom aliases** for short links.
4. Support **link expiration**, with default and user-defined timeouts.

---

## ⚖️ Non-Functional Requirements

1. **High availability**: Redirection service must always be operational.
2. **Low latency**: Real-time URL redirection.
3. **Unpredictable**: Short links should be **non-sequential and non-guessable**.

---

## ⌚ Extended Requirements

1. **Analytics**: Track number of redirections per short link.
2. **REST API**: Expose the service functionality for programmatic access.

---

## ⚙️ System Estimates (Based on 1M new URLs/month, 1000 bytes per URL entry)

### 1. Traffic

* **New URLs**: 1M/month ≈ \~0.39 URLs/sec
* **Redirection requests**: 100x read/write = \~39 URLs/sec

### 2. Storage

* Data per URL entry: \~1000 bytes
* 1M/month × 12 months × 5 years = 60M entries
* Storage: 60M × 1000B = **60GB**

### 3. Bandwidth

* **Write**: 0.39 req/sec × 1000B = \~390B/sec
* **Read**: 39 req/sec × 1000B = \~39KB/sec

### 4. Memory (Caching)

* Daily traffic: 39 req/sec × 3600 × 24 = \~3.37M requests/day
* Cache 20% hot URLs (80-20 rule):

  * Memory: 0.2 × 3.37M × 1000B = **\~675MB**

> Actual memory usage may be lower due to duplicate hits.

---

## ✅ Summary Table

| Component          | Estimate            |
| ------------------ | ------------------- |
| New URLs/sec       | \~0.39              |
| Redirections/sec   | \~39                |
| Incoming data/sec  | \~390 B             |
| Outgoing data/sec  | \~39 KB             |
| Storage (5 years)  | 60 GB               |
| Memory (for cache) | \~675 MB (hot URLs) |

---
