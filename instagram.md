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

- **Total Users**: 500M  
- **Daily Active Users**: 1M  
- **Photos/day**: 2M ➝ ~23 photos/sec  
- **Avg Photo Size**: 200KB  
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
## 🗄️ 5 Database Schema
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
