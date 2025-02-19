# Consistent Hashing Notes

## What is Consistent Hashing?
- A technique used in **distributed systems** to minimize data movement when servers are added or removed.
- Used by **DynamoDB, Apache Cassandra, Akamai CDN, Discord**.
- Helps with **load balancing**, **data partitioning**, and **fault tolerance**.

---

## Why Do We Need It?
### Horizontal Scaling
- **Data is too large for a single server** → Spread across multiple servers.
- This is called **horizontal scaling**.

### Simple Hashing and Its Problem
1. **Hash each object key** using a function like `MD5` or `MurmurHash`.   ---> **OBJECT KEYS !!**
2. **Use modulo operation** to assign objects to servers:

   {server} = {hash(object key)} % {number of servers}
   
<img width="979" alt="Screenshot 2025-02-20 at 12 42 56 AM" src="https://github.com/user-attachments/assets/efc570d7-e419-47cb-bcfb-3511f92b2bce" />

#### Example:
- **4 servers** (`S0, S1, S2, S3`)
- **8 keys** distributed among them.

| Object Key | Hash Value | Server (`mod 4`) |
|------------|------------|------------|
| K0 | 12 | 0 (S0) |
| K1 | 25 | 1 (S1) |
| K2 | 39 | 3 (S3) |
| K3 | 44 | 0 (S0) |
| K4 | 51 | 3 (S3) |
| K5 | 63 | 3 (S3) |
| K6 | 72 | 0 (S0) |
| K7 | 87 | 3 (S3) |

<img width="834" alt="Screenshot 2025-02-20 at 12 44 16 AM" src="https://github.com/user-attachments/assets/e923be81-983e-4ef9-bb4e-08577162e98b" />

- **Problem:** If the number of servers changes, most keys are remapped.

#### When a Server is Removed (S1 goes down)
- The cluster now has **3 servers** (`mod 3` instead of `mod 4`).
- Almost **all objects get reassigned**, even those not on `S1`.

#### When a Server is Added (S4 added)
- The new server changes the modulo operation (`mod 5`).
- **75% of requests are remapped** → Causes cache misses and load spikes.
<img width="925" alt="Screenshot 2025-02-20 at 12 45 33 AM" src="https://github.com/user-attachments/assets/98a60190-2469-4f96-b1cc-582f07603b96" />
---

## How Consistent Hashing Solves This
1. **Hash both object keys and server names/IPs**.   ---> **OBJECTS KEYS AND SERVER NAMES/IPS!!**
2. **Map everything onto a circular "hash ring"**.
3. **Objects move to the nearest server (clockwise direction)**.
<img width="943" alt="Screenshot 2025-02-20 at 12 47 39 AM" src="https://github.com/user-attachments/assets/ddd59a47-5ca8-4197-b1a6-d028fcef70ee" />
<img width="1121" alt="Screenshot 2025-02-20 at 12 48 18 AM" src="https://github.com/user-attachments/assets/0bed4f66-93e4-406a-a9a0-d98723183a8f" />
<img width="405" alt="Screenshot 2025-02-20 at 12 49 31 AM" src="https://github.com/user-attachments/assets/f72189fd-2e28-42bf-b520-8f7c76dfb6db" />

### Example (4 Servers, Hash Ring)
- **Servers (`S0, S1, S2, S3`) are placed on the ring.**
- **Objects are placed on the ring using the same hash function.**
- To find a server for an object:
  - **Move clockwise** from the object's position until a server is found.
<img width="1091" alt="Screenshot 2025-02-20 at 12 50 33 AM" src="https://github.com/user-attachments/assets/8ea38695-143e-4c03-8d56-dbdea596a00f" />


#### Adding a Server (S4 added)
- `S4` is placed between `S0` and `S1`.
- **Only a few objects move to `S4`** (instead of all objects being remapped).
<img width="917" alt="Screenshot 2025-02-20 at 12 51 12 AM" src="https://github.com/user-attachments/assets/0d0d820a-de92-4856-a09a-fa13894619ca" />
  
#### Removing a Server (S1 removed)
- **Only objects mapped to `S1` are reassigned** to the next clockwise server (`S2`).
- **Minimal data movement** compared to simple hashing.
<img width="524" alt="Screenshot 2025-02-20 at 12 52 04 AM" src="https://github.com/user-attachments/assets/88136d9c-f1f9-4211-83b8-b16a35e3aa71" />

---

# Uneven Distribution in Consistent Hashing
<img width="838" alt="Screenshot 2025-02-20 at 12 54 50 AM" src="https://github.com/user-attachments/assets/55733c37-c895-4d1d-a482-8893c73499f4" />
## Why Does Uneven Distribution Happen?

1. **Random Placement on the Hash Ring**
   - Servers are **hashed to random positions** on the hash ring.
   - The distribution is **unlikely to be perfectly even**.
   - Some servers may end up **handling a larger portion of the ring** than others.

2. **Example: Uneven Server Placement**
   - Suppose three servers (`S0, S1, S2`) are placed like this:
     ```
     |----------------|------|--------------|------------------|
     S0             S1      S2             S0 (wraps around)
     ```
   - **S2 has a larger segment**, so it will store **more objects** than `S0` or `S1`.
   - This causes **load imbalance**.



3. **Frequent Server Changes Make It Worse**
   - **Adding or removing servers** changes the hash ring.
   - **Example: Removing `S1`**
     - The objects assigned to `S1` now move to `S2`.
     - `S2`'s segment **doubles in size**, creating an even bigger imbalance.
<img width="884" alt="Screenshot 2025-02-20 at 12 55 57 AM" src="https://github.com/user-attachments/assets/11898fee-d351-4230-add7-67dc9e3d4ad6" />
   - This effect **disrupts load balancing** and can overload certain servers.
--

## Virtual Nodes (Fixing Uneven Distribution)
- **Problem:** Servers on the hash ring might be unevenly distributed.
- **Solution:** Each server is assigned **multiple virtual nodes**.

#### Example:
- Instead of just `S0` and `S1`, create:
  - `S0_0`, `S0_1`, `S0_2` (for `S0`)
  - `S1_0`, `S1_1`, `S1_2` (for `S1`)
- **More balanced object distribution.**
- **Real-world systems use many virtual nodes per server.**
<img width="977" alt="Screenshot 2025-02-20 at 12 57 27 AM" src="https://github.com/user-attachments/assets/47718aed-5a1b-4323-83e5-7de1e2e5c02a" />
- the segments labeled s0 are managed by server 0, and  those labeled s1 are handled by server 1.
  
---

- In real world systems, the number of  virtual nodes is much larger than three. As the number of virtual nodes increases,  the distribution of objects becomes more balanced.  Having more virtual nodes means taking more space  to store the **metadata** about the virtual nodes.  This is a **trade-off** , and we can tune the number  of virtual nodes to fit our system requirements.
  
---

## Real-World Applications
- **Amazon DynamoDB, Apache Cassandra** → **Data partitioning**, reducing rebalancing overhead.
- **Akamai CDN** → **Even distribution of web content** among edge servers.
- **Google Load Balancer** → **Minimizes reassigning persistent connections** when a backend goes down.

---

## Summary
1. **Consistent hashing minimizes object movement** when servers change.
2. **Uses a hash ring instead of modulo-based hashing**.
3. **Virtual nodes** ensure **better load balancing**.
4. **Used in distributed databases, CDNs, and load balancers**.

---

This is why **consistent hashing is the preferred method** in distributed systems. 
