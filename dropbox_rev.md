<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/0f5e3ff7-e133-4b1c-935e-ce341f2728d6" />

# ☁️ The Story of "CloudBox" — A Dropbox-Style System

Meet **Agniva** — a graduate student juggling research papers, presentations, and travel between home, campus, and conferences.  
He uses **CloudBox** (our Dropbox-like system) to store, sync, and share his work.

---

## 📅 Day 1 — First Upload

Agniva drags `final_thesis.pdf` into his **CloudBox folder** on his laptop.

### 🔍 Step 1: Watcher Spots the Change
The **Watcher** (a local folder monitor) immediately detects:
- **New file created** → `/thesis/final/final_thesis.pdf`

It notifies the **Indexer**.

---

### 🧠 Step 2: Indexer Prepares Upload
The **Indexer**:
- Updates the **Internal Metadata DB** with file name, size, version, and path.
- Passes the file to the **Chunker**.

---

### ✂️ Step 3: Chunker Splits the File
The **Chunker**:
- Breaks the 8MB file into **two 4MB chunks**.
- Calculates **SHA-256 hashes** for deduplication.
- Sends only **new chunks** to be uploaded.

---

### ☁️ Step 4: Upload to the Cloud
1. **Block Server**: Receives file chunks.
2. **Block Storage**: Stores chunks persistently (e.g., Amazon S3).
3. **Metadata Server**: Records:
   - File ID, size, version, path
   - Chunk order and hashes
   - Owner ID and permissions
4. **Metadata DB**: Persists this info for future retrieval.

---

### 🔔 Step 5: Notify Other Devices
The **Synchronization Service**:
- Validates file update using the Metadata DB.
- Publishes an event to the **Request Queue** (shared for all updates).
- Sends update messages to **Response Queues** for Agniva’s phone and tablet.

---

## 📱 Day 2 — Sync on the Go

On the bus, Agniva opens **CloudBox** on his phone.

1. **Long Polling** connection checks his **Response Queue**.
2. The phone receives a notification:  
   > "final_thesis.pdf is available. Chunks: [1,2]"
3. It fetches file metadata from the **Metadata Server** and chunks from the **Block Server**.
4. **Metadata Cache** and **Chunk Cache** speed up access to frequently used files.

---

## ✏️ Day 3 — Editing a File

Agniva edits just one paragraph in `final_thesis.pdf` on his laptop.

1. **Watcher** detects change → **Indexer** updates local metadata.
2. **Chunker** finds only **Chunk 2** changed.
3. Only **Chunk 2** is uploaded to the **Block Server**.
4. Metadata DB version is updated (v2), and devices are notified.
5. The **Sync Service** sends update instructions to phone and tablet via their **Response Queues**.

---

## 📦 Scaling for Millions
To serve **500M users**, CloudBox uses:
- **Message Queues** (Kafka/RabbitMQ/SQS) for async updates.
- **Load Balancers**:
  - Between clients ↔ Block Servers.
  - Between clients ↔ Metadata Servers.
- **Round Robin** or **Smart LB** for efficient request distribution.

---

## 🗂️ Metadata Partitioning
With **billions of files**, Metadata DB is sharded:
- **Vertical**: Split tables by type (user vs file data).
- **Range-based**: Partition by file name range.
- **Hash-based**: Evenly spread FileIDs via hashing (with consistent hashing for rebalancing).

---

## ⚡ Caching for Performance
- **Chunk Cache**: Hot file chunks in memory for fast retrieval.
- **Metadata Cache**: Frequently used file/user info in memory.
- **LRU eviction** to make space for new items.

---

## 🔐 Security & Permissions
Each file in Metadata DB stores:
- **Owner**
- **View/Edit lists**
- **Public flag**
  
Only authorized users can view/edit/download.

---

## 🌍 Why It Works Well
CloudBox achieves:
- **Real-time sync** with minimal bandwidth (thanks to chunking, deduplication, delta uploads).
- **High scalability** via queues, sharding, load balancing.
- **Fast performance** from caching.
- **Data reliability** via redundant, geo-distributed storage.
- **User trust** with ACID-compliant metadata operations and strict permissions.

---

**In short**:  
Every time Agniva adds, edits, or shares a file, CloudBox:
1. Detects changes locally.
2. Uploads only what’s necessary.
3. Updates metadata.
4. Notifies all relevant devices.
5. Scales to millions of users without slowing down.

☁️ *From thesis drafts to photos — CloudBox keeps Agniva’s digital life safe, synced, and accessible anywhere.*

---
---
---

# ☁️ Dropbox-Style Cloud Storage — Quick Revision

## 1. Why Cloud Storage?
- **Availability**: Anywhere, anytime access.
- **Reliability/Durability**: Multiple copies, geo-distributed.
- **Scalability**: Virtually unlimited, pay-as-you-go.

## 2. Core Functional Requirements
- File **Upload/Download**
- **Sharing** (files/folders)
- **Automatic Sync** across devices
- Large file support (GB scale)
- **ACID compliance**
- **Offline Editing** + later sync
- **Versioning/Snapshots**

## 3. Design Considerations
- High **read/write load**, balanced ratio.
- **Chunking** (e.g., 4MB):
  - Retry failed chunks only.
  - Partial updates.
- **Optimizations**:
  - Chunk **Deduplication**
  - **Local metadata cache**
  - **Delta uploads** (diffs only)

## 4. Capacity Estimates (Assumptions)
- Users: 500M total, 100M DAU.
- Files/user: 200 → 100B total.
- Avg file: 100KB → ~10 PB storage.
- 1M active connections/minute.

## 5. High-Level Architecture
<img width="911" height="490" alt="image" src="https://github.com/user-attachments/assets/ad7bfe53-710e-4c3b-b8fa-c97f942f933b" />

- **Clients**: Monitor folder, upload/download, sync.
- **Block Server**: Handles chunks, stores in cloud storage.
- **Metadata Server**: Stores file info, permissions.
- **Sync Server**: Propagates updates across devices.

## 6. Client Components
<img width="1010" height="622" alt="image" src="https://github.com/user-attachments/assets/7dcbb94a-5b8e-4e17-869c-63889d541a42" />

- **Internal Metadata DB**: Stores local file/chunk info, versions, and paths for quick change detection.
- **Chunker**: Splits files into chunks, detects changes, and reassembles files.
- **Watcher**: Monitors workspace folder for file create/update/delete events.
- **Indexer**: Updates metadata, manages chunk upload/download, coordinates with sync service.
- Long Polling for near real-time sync.
- Exponential Backoff for retries.
- Mobile: on-demand sync.

## 7. Metadata DB
- Stores **chunks**, **files**, **users**, **devices**, **workspace**.
- Relational DB → ACID & simpler sync logic.
- Crucial for:
  - File reconstruction
  - Ownership & permissions
  - Version tracking
  - Device update list

## 8. Synchronization Service
- Detects changes, sends modified chunks only.
- Validates with metadata DB.
- Notifies devices via **Message Queues**.
- Scales with Kafka/RabbitMQ/SQS.

## 9. Message Queuing
<img width="1087" height="461" alt="image" src="https://github.com/user-attachments/assets/209f81c2-fc82-41c4-8561-4114e8dcfd49" />

- **Request Queue**: All client updates → Sync Service.
- **Response Queue**: Per client/device → update instructions.
- Benefits: Async, scalable, reliable, loosely coupled.

## 10. Block/Cloud Storage
- Stores actual file chunks.
- Works with caching layer.
- Storage-agnostic (S3, GCS, in-house).

## 11. Metadata Partitioning
- **Vertical**: Split by feature/tables.
- **Range-based**: By name/path.
- **Hash-based**: By FileID hash (+ consistent hashing).

## 12. Caching
- **Chunk Cache**: Hot file chunks in memory.
- **Metadata Cache**: Frequently accessed file/user info.
- Replacement: **LRU**.

## 13. Load Balancing
- **Round Robin**: Simple, fault-tolerant.
- **Smart LB**: Load-aware routing.
- Used between clients ↔ block/metadata servers.

## 14. Security & Sharing
- Metadata stores:
  - Owner
  - View/Edit permissions
  - Public flag
- Access control critical for privacy.

---
**Key Principles**:  
Chunking + Deduplication + Delta Sync → Efficient.  
Metadata/Block separation → Flexible & scalable.  
Queues + Caches + LBs → High performance & availability.
