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

## ⚙️ 6.1 Component Design - Client

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
