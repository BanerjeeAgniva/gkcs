# Designing a URL Shortening Service like TinyURL

---

## 🔍 1. Why URL Shortening?

URL shortening creates compact aliases ("short links") for long URLs to:

* Save space (e.g., in tweets or printed material)
* Reduce typing errors
* Optimize links across devices
* Track link analytics and performance
* Hide affiliate or tracking URLs

**Example:**
Original: `https://www.agniva.com/photographs.../9348349875/`
Shortened: `http://tinyurl.com/jlg8zp`

---
> *Requirements can be functional/non-functional/extended*  
> *Functional:* User can log in with email and password — *What the system does*  
> *Non-Functional:* Login response must be under 1 second — *How well it does it*  
> *Extended:* System must comply with GDPR — *External rules/constraints (e.g., legal, regulatory)*

## 2. Requirements 

# ✅ Functional Requirements

1. Generate a **unique short link** for a given long URL.
2. Redirect a **short link** to the original URL.
3. Support **custom aliases** for short links.
4. Support **link expiration**, with default and user-defined timeouts.

---

# ⚖️ Non-Functional Requirements

1. **High availability**: Redirection service must always be operational.
2. **Low latency**: Real-time URL redirection.
3. **Unpredictable**: Short links should be **non-sequential and non-guessable**.

---

# ⌚ Extended Requirements

1. **Analytics**: Track number of redirections per short link.
2. **REST API**: Expose the service functionality for programmatic access.
> *REST API allows other programs or services to interact with your application by sending HTTP requests (like GET, POST, etc.) to specific URLs that perform certain actions.*
---

## 📊 3. Capacity Estimation

### 📥 Write Load

- **New URLs/month**: 500 million (Assumption1) 
- = 500M / (30 × 24 × 3600) ≈ **200 URLs/sec** ---> Queries per second QPS = Writes per second WPS

### 📤 Read Load

- **Read:Write ratio** = 100:1  (Assumption2) ----> R/W ratio 
- = 100 × 200 = **20,000 redirections/sec**  Reads per sec = R/W * WPS 

---

### 💾 Storage Estimation

- Store data for 5 years  (Assumption3)
- Total URLs = 500M × 12 × 5 = **30 billion**  
- Avg size per object ≈ 500 bytes  (Assumption4) -----> SizeObj 
- **Total storage = 30B × 500 bytes = 15 TB**  1TB is 1000GB = 10^12 bytes

---

### 📶 Bandwidth Estimation
> *Bandwidth is the maximum amount of data that can be transferred over a network in a given amount of time. Higher bandwidth means faster data transfer.*
- **Write (incoming)** = 200 × 500B = **100 KB/sec**  ----> WPS * SizeObj
- **Read (outgoing)** = 20K × 500B = **10 MB/sec**    ----> RPS * SizeObj

---

### 🧠 Memory for Caching

- 80/20 rule: 20% of URLs generate 80% of traffic
- Total daily redirects ≈ 20K × 3600 × 24 = **1.7B/day**
- Cache 20% = 0.2 × 1.7B × 500 bytes ≈ **170 GB**
> ⚠️ Duplicate requests reduce actual memory usage

---

## 🧮 Summary

| **Metric**               | **Estimate**                                    | **Formula / Notes**               |
|--------------------------|------------------------------------------------|------------------------------------|
| New URLs/sec             | 200                                            | Writes Per Second WPS              |
| Read/Write Ratio         | 100:1                                          | Read/Write (R/W)                   |
| URL Redirections/sec     | 20,000                                         | RPS = WPS × R/W                    |
| Size per Object          | 500 Bytes                                      | Size of each URL object            |
| Incoming Bandwidth       | 100 KB/sec                                     | WPS × SizeObj                      |
| Outgoing Bandwidth       | 10 MB/sec                                      | RPS × SizeObj                      |
| Storage (5 years)        | ~15 TB                                         | WPS × 5 × 12 × 30 × 24 × 3600 × SizeObj |
| Cache Memory (per day)   | ~170 GB                                        | 20% × RPS × 24 × 3600 × SizeObj    |


---

## 📡 **System APIs (RESTful)**

These APIs will be used by developers or internal services to interact with your URL shortening platform.

### 🔗 1. `createurl`

**POST** `/api/v1/shorten`

**Description:**
Create a shortened URL (with optional custom alias and expiration).

**Parameters (in JSON body):**

```json
{
  "api_dev_key": "string",        // Required
  "original_url": "string",       // Required
  "custom_alias": "string",       // Optional
  "user_name": "string",          // Optional
  "expire_date": "YYYY-MM-DD",    // Optional
}
```
> *api_dev_key (string): The API developer key of a registered account. This will be 
used to, among other things, throttle users based on their allocated quota.*

**Response:**

* `200 OK` (Success):

```json
{
  "short_url": "http://short.ly/abc123"
}
```

* `400 Bad Request`: Invalid inputs
* `409 Conflict`: Custom alias already in use
* `401 Unauthorized`: Invalid API key

---

### ❌ 2. `deleteurl`

**DELETE** `/api/v1/url/{url_key}`

**Headers / Query:**

* `api_dev_key`: API key for authentication

**Example:**
`DELETE /api/v1/url/abc123?api_dev_key=XYZ123`

**Response:**

* `200 OK`:

```json
{
  "message": "URL Removed"
}
```

* `404 Not Found`: URL not found
* `401 Unauthorized`: Invalid API key

---

## 🧩 Notes

* All APIs should be **rate-limited** based on `api_dev_key`.
* Use **JWT or API keys** for security and user-specific quotas. JWT(JSON Web Token)
* Errors should return structured messages with HTTP status codes and explanations.

---
# 6. Basic System Design and Algorithm (URL Shortener)

## 🎯 Objective
Design a system to generate **short, unique keys** for long URLs (e.g., `http://tinyurl.com/abc123`).

---

## 🖼️ Diagram Overview
<img width="1137" height="562" alt="image" src="https://github.com/user-attachments/assets/e71cfae9-3ff7-4744-a074-182a6f96630c" />

# Labeled Steps:
1. **Client Request**: Client sends a request to shorten a URL.
2. **Server Encodes URL**: The server passes the URL to the encoding component.
3. **Encoding**: Encoding logic creates a short key (e.g., via hashing).
4. **Database Storage**: Encoded short URL is attempted to be inserted into the database.
5. **Collision Handling**: If a duplicate key exists, append a sequence and retry encoding.
6. **Insert Result**: Insertion success/failure is returned to the server.
7. **Response**: Server returns the shortened URL to the client.

---

## 🔐 Key Generation Strategy

### a. Encode Actual URL
- Compute a **hash** of the original URL using MD5/SHA256.
- Use **Base64/Base62/Base36** to encode hash into a short string.
  - **Base62**: `[A-Z, a-z, 0-9]`
  - **Base64**: Adds `.` and `-`
- Choose a short key length:
  - 6 characters → 64⁶ = ~68.7 billion combinations
  - 8 characters → 64⁸ = ~281 trillion combinations

### 🔁 Key Truncation Strategy
- Take the first 6 or 8 characters of the encoded string.
- If collision occurs:
  - Try different segments of the hash.
  - Or slightly alter characters.

---

## ⚠️ Issues with Simple Hashing

1. **Duplicate URLs**:
   - Two users entering the same URL would receive the same shortened key.
   - This violates the uniqueness requirement for each user submission.

2. **URL-Encoding Conflicts**:
   - URLs like:
     - `http://example.com/?id=design`
     - `http://example.com/%3Fid%3Ddesign`
   - These are technically equivalent but encoded differently.

---

## ✅ Workarounds

- **Append a sequence number** to each input before hashing:
  - Ensures uniqueness.
  - Not stored in DB, just used for generating a unique hash.
  - Potential drawback: sequence management & overflow risks.

- **Append User ID** (if available):
  - Generates different keys for same URL across users.
  - If user is anonymous, ask for a uniqueness key manually.

- **Retry on Collision**:
  - If a generated key exists in DB, retry encoding with a different suffix or hash variant until success.

---

## 🧠 Design Tradeoffs

| Strategy                 | Pros                            | Cons                            |
|--------------------------|----------------------------------|----------------------------------|
| Hashing with Truncation  | Simple, Fast                     | Collision risk                   |
| Append Sequence/User ID  | Ensures uniqueness               | Needs sequence mgmt or auth      |
| Retry on Failure         | Eventually generates unique keys | Latency in worst-case scenarios  |

---

### b 🔑 Key Generation Service (KGS) - Generating keys offline 

## 📦 Offline Key Generation
- A **Key Generation Service (KGS)** pre-generates random 6-letter keys.
- Keys are stored in a **key-DB**.
- No need to hash or encode URLs.
- **Avoids duplication/collision** issues at runtime.

---

## ⚠️ Concurrency Handling
- Risk: Multiple servers might read the **same key** concurrently.
- **Solution**:
  - KGS uses two tables:
    - `unused_keys`
    - `used_keys`
  - On key allocation: move from `unused` → `used`.
  - **KGS loads keys into memory**, marking them used immediately.
    - If KGS crashes before use: keys wasted (acceptable due to large key pool).
  - Synchronization or locking required when accessing key memory structure.

---

## 💾 Key-DB Size Estimation

(each key is 6 characters and 1character takes 1byte space) 
- Using Base64 encoding:
  - Total unique keys possible = 64⁶ ≈ **68.7 billion**
- Storage size calculation:
  - Each key = **6 bytes**
  - Total storage = 6 × 68.7 billion = **412 GB**

---

## 🧱 Fault Tolerance
- **KGS is a single point of failure**.
- Solution: Add a **standby replica** to take over if the primary fails.

---

## ⚡ Caching at App Server
- App servers can **cache a batch of keys** from key-DB.
- If a server crashes with unused keys, they’re **lost** (acceptable loss).

---

## 🔍 Key Lookup Process
- Check if key exists in **main database** or **key-value store**.
  - If present → return `HTTP 302 Redirect` to original URL.
  - If absent → return `HTTP 404 Not Found` or redirect to homepage.

---

## ✏️ Custom Alias Size Limit
- Users can optionally create **custom aliases**.
- Limit custom keys to **16 characters** for consistency.
  - Improves storage, indexing, and user experience.

<img width="926" height="372" alt="image" src="https://github.com/user-attachments/assets/c09fdd2d-36fe-4344-bc59-cae68eb16702" />

## 📊 Diagram Flow Explanation

- **Clients** send requests to the **Application Server**, optionally including a **custom alias**.

- The **Application Server** performs the following:
  - If no custom alias is provided:
    - Consults the **Key Generation Service (KGS)** to get a new unique key.
  - Checks the **Key-DB** to:
    - Ensure key uniqueness (no collisions).
    - Store the generated or validated key.
  - Uses the **Main Database** to:
    - Map the alias (generated or custom) to the original long URL.

## 🧩 7. Data Partitioning and Replication

To scale the database and support billions of URLs, we need to partition and replicate the data.

---

### 📘 a. Range-Based Partitioning

- Partition based on the **first letter** of the URL or key.
  - Example: URLs starting with `A` go to Partition A, `B` to Partition B, etc.
- Can group less frequent letters together (e.g., `Q`, `X`, `Z`).
- ✅ **Pros**:
  - Simple and predictable.
- ❌ **Cons**:
  - May lead to **unbalanced partitions**.
    - Example: Too many URLs starting with `E` → partition overload.

---

### 🔐 b. Hash-Based Partitioning

- Compute a **hash** of the URL or short key.
- Use the hash to assign the object to a partition:
  - Example: `hash(key) % 256` → Partition ID between 1–256.
- ✅ **Pros**:
  - Better distribution than range-based.
- ❌ **Cons**:
  - Still possible to get **hot partitions** (uneven load).

---

### ♻️ Consistent Hashing (Solution to Overload)

- Places **both keys and partitions** on a hash ring.
- A key is stored in the **next clockwise partition** on the ring.
- ✅ **Advantages**:
  - Smooth scaling: Adding/removing partitions only affects nearby keys.
  - Reduces re-distribution of data.
  - Helps maintain balanced load.

## 🧠 8. Cache

- Use **Memcache** (or similar) to store hot URLs with their hashes.
- App servers check cache **before DB** for faster access.

### 🧮 Cache Size
- Start with **20% of daily traffic**.
- Need ≈ **170GB RAM** for 20% traffic.
- Can use **1 large (256GB)** server or **multiple small** ones.

### 🔄 Eviction Policy
- Use **LRU (Least Recently Used)** policy.
- Discard URLs least recently accessed.
- Use **Linked Hash Map** to track access order.

### 📡 Cache Replication
- Replicate cache to distribute load.
- On **cache miss**, app server fetches from DB and:
  - Updates local cache.
  - Notifies all replicas to update.
  - Replicas **ignore** if entry already exists.

<img width="891" height="477" alt="image" src="https://github.com/user-attachments/assets/930a597e-4565-4fe2-9300-f81e5c1f6f8a" />
<img width="1066" height="538" alt="image" src="https://github.com/user-attachments/assets/81efe487-6216-4017-8db2-29ea7e5d0971" />

## 🌐 9. Load Balancer (LB)

### 📍 Where to place LB:
1. Between **Clients ↔️ App Servers**
2. Between **App Servers ↔️ Database**
3. Between **App Servers ↔️ Cache**

### 🔄 Strategy:
- Start with **Round Robin**:
  - Distributes requests evenly.
  - Skips dead servers automatically.
  - ✅ Simple and low overhead.
  - ❌ Doesn't consider current server load.

### 🧠 Smarter LB:
- Query backend for **load info** (CPU, memory, etc.).
- Dynamically adjust traffic based on server **health/performance**.

<img width="900" height="391" alt="image" src="https://github.com/user-attachments/assets/29027fb0-e749-4c28-b7ee-035a85e0a2ca" />

## 🧹 10. Purging or DB Cleanup

- ❓ Should links expire?
  - Use **expiration time** (e.g., default: 2 years).
  - Expired links not returned to users.

### 🐢 Lazy Cleanup Strategy
- Avoid active scanning (too heavy on DB).
- Instead:
  - ⛔ On access to expired link: delete & return error.
  - 🧽 Run lightweight **Cleanup Service** periodically (during low traffic).
  - 🗑 Delete from storage and cache.

### ♻️ Reusing Keys
- After deletion, return keys to **Key-DB** for reuse.

### ⚠️ Inactive Links?
- Removing unvisited links (e.g., older than 6 months) is tricky.
- Storage is cheap → may choose to **keep links forever**.


## 📊 11. Telemetry

- Track usage stats: views, country, timestamp, referrer, browser, platform.
- Frequent updates to DB row (for popular URLs) can cause write contention.
- Better approach: asynchronously log events (e.g., message queue or log service) and aggregate later.

---

## 🔐 12. Security and Permissions

- Support private URLs or user-specific access.
- Store permission level (`public/private`) with each URL.
- Use separate table:  
  - **Key**: URL hash / short key  
  - **Columns**: allowed `UserIDs`
- Return `HTTP 401` for unauthorized access.
