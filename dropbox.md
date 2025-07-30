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

