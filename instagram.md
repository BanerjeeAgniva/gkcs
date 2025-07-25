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
