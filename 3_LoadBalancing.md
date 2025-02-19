# Consistent Hashing

## Introduction
- Consistent hashing is an important concept in distributed systems.
- It helps distribute requests across multiple servers efficiently.
- Essential for scaling systems that handle large amounts of requests.

## Server-Client Model
- A **server** runs an algorithm to process requests.
- A **client** (e.g., a mobile user) sends a request to the server.
- The server processes the request and returns a response.

### Example:
- A facial recognition algorithm that adds a mustache to an image.
- The client sends an image, the server processes it, and sends back the modified image.

## Scaling Issues
- Initially, a single server handles all requests.
- As demand grows, a single server is insufficient.
- More servers are added to distribute the load.

## Load Balancing Problem
- When multiple servers exist, we must decide where to send each request.
- The goal is to distribute requests evenly among servers.
- This process is called **load balancing**.

### Example:
If we have **N** servers, each server should handle roughly **1/N** of the requests.

## Hashing Requests
- Each request has a unique **request ID**.
- Request IDs are assumed to be uniformly random.
- We use a **hash function** to map request IDs to servers.

### Example:
- Assume 4 servers (S0, S1, S2, S3)
- Request ID: `R1 = 10`
  - `H(R1) = 3`
  - `3 mod 4 = 3` → Sent to **Server S3**
- Request ID: `R2 = 20`
  - `H(R2) = 15`
  - `15 mod 4 = 3` → Sent to **Server S3**
- Request ID: `R3 = 35`
  - `H(R3) = 12`
  - `12 mod 4 = 0` → Sent to **Server S0**
- The hash function ensures requests are evenly distributed.

## Request Stickiness
- Many requests are tied to **specific users**.
- If the same user makes multiple requests, they should ideally go to the same server.
- This enables caching for efficiency.

## Issue with Adding More Servers
- When a new server is added, the entire hashing scheme changes.
- Example:
  - Initially, `mod 4` was used (4 servers)
  - Adding a 5th server requires `mod 5`
  - This remaps all request IDs to different servers
  - **Results in cache invalidation and inefficient load distribution**

### Example:
Before adding a server:
- `H(R1) = 3` → `3 mod 4 = 3` → Sent to **S3**
- `H(R2) = 15` → `15 mod 4 = 3` → Sent to **S3**
- `H(R3) = 12` → `12 mod 4 = 0` → Sent to **S0**

After adding a 5th server:
- `H(R1) = 3` → `3 mod 5 = 3` → Sent to **S3** (Unchanged)
- `H(R2) = 15` → `15 mod 5 = 0` → Sent to **S0** (Changed)
- `H(R3) = 12` → `12 mod 5 = 2` → Sent to **S2** (Changed)

## Solution: Consistent Hashing
- Instead of remapping all requests, **consistent hashing** minimizes the number of changes.
- Requests are distributed in a circular space (hash ring).
- When a new server is added, only a small portion of requests are reassigned.

### Example Using a Hash Ring:
1. Assume a **hash space of 0-99**.
2. With 4 servers, each server gets 25% of the space.
3. Adding a new server divides the space more evenly, but **only a few requests are reassigned**.

### Visualization:
Before adding a server:
```
| S0 (0-24)  | S1 (25-49) | S2 (50-74) | S3 (75-99) |
```
After adding **S4**:
```
| S0 (0-19)  | S1 (20-39) | S2 (40-59) | S3 (60-79) | S4 (80-99) |
```
- Only a few requests move to S4 instead of reassigning everything.

## Benefits of Consistent Hashing
- **Minimizes request reassignment** when adding/removing servers.
- **Improves caching efficiency** since most requests go to the same server.
- **Balances load effectively** while reducing unnecessary redistributions.

## Summary
- Load balancing distributes requests across multiple servers.
- Standard hashing (`mod N`) leads to large-scale changes when adding servers.
- **Consistent hashing** solves this by **minimizing remapped requests**.
- This ensures efficient caching and **better scalability**.

---

<img width="1271" alt="Screenshot 2025-02-19 at 9 08 24 PM" src="https://github.com/user-attachments/assets/762aec04-0544-48b6-9a8a-4889c2633001" />

# Load Balancing Failure When Increasing Servers (4 → 5)

## Initial Setup (4 Servers)
We initially have **4 servers** (`S0, S1, S2, S3`).  
Requests are **hashed and mapped** using:

\[
\text{server} = (\text{hash(request\_ID)} \mod 4)
\]

Each server gets **25%** of the request IDs.

### Example:
- **Request R1** → `H(10) = 3`, `3 mod 4 = 3` → **Sent to S3**
- **Request R2** → `H(20) = 15`, `15 mod 4 = 3` → **Sent to S3**
- **Request R3** → `H(35) = 12`, `12 mod 4 = 0` → **Sent to S0**

## After Adding a New Server (S4)
Now we have **5 servers** (`S0, S1, S2, S3, S4`).  
New hashing formula:

\[
\text{server} = (\text{hash(request\_ID)} \mod 5)
\]

### Recomputed Mapping:
- **Request R1** → `H(10) = 3`, `3 mod 5 = 3` → **Still S3** ✅ (No change)
- **Request R2** → `H(20) = 15`, `15 mod 5 = 0` → **Moved to S0** ❌
- **Request R3** → `H(35) = 12`, `12 mod 5 = 2` → **Moved to S2** ❌

### Impact:
- **75% of requests got reassigned to different servers** 🔄.
- **Cache stored in previous servers is lost**.
- **Requires redistributing ~100 request buckets**.
- **Operations cost is O(100), as all stored mappings must be recomputed**.

## Why This is a Problem?
- **User sessions break** due to sudden server changes.
- **Cache performance drops** because users are mapped to new servers.
- **Data replication or re-fetching is needed** to restore cache.

This is why **consistent hashing** is needed to minimize remapping when adding/removing servers.

