# **Database Sharding**

---

## **Why Not Traditional Approaches?**
1. **SQL Optimization**  
   - Query optimization helps, but it’s not enough for massive datasets.  
   
2. **Indexing**  
   - Indexing improves search speed, but large indexes can become **bottlenecks** in high-volume databases.  

3. **NoSQL?**  
   - While NoSQL databases can handle scalability, **sharding** applies to both **SQL and NoSQL** databases.
   
---

# **Sharding**
<img width="560" alt="Screenshot 2025-03-02 at 10 38 04 PM" src="https://github.com/user-attachments/assets/356cdc31-d93d-4740-83d3-1d833b566410" />


## **What is Sharding?**
Imagine you have a **large pizza**, and you **can't eat the entire thing alone**.  
So, you **slice it into pieces** and invite your **friends** over to share.  

Each friend gets **one slice**, meaning you have effectively **partitioned** the pizza based on the number of friends.  

Similarly, in **databases**, when there is a **large amount of data**, handling all requests on a **single server** becomes **inefficient**.  
To **distribute the load**, we **partition the data** across multiple servers.  


# **Database Sharding**

## **Breaking (Partitioning) the Pizza (Data) into Multiple Pieces**
Sharding is the process of **breaking data into pieces** and then **allocating** it to different servers **based on a key**.  
This is known as **horizontal partitioning**.

**Important:**  
The **"servers"** in sharding are **database servers**, not **application** or **platform servers**, which handle user interactions and business logic.

## **Database Sharding: A Specific Type of Horizontal Partitioning**
Sharding is a **subset** of horizontal partitioning that distributes **database load** across multiple servers.

---

## **Database Properties**
1. **Consistency**  
   - Any data written to the database must **stay** in the database.  
   - Any **update** should be **reflected** when fetching the data.  

2. **Availability**  
   - The database should always be **accessible**.  
   - It **should not crash** or stay down.  

---

## **What Do You Shard Your Data On?**
- **User ID?**  
  - It **depends** on the application.  
  - Example: **Tinder** shards its data **based on location**.  
    - If **User X** is in a **specific location**, their request is directed to **only one shard** (e.g., **Database Server 7**).  
<img width="446" alt="Screenshot 2025-03-03 at 2 37 09 PM" src="https://github.com/user-attachments/assets/27d9f6cb-e705-416d-8592-9f8406ab6ddc" />

---

## **Drawbacks of Sharding**
1. **Joins Across Shards**  
   - Queries need to **access multiple shards**, retrieve data, and then **join** the results.  
   - This is **computationally expensive** and **slower**.  

2. **Fixed Number of Shards**  
   - Ideally, you want the number of database servers to be **flexible**.  
   - However, traditional sharding has a **fixed number of shards**, making scaling difficult.  

### **Solutions**
- **Consistent Hashing / Memcache**  
  - Helps with distributing load dynamically across servers.  
- **Hierarchical Sharding (Minishards)**  
  - Further **break down shards** into **smaller shards** for better scalability.  

---

## **Best Practices**
### **1) Indexing on Shards**
**Example:**  
- Query: _Find all people in New York who are older than 50._  
- **Solution:**  
  - Shard **based on city ID** (e.g., **New York → City-ID 123**).  
  - **Index within the shard** on **age > 50** for fast retrieval.  

### **2) Handling Shard Failures**
- Use a **Master-Slave Architecture**:  
  - **Master node** handles the latest **write operations**.  
  - **Slave nodes** replicate data from the master and handle **read operations**.  
  - If the **master fails**, one of the slaves **promotes itself** to master.  
  - Read requests are redirected to the **least busy slave** for **load balancing**.  

---

Sharding helps scale databases efficiently but comes with challenges like **complex queries across shards** and **fixed shard limits**.  
By **implementing best practices** like **consistent hashing, minishards, indexing, and master-slave architecture**, we can **optimize** performance and **improve scalability**. 


