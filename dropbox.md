# ☁️ Designing Dropbox (Cloud File Storage System)

## 📌 Overview

Let's design a cloud-based file hosting service like **Dropbox** or **Google Drive**.

> Cloud file storage enables users to store their data on remote servers managed by providers and access them via the internet.

**Similar Services**:  
- Google Drive  
- OneDrive  
- iCloud Drive  
- Box
  
---

## 🌐 1. Why Cloud Storage?

Cloud storage has grown rapidly due to its convenience and the shift toward multi-device usage.

### 🔑 Benefits

#### ✅ Availability  
- Data is accessible **anywhere, anytime** from any internet-enabled device. 
#### ✅ Reliability & Durability  
- Cloud storage ensures **data is never lost** by:
  - Storing **multiple copies**.
  - Distributing them across **geographically separated servers**.
#### ✅ Scalability  
- Users don’t need to worry about storage limits.
- **Virtually infinite storage**, pay-as-you-go model.
---

## 📌 2. Requirements and Goals

### 🎯 Core Functional Requirements

1. **File Upload/Download**  
   - Users can upload/download files from **any device**.

2. **Sharing Support**  
   - Users can share **files or folders** with others.

3. **Automatic Sync**  
   - Files updated on one device should **sync across all devices**.

4. **Large File Support**  
   - Must handle **large files** (up to several GBs).

5. **ACID Compliance**  
   - Ensure **Atomicity**, **Consistency**, **Isolation**, and **Durability** in all file operations.

6. **Offline Editing**  
   - Users can **add/delete/modify files offline**.  
   - Changes should sync once the user is back online.

### 🌟 Extended Requirement

- **Versioning / Snapshots**  
  - Support restoring files to **previous versions** (snapshotting).

---

## 📌 3 Design Considerations

- **High Read & Write Load**: Expect large volume on both ends.
- **Read ≈ Write Ratio**: Balanced usage, not read-heavy like many other systems.
### 📦 Chunking Strategy
- **Split files into small chunks** (e.g., **4MB**). ---> Assunption: 4MB 
- Benefits:
  - Retry only **failed chunks**, not whole file.
  - Enables **partial updates** (upload only changed chunks).
### 🧠 Optimization Concepts

- **Chunk Deduplication**:
  - Store identical chunks **once** to save **space & bandwidth**.
> 👤 User A uploads a 100MB video.  
> 👤 User B uploads the **same** video.  
> 📦 Server checks chunks → finds them **identical** → stores them **only once**.  
> ✅ Saves space & avoids redundant uploads.
- **Local Metadata Caching**:
  - Store file name, size, etc. on **client-side** to reduce **server round-trips**.
> 👤 User opens Dropbox app on their phone.  
> 📄 The app **already knows** file names, sizes, last modified time.
> 
> ✅ No need to hit server just to list files → **faster load times**.

- **Delta Uploads (Diffs)**:
  - For **small edits**, send only the **differences**, not entire chunks.
> 👤 User edits a 100MB document → changes **1 paragraph**.  
> Instead of re-uploading 100MB, client sends just the **tiny diff chunk**.  
> ✅ Saves **upload time** and **bandwidth**.

---
## 📏4  Capacity Estimation & Constraints
- **📊 Users**:
  - Total Users: **500M** ---> Assumption
  - Daily Active Users (DAU): **100M** ---> Assumption
  - Devices per user: **~3** ---> Assumption

- **📁 File Estimates**:
  - Avg. files/photos per user: **200** ---> Assumption
  - Total Files: **100B** --> 200files/users * 500M users 
  - Avg. file size: **100KB** ---> Assumption

- **🗄️ Storage Estimate**:
  - `100B files * 100KB` = **10 Petabytes (PB)**

- **🔌 Active Connections**:
  - Up to **1 million** active connections **per minute** ---> Assumption
    
---
## 📦5 High-Level Design – Dropbox

### 📁 Core Concept:
- User selects a **workspace folder** on each device.
- Files in this folder are:
  - ⬆️ Uploaded to **cloud**
  - 🔄 Synced across **all user devices**
  - 🗑️ Updated or deleted files are **propagated to cloud & all devices**

### 🔧 Components Overview:
<img width="911" height="490" alt="image" src="https://github.com/user-attachments/assets/34801dd7-0423-49d1-b478-9b20172a548f" />

#### 🟨 Block Server
- Works with **clients** to handle:
  - File **upload/download**
- Sends data to:
  - ☁️ **Cloud Storage** for actual file chunks
#### 🟧 Metadata Server
- Stores & manages:
  - File **name, size, directory**
  - **Sharing** info (e.g., who can access)
- Writes data to:
  - 🗃️ **Metadata Storage** (DB - SQL/NoSQL)
#### 🟩 Synchronization Server
- Ensures all devices:
  - Receive **updates** after any file action
  - Sync **automatically** across devices

## 📦 Dropbox File Upload Example
### 👨‍💻 Client (Laptop)
- Agniva adds `project_notes.pdf` to the Dropbox folder.
- Client app splits the file into **chunks** and begins upload.
### 🟨 Block Server
- Receives **file chunks** from the client.
- Stores chunks in **Cloud Storage**.

✅ Now the file **data** is safely stored in the cloud.

### 🟧 Metadata Server
Stores metadata like:
- **File name**: `project_notes.pdf`
- **Size**: `1.2MB`
- **Path**: `/school/notes/`
- **Uploaded by**: `agniva123`

→ Saves this to the **Metadata Storage DB**.

📒 Helps identify & retrieve files later by name, user, folder, etc.

### 🟩 Synchronization Server
- Notices a **new file upload**.
- Notifies Agniva’s **tablet** and **phone** clients:

> “Hey! There’s a new file — `project_notes.pdf` — sync it!”

Tablet and phone then:
- Fetch file metadata from **Metadata Server**
- Download file chunks from **Block Server**

✅ Result: Agniva now has `project_notes.pdf` synced across all devices.

---

## ⚙️ 6.1 c- Client

The **Client Application** is the user's local agent that:
- Monitors the workspace folder
- Uploads/downloads files
- Syncs with cloud storage & metadata
- Handles changes, conflicts, and updates

### 🔧 Key Responsibilities of the Client:
1. **Upload & Download Files**
2. **Detect File Changes** (create/update/delete)
3. **Handle Conflicts** (offline or concurrent updates)

### 📦 Efficient File Transfer with Chunks
- Files are split into **4MB chunks**
- Transfer **only modified chunks**, not the entire file
- Benefits:
  - Saves **bandwidth**
  - Improves **IOPS** (Input/Output Operations per Second)
  - Handles **network interruptions** better

### 🗃️ Local Metadata Copy
- ✅ Store file info (name, size, version, etc.) on **client**
- ⏱️ Enables **offline updates**
- 🚀 Reduces **round trips** to metadata server

### 🔁 Change Detection from Other Clients

- ❌ **Polling**: Not scalable — wastes bandwidth and CPU
- ✅ **Long Polling**:
  - Client sends request → Server **waits** until there’s data
  - Sends response once there's an update
  - Client then sends a **new request** for future updates
  - ⏱️ Achieves **near real-time sync**

### 🔍 Components of the Client

#### 🗂️ I. Internal Metadata DB
- Tracks:
  - Files
  - Chunks
  - Versions
  - File locations

#### ✂️ II. Chunker
- Splits files into **chunks**
- Reconstructs full file from chunks
- Transfers **only changed parts** to cloud

#### 👀 III. Watcher
- Monitors **local workspace folder**
- Notifies `Indexer` of:
  - File create/update/delete
- Also **listens to sync messages** from the server

#### 🧠 IV. Indexer
- Processes events from `Watcher`
- Updates **internal metadata**
- Uploads/downloads **chunks**
- Talks to:
  - **Cloud storage** for file data
  - **Synchronization Server** for broadcasting changes

### 🐌 Handling Slow Servers
- Use **Exponential Backoff**:
  - Wait longer between retries if server is slow or unresponsive
  - Reduces pressure on already stressed servers

### 📱 Special Case: Mobile Clients
- Sync changes **on demand**
- Save **bandwidth** and **battery**

✅ **Summary**:
Clients handle all file sync, chunking, and metadata updates locally while coordinating with the backend through efficient protocols like long polling and intelligent chunk management.

<img width="910" height="586" alt="image" src="https://github.com/user-attachments/assets/ba69f3c9-9db3-42a2-bc4f-6531224aeea5" />

## 🔄 Example Walkthrough:
Agniva adds `final_thesis.pdf` (8MB) to their Dropbox folder on their laptop.

### 🧩 Chunker
- Splits the file into **two 4MB chunks**
- Each chunk is **hashed** to detect duplicates

➡️ Passes chunks to **Indexer**

### 📘 Internal DB
- Stores:
  - File name: `final_thesis.pdf`
  - Chunk hashes
  - File version
  - Location on local disk
- Helps detect **local changes** or **conflicts**

### 👁️ Watcher
- Monitors the workspace folder for:
  - New files
  - Updates
  - Deletions
- Detects that `final_thesis.pdf` was added

➡️ Notifies **Indexer**

### 🧠 Indexer
- Updates **internal DB** with chunk info
- Uploads chunks to **Block/Cloud Storage**
- Updates metadata like:
  - Last modified time
  - File version
- Notifies **Synchronization Service** that:
  - A new file has been uploaded
    
### ☁️ Block/Cloud Storage (🟧)
- Receives and stores the **file chunks**
- These chunks are now available in the cloud
- Other devices can **download them on demand**

### 🔁 Synchronization Service (🟨)
- Receives notification of the new file
- Updates the **Metadata DB**:
  - File: `final_thesis.pdf`
  - User: agniva123
  - Size: 8MB
  - Chunk count: 2

➡️ Sends update notifications to **other clients** (tablet, phone)

### 🔗 Control & Data Flow
- **Data Flow**: Chunks move from Chunker → Block Storage
- **Control Flow**: Metadata and file events move through:
  - Watcher → Indexer → Sync Service → Metadata DB
   
## 📦 Summary

| Component           | Role                                              |
|--------------------|---------------------------------------------------|
| Chunker            | Splits/assembles file chunks                      |
| Watcher            | Monitors folder for changes                       |
| Internal DB        | Stores local metadata                             |
| Indexer            | Uploads/downloads chunks, syncs metadata          |
| Block Storage      | Stores actual file data                           |
| Sync Service       | Coordinates cross-device updates                  |
| Metadata DB        | Tracks file info (who/what/where/when)            |

All components work together to ensure **real-time**, **efficient**, and **reliable** syncing across devices!

## 🗂️ 6.2. Metadata Database — Explained with Agniva's Example

Continuing with the earlier example:

Agniva uploads `final_thesis.pdf` (8MB) to Dropbox from his laptop. Here's how the **Metadata Database** comes into play.

## 🔍 What's the Metadata Database?

It stores **information about the file**, not the actual file chunks.

Think of it as a giant spreadsheet that helps Dropbox know:
- Who uploaded what?
- Where is the file stored?
- What chunks form the file?
- What device did the update come from?
  
## 📘 What Does It Store?

### 1. **Chunks Table**
| Chunk ID       | File ID       | Order | Hash                                 |
|----------------|---------------|-------|--------------------------------------|
| chunk_abc123   | file_xyz789   | 1     | `f68de3...`                          |
| chunk_def456   | file_xyz789   | 2     | `32cde1...`                          |

> For `final_thesis.pdf`, there are 2 chunks. The table stores each chunk's hash for **deduplication** and **integrity check**.

### 2. **Files Table**
| File ID     | Name               | Size  | Version | Last Modified      | Owner     |
|-------------|--------------------|-------|---------|---------------------|-----------|
| file_xyz789 | final_thesis.pdf   | 8MB   | v1      | 2025-08-02 10:45AM | agniva123 |

> This lets Dropbox know that Agniva owns the file, it's 8MB in size, and is currently at **version 1**.

### 3. **Users Table**
| User ID     | Name      | Email                 |
|-------------|-----------|------------------------|
| agniva123   | Agniva    | agniva@bitsmail.com    |

> Basic user information used for permissions and audit tracking.

### 4. **Devices Table**
| Device ID    | User ID   | Device Type | Last Sync         |
|--------------|-----------|-------------|--------------------|
| dev_laptop1  | agniva123 | Laptop      | 2025-08-02 10:47AM |
| dev_phone1   | agniva123 | Phone       | 2025-08-02 10:40AM |
| dev_tab2     | agniva123 | Tablet      | 2025-08-02 10:39AM |

> Lets Dropbox track which devices belong to Agniva and when they last synced.

### 5. **Workspace Table**
| Workspace ID | User ID   | Path              |
|--------------|-----------|-------------------|
| ws_001       | agniva123 | /thesis/final/    |

> The workspace tells Dropbox **which folders to sync** across devices.

## 💡 Relational vs NoSQL — What Should We Choose?

### ✅ Relational (e.g., MySQL)
- **Pro**: Native **ACID** support → ensures safe updates when Agniva edits a file on multiple devices at once.
- **Con**: May be harder to scale with billions of files.

### ⚠️ NoSQL (e.g., DynamoDB)
- **Pro**: Highly scalable and fast.
- **Con**: Doesn't support ACID by default → Dropbox will need to add that logic manually (e.g., version checks in sync service).

using a relational database can simplify the implementation of the Synchronization Service 
as they natively support ACID properties.

## 🧠 Summary: Why Metadata DB is Crucial

- Without metadata, Dropbox wouldn’t know:
  - How to **rebuild** a file from its chunks
  - Who a file belongs to
  - What’s changed recently
  - Which devices need to be updated

It's like the **Dropbox brain** 🧠 — tracking everything while the actual files live in cloud storage.

## 🔁 6.3. Synchronization Service

Let’s continue with Agniva's file upload journey to Dropbox — now focusing on how **file changes are synced** across devices.

## 📂 Scenario:

Agniva modifies his `final_thesis.pdf` on his **laptop** at 11:00 AM.

He had earlier uploaded this file, and it was already available on his **tablet** and **phone**.

What happens next?

### 🧠 Enter the Synchronization Service

As soon as the **client app on Agniva's laptop** detects the file change:

1. The **Watcher** detects the edit in the Dropbox folder.
2. The **Chunker** breaks the modified file into 4MB chunks.
3. Only the modified chunks are identified (say, **Chunk 2** was changed).
4. The **Indexer** updates the local internal metadata DB.
5. Then the update is sent to the **Synchronization Service**.

### 🔁 Sync Service Workflow

The **Synchronization Service**:

1. **Validates** the update using the Metadata DB (checks version consistency).
2. Uses **SHA-256 hashes** to confirm if Chunk 2 is new or already exists (to apply deduplication).
3. Uploads only the **modified chunk** to Cloud Storage (not the entire file).
4. Sends **notifications** to Agniva’s other devices (tablet & phone):
   > “Hey! `final_thesis.pdf` was updated. Sync required.”

### 🔔 Tablet & Phone Sync

When Agniva’s **tablet and phone** come online:

- They **receive the notification**.
- Request the updated metadata and chunks from:
  - **Metadata DB** (via Sync Service)
  - **Block Server** (to download new chunk)
- Their local copies of the file get updated.

## 📉 Why Transmit Only Differences?

Imagine `final_thesis.pdf` is 8MB and Agniva only changed one sentence.

Instead of sending the whole file:

- Only the **changed 4MB chunk** is transmitted.
- Sync time and bandwidth is drastically reduced.

📉 Saves:
- Cloud bandwidth
- User data (especially on mobile)
- Server IOPS

## 📬 How Does It Scale to Millions?

To manage millions of users like Agniva, we use **communication middleware** (like Kafka, RabbitMQ, etc.).

- Clients publish update requests to a **global queue**.
- Multiple instances of the Sync Service **consume messages** in parallel.
- Each Sync Service instance:
  - Pulls update events
  - Processes metadata validation
  - Sends notifications to relevant devices

This ensures high **availability**, **low latency**, and **scalability** 🚀.

## 🧠 Summary

| Role                     | Action                                                                 |
|--------------------------|------------------------------------------------------------------------|
| Client (Agniva’s laptop) | Detects & sends updated chunks                                         |
| Sync Service             | Verifies, deduplicates, updates Metadata DB, notifies other devices    |
| Tablet & Phone           | Pull metadata + modified chunks to update local copy                   |
| Middleware               | Scales message flow across distributed Sync Service instances          |

## 🧪 Example Timeline

| Time        | Event                                            |
|-------------|--------------------------------------------------|
| 11:00 AM    | Agniva edits file on laptop                      |
| 11:01 AM    | Modified chunk sent to Sync Service              |
| 11:02 AM    | Sync Service validates & updates metadata        |
| 11:02:30 AM | Tablet and phone are notified                    |
| 11:03 AM    | Tablet pulls update and syncs                    |

## 📬 6.4. Message Queuing Service — Explained with Agniva’s Example

Let’s continue Agniva’s story to understand how Dropbox-style systems **scale to millions of users** and sync requests in real time using message queues.

### 🧠 Context

Agniva has:
- A **laptop** (where he edited `final_thesis.pdf`)
- A **tablet**
- A **phone**

When he updates the file, this change needs to be:
- Sent to the backend (**asynchronously**)
- Processed by the **Synchronization Service**
- Notified to his **other devices**

How is that achieved efficiently? 👉 Through **Message Queues**.

## 📨 Components of the Message Queuing System

<img width="1087" height="461" alt="image" src="https://github.com/user-attachments/assets/95446204-a1c8-48dc-8335-8a13d538cc58" />

### 1. **Request Queue** (🟡 Global, Shared)
- All clients (like Agniva’s laptop) **send update events** to this queue.
- Think of it like a **giant inbox** for the Synchronization Service.

📦 **Example**:  
Agniva’s laptop sends this message:
```json
{
  "user": "agniva123",
  "file": "final_thesis.pdf",
  "modified_chunk": 2,
  "timestamp": "11:00 AM"
}
```

It lands in the **Request Queue**.

🛠 The Synchronization Service picks it up, verifies it with Metadata DB, and processes it.

### 2. **Response Queues** (📥 Per Client/Device)

* Each **subscribed device** (like Agniva’s tablet and phone) has its **own response queue**.
* Sync Service sends **notifications** to these queues.

📨 **Message to Agniva’s phone queue**:

```json
{
  "file": "final_thesis.pdf",
  "action": "modified",
  "chunk_ids": [2],
  "by": "agniva123",
  "at": "11:00 AM"
}
```

✅ When Agniva’s phone comes online, it **checks its response queue**, sees the message, and syncs the file.

🧹 Once read, the message is **deleted from that queue**, so it doesn’t go to other clients mistakenly.

## 🧩 Why Use Message Queues?

* ✅ **Asynchronous** — clients don’t need to wait for Sync Service response.
* 📈 **Scalable** — queues can handle **millions** of messages.
* 🔁 **Reliable** — messages persist until processed.
* 📬 **Loosely Coupled** — clients and Sync Service don’t need direct contact.

## 🔁 Example Flow Recap

| Step | Actor                | Action                                                       |
| ---- | -------------------- | ------------------------------------------------------------ |
| 1    | Laptop client        | Sends file update → `Request Queue`                          |
| 2    | Sync Service         | Picks from `Request Queue`, updates metadata, processes file |
| 3    | Sync Service         | Sends change → `Response Queues` (tablet + phone)            |
| 4    | Tablet/Phone clients | Poll their queue, get update message, sync modified chunk    |

## 🧠 Summary Table

| Queue Type     | Scope      | Purpose                       | Example                       |
| -------------- | ---------- | ----------------------------- | ----------------------------- |
| Request Queue  | Shared     | Client → Sync Service updates | Laptop update message         |
| Response Queue | Per Client | Sync Service → Notify updates | Tablet/Phone sync instruction |

## ⚙️ Real-world Analogs

| Tech Example | Usage                        |
| ------------ | ---------------------------- |
| **Kafka**    | High throughput pub-sub      |
| **RabbitMQ** | Reliable queue w/ ack system |
| **AWS SQS**  | Scalable cloud message queue |

📦 Thanks to **message queues**, Dropbox can efficiently manage **millions of device updates** without being overwhelmed or tightly coupled.

## 6.5 Cloud/Block Storage 

Cloud/Block Storage stores chunks of files uploaded by the users. Clients directly 
interact with the storage to send and receive objects from it. Separation of the 
metadata from storage enables us to use any storage either in the cloud or in-house.

<img width="1505" height="711" alt="image" src="https://github.com/user-attachments/assets/29e29388-6c79-4afb-be4c-4f369260313b" />

## 🗂️ Dropbox-style Architecture Diagram — Explained

The diagram illustrates the **end-to-end architecture** of a Dropbox-like file synchronization and storage system. Let’s break it down, step-by-step, using Agniva’s interaction with Dropbox.

### 👨‍💻 Clients

- **Web Client**, **Mobile Client**, **Desktop Client**
  - These are the devices Agniva uses to interact with Dropbox.
  - All clients connect to the backend **via the cloud** (represented by the cloud icon).
  - Example: Agniva uploads `research_notes.pdf` from his laptop.

### ☁️ Cloud Gateway

- Acts as a **central access point**.
- Routes incoming client requests to the appropriate backend services via **load balancers**.
- Handles REST API calls, user authentication, etc.

### 🧰 Load Balancers

- Distributes requests evenly across multiple backend servers.
- Ensures **scalability** and **fault tolerance**.
- Example: Agniva’s file upload request is routed to one of the available **Block Servers** or **Metadata Servers**.

## 🔩 Core Components

### 📦 Block Server

- Responsible for storing **actual file chunks**.
- Handles upload/download of data between clients and storage.
- Talks directly to **Block/Cloud Storage** for persistent storage.
- Example: Agniva’s file is split into 4MB chunks and stored here.

### 🧠 Metadata Servers

- Manage all metadata: file paths, ownership, versions, timestamps.
- Communicate with the **Metadata DB** and cache layer.
- Work with the **Notification Server** to sync changes across devices.

### 🗃️ Metadata DB

- Stores structured metadata like:
  - File names and hierarchy
  - Ownership
  - Versions and timestamps
  - User and device details

### ⚡ Metadata Cache Server

- Improves performance by storing **frequently accessed metadata** in memory.
- Example: If Agniva frequently opens his "Assignments" folder, metadata for that folder is served quickly from here.

### ⚡ Storage Cache Server

- Caches **frequently accessed file chunks**.
- Reduces load on primary block storage.
- Example: Agniva opens the same PDF on multiple devices — served from here for speed.

### 📤 Block/Cloud Storage

- The **long-term file chunk storage**.
- Scalable storage backend (e.g., Amazon S3, Google Cloud Storage).
- Used when the cache layer doesn’t have the required file chunk.

### 📣 Notification Server + 🔁 Synchronization Queue

- When a file is updated on one device:
  1. The **Notification Server** receives the event.
  2. Pushes an update to the **Synchronization Queue**.
  3. Notifies **other devices** (e.g., Agniva’s tablet and phone) to sync the changes.
- Ensures **eventual consistency** across all clients.

## 🔁 Full Flow (Agniva uploads a file)

| Step | Component           | Action                                                      |
|------|---------------------|-------------------------------------------------------------|
| 1    | Client              | Agniva uploads `research_notes.pdf`                         |
| 2    | Cloud Gateway       | Forwards request                                            |
| 3    | Load Balancer       | Routes to a Block Server and Metadata Server               |
| 4    | Block Server        | Stores file chunks in Block/Cloud Storage                  |
| 5    | Metadata Server     | Updates file metadata in Metadata DB                       |
| 6    | Metadata Cache      | Frequently accessed metadata cached                        |
| 7    | Notification Server | Sends change event via Synchronization Queue               |
| 8    | Tablet & Phone      | Receive sync notification, fetch updated chunks            |

## ✅ Key Benefits of This Architecture

- ⚖️ **Scalability** — load balancers and distributed services
- ⚡ **Performance** — caching layers
- 🔄 **Real-time Sync** — notification + sync queue
- 📦 **Efficient Storage** — block-level file management
- 🔐 **Separation of Metadata & Data** — optimized access paths

---

## 📁 7. File Processing Workflow — Simple Example

👤 **Client A**: Agniva (on laptop)  
👥 **Clients B & C**: Agniva’s tablet and phone

Agniva is editing a shared file: `writeup.txt`

### 📌 Step-by-step Workflow

1. ✅ **Chunk Upload**
   - Agniva edits `writeup.txt` and saves changes.
   - Laptop (Client A) uploads modified chunks to **Cloud Storage**.

2. 🧠 **Metadata Update**
   - The client sends updated metadata (file version, chunk info, timestamp) to the **Metadata Server**.
   - Changes are committed to the **Metadata DB**.

3. 📬 **Send Notifications**
   - Once confirmed, the **Synchronization Service** sends update notifications via **Message Queuing Service**.
   - Clients B and C (tablet, phone) get individual messages in their **response queues**.

4. ⬇️ **Receive + Sync**
   - When the tablet and phone come online:
     - They poll their response queues.
     - Fetch the latest metadata.
     - Download only the updated chunks from **Block Server**.
