## 📸 1 What is Instagram (Simplified)

- **Purpose**: Photo sharing social network  
- **Users**: Upload & share images  
- **Follow System**: Users can follow others  
- **News Feed**:  
  - Displays top photos  
  - From followed users  
- **Features (for design scope)**:  
  - Upload photo  
  - Follow user  
  - View feed (photos from followed users)
---
## 📌 2 Requirements & Goals
### ✅ Functional
- Upload / Download / View photos  
- Search photos by **title**  
- **Follow** users  
- Generate **News Feed** (top photos from followed users)
### ⚙️ Non-Functional
- **High availability**  
- **Latency**: < 200ms (News Feed)  
- **Eventual consistency** acceptable -  if a user doesn’t see a photo for a while; it should be fine.   
- **High reliability** (no data loss for uploads)
### ❌ Out of Scope
- Tags  
- Comments  
- User tagging  
- Recommendations ("Who to follow")
---
## 💡3 Design Considerations
- **Read-heavy** system ➝ optimize photo retrieval  
- **Unlimited uploads** ➝ storage management critical  
- **Low latency** for photo views  
- **100% reliability** for uploaded content (no loss)
---
## 📊 4 Capacity Estimation

- **Total Users**: 500M  -----> Assumption1
- **Daily Active Users**: 1M  -----> Assumption2
- **Photos/day**: 2M ➝ ~23 photos/sec  -----> Assumption3 
- **Avg Photo Size**: 200KB  -----> Assumption4
### 🗃️ Storage Needs
- **Daily Storage**: 2M × 200KB = 400GB/day  
- **10-Year Storage**: 400GB × 365 × 10 ≈ 1425TB (~1.4PB)
---
## 🧱5 High-Level System Design
<img width="942" height="405" alt="image" src="https://github.com/user-attachments/assets/b94e8426-43fe-4119-a773-27e1f0edecee" />

### 🧾 Core Scenarios
- **Upload Photo**  
- **View/Search Photo**
### 🗄️ Components
- **Image Hosting System**  
  - Coordinates upload/view/search
- **Image Storage (Object Storage)**  
  - Stores actual image files
- **Image Metadata (DB Storage)**  
  - Stores image info: user, title, upload time, etc.
### 🔁 Flow
1. Client ➝ Upload ➝ Image Hosting ➝ [Image Storage + Metadata]
2. Client ➝ View/Search ➝ Image Hosting ➝ Metadata ➝ Serve Image
---
## 🗄️ 6 Database Schema
### 📸 Tables
<img width="918" height="337" alt="image" src="https://github.com/user-attachments/assets/95080b77-ea13-4cd7-af5b-5ec43772aec2" />

- **Photo**
  - Stores: photo location, user, timestamps, coordinates
  - Index: `PhotoID`, `CreationDate` (fetch recent photos fast)
- **User**
  - Basic user profile info
- **UserFollow**
  - Follows graph: (UserID1 → follows → UserID2)
### 🛠️ RDBMS vs NoSQL
- Use **RDBMS (e.g. MySQL)** when you need joins
- Use **NoSQL (e.g. Cassandra)** for **scalability**, high availability, and wide-column storage
### 🗃Storage Options
#### 📂 HDFS (Hadoop Distributed File System)
- Distributed storage system for big data
- Stores large files across multiple machines
- Fault-tolerant and scalable
#### ☁️ Amazon S3
- Object storage service by AWS
- Stores files (like images) with high durability
- Access via HTTP; supports versioning, permissions
### 🧩 Cassandra (Wide-Column NoSQL DB)
- Highly available, distributed DB
- Data stored in **rows and dynamic columns**
- Suitable for:
  - UserPhoto table: `Key = UserID`, `Value = list of PhotoIDs`
  - UserFollow table: `Key = UserID`, `Value = list of followed UserIDs`
- **Replicas** for reliability
- Deletes are **eventual**: data is retained temporarily before permanent removal
### ✅ Summary
- Use **object storage** (S3, HDFS) for photo files
- Use **metadata DB** (SQL/NoSQL) for search, feed, relationships
- Choose **Cassandra** for mapping
---
## 7 Data Size Estimation 
Assuming each “int” and “dateTime” is 4 bytes
### 👤 User Table
- **Schema**: 
  - UserID (4B)
  - Name (20B)
  - Email (32B)
  - DateOfBirth (4B)
  - CreationDate (4B)
  - LastLogin (4B)
- **Total/Row**: `68 bytes`
- **Users**: `500M`  Assumption1
- **Storage**: `500M * 68B = ~32 GB`
### 🖼️ Photo Table
- **Schema**:
  - PhotoID (4B)
  - UserID (4B)
  - PhotoPath (256B)
  - PhotoLat/Lon (4B + 4B)
  - UserLat/Lon (4B + 4B)
  - CreationDate (4B)
- **Total/Row**: `284 bytes`
- **New Photos/Day**: `2M` Assumption3
- **Daily Storage**: `2M * 284B = ~0.5 GB`
- **10-Year Storage**: `0.5GB * 365 * 10 = ~1.88 TB`
### 👥 UserFollow Table
- **Schema**: 
  - FollowerID (4B)
  - FolloweeID (4B)
- **Total/Row**: `8 bytes`
- **Assumption**: 500M users, avg. 500 follows/user
- **Total Relationships**: `500M * 500 = 250B`
- **Storage**: `250B * 8B = ~1.82 TB`
### 🧮 Total Storage (10 Years)
- User Table: `32 GB`
- Photo Table: `1.88 TB`
- UserFollow Table: `1.82 TB`

> **Total ≈ 3.7 TB**
---
## 🧩8 Component Design – Read vs Write Separation

### ⚙️ Problem:
- **Writes (uploads)** are slower (disk-heavy), can **consume all connections**.
- **Reads (downloads)** are faster (often from cache), but get **blocked** if too many uploads happen.
### 🌐 Web Server Limit:
- Assume max **500 concurrent connections**.
- Cannot handle >500 uploads or reads **at once**.
### ✅ Solution:
- **Separate upload (write)** and **download (read)** services.
- Run **dedicated servers** for each:
  - Optimize independently.
  - Prevent uploads from hogging the system.
  - Improves **scalability** and **availability**.
<img width="928" height="421" alt="image" src="https://github.com/user-attachments/assets/ea13ad0c-dce3-4ed1-b6a1-904bdbdc1b1c" />

### 📤 Upload Flow
1. **Client** → sends an **Upload Image Request**.
2. Request is handled by the **Upload Image Service**.
3. This service performs **two operations**:
   - Stores actual photo to **Image Storage** (e.g., S3, HDFS).
   - Saves photo details (user, timestamp, location) to **Image Metadata DB**.

### 📥 Download/View/Search Flow
1. **Client** → sends a **Download/View/Search Request**.
2. Request is handled by the **Download Image Service**.
3. This service:
   - Fetches metadata from the **Image Metadata DB** (e.g., filename, photo ID, path).
   - Uses metadata to retrieve actual image from **Image Storage**.
4. Image is then returned to the user.'
### 📈 Benefit:
- System handles high traffic efficiently.
- **Reads stay responsive** even during heavy upload times.
---
## 🔄 9 Reliability & Redundancy
<img width="1063" height="561" alt="image" src="https://github.com/user-attachments/assets/e3ce8f26-dfb1-4973-aa51-69a71022d3de" />
  
### ✅ Key Reliability Concepts
- **Replication**: 
  - Images are stored in multiple storage nodes to avoid single points of failure.
- **Failover Support**:
  - Upload/download services have **redundant instances** (extra service copies).
  - If one fails, another takes over—either **automatically** or with **manual intervention**.
- **Backup of Metadata**:
  - Metadata is crucial to locate and manage images.
  - Backing it up prevents loss of searchability and associations.
### 💡 Why Redundancy Matters
- Prevents **service downtime**.
- Ensures **zero data loss**.
- Enables **scalability** and **fault tolerance**.
> A well-designed photo-sharing system like Instagram must expect failures and build in enough backup and failover to stay online no matter what.
---
## 10🧩 Data Sharding – Instagram 
To manage huge volumes of metadata (e.g. photos, users), we **shard** the data — i.e., split it across multiple database servers.
If one DB shard is 1TB, we will need four shards to store 3.7TB of data. Let’s assume for better performance and 
scalability we keep 10 shards. 
### 🔹 a. Sharding by UserID
- **Logic**: Store all photos of a user on the same shard using:
ShardID = UserID % 10
- **PhotoID**: Append ShardID to a local auto-increment ID for uniqueness.
- Example: `UserID: 9383 → Shard 3`
- PhotoID on Shard 3: `12345 → "3_12345"`
#### ❗ Problems:
1. **Hot users** → popular users overload a shard.
2. **Uneven distribution** → some users upload many photos.
3. **Shard limits** → can’t store all user data on one shard forever.
4. **Single point risk** → if shard fails, all user’s photos are unavailable.
### 🔸 b. Sharding by PhotoID
- **Logic**: Generate a global unique PhotoID first, then:
ShardID = PhotoID % 10
- **ID Generation**:
- Use a dedicated ID Generator DB. ---> for example generates 38457 for a photo
- Example: 
  - Insert into a simple `PhotoID` table with `BIGINT` (64-bit).
  - Get `PhotoID = 38457 → Shard 7`
#### ✅ Benefits:
- Even data spread.
- Avoids hot shard due to single user.
#### ❗ Trade-off:
- **Key generator DB** is a single point of failure.
### 🛠️ Reliability Fix:
- Use **two ID generators**:
  <img width="273" height="166" alt="image" src="https://github.com/user-attachments/assets/138ca158-9af1-4135-99e9-7a7ce26334b4" />

- One gives even IDs, another gives odd IDs.
- Example: `PhotoID: 1002 (even) from DB1`, `1003 (odd) from DB2`
- Add a **load balancer** to round-robin between generators.
### 📈 Planning for Scale
- Use **logical partitions** upfront (e.g., 1000 shards).
- Map these to **physical DBs** initially (e.g., 10 per DB).
- Maintain a **config file or mapping DB**:
- When a server is full, update config to move partitions.
  ### 📈 Planning for Scale (Explained with Example)
To handle future growth without major re-engineering, we **plan for scale from the start** by logically splitting data and allowing flexible mapping.
#### 🔹 Use **logical partitions** upfront  
Even if we don’t need hundreds of database shards right now, we divide our data into, say, **1000 logical partitions (or shards)**. These are just virtual divisions that help with organizing and routing data.
🧠 **Why?**  
- Makes future scaling easier  
- Avoids expensive re-sharding later
🔢 **Example**:  
Let’s say `PhotoID = 348922`.  
Then:  
Partition = PhotoID % 1000 = 922
So, this photo belongs to logical **Partition 922**.
#### 🔹 Map logical partitions to **physical databases**
Initially, we don’t need 1000 physical databases. We can start with 10 servers and assign **100 logical partitions per server**.
📦 **Example**:
```yaml
DB-0: Partitions 0–99
DB-1: Partitions 100–199
...
DB-9: Partitions 900–999
```
So, Partition 922 (from the previous example) would go to **DB-9**.
#### 🔹 Maintain a **config file or mapping database**
We keep a centralized configuration that maps each **logical partition to a physical database**.
📄 **Example (YAML-style config):**
```yaml
partition_map:
  0-99: db0.company.com
  100-199: db1.company.com
  ...
  900-999: db9.company.com
```
When a request comes in, the system:
Computes the partition (PhotoID % 1000)
Looks up which DB to hit using this map
🔹 When a server is full, update the config
If a database server (e.g., DB-9) becomes too full or overloaded, we can move some partitions (e.g., 900–949) to a new server (e.g., DB-10), and simply update the config.
📥 Before:
```yaml
900-999: db9.company.com
```
📤 After:
```yaml
900-949: db10.company.com
950-999: db9.company.com
```
✅ No change needed in application logic.
🚀 Just update the config and restart services if needed.

✅ Advantages
1.Efficient and scalable from the start
2.Easy to migrate data gradually
3.Load balancing across DBs
4.No hard downtime for re-partitioning

### ✅ Summary

| Sharding Method     | Pros                          | Cons                                |
|---------------------|-------------------------------|-------------------------------------|
| By UserID           | Easy grouping per user        | Hot users, uneven load              |
| By PhotoID          | Balanced distribution         | Needs reliable ID generation        |
| Logical Partitions  | Easy to scale/migrate shards  | Needs mapping logic/config system   |

---
