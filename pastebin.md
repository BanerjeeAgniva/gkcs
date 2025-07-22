# 📝 Pastebin System Design

## 📌1.  What is Pastebin?
<img width="955" height="570" alt="image" src="https://github.com/user-attachments/assets/daf2a14a-536f-4804-b035-6f1d62d38df3" />
https://pastebin.com/QhTBy7Gw

*Pastebin is a simple service where users can store and share plain text via a unique URL.*

### 🔗 Example Workflow:
1. User pastes some text (e.g., code snippet or notes).
2. System stores it in a database.
3. System returns a **short unique URL** (e.g., `pastebin.com/xyz123`).
4. Anyone with that URL can **view the text**.

## 🧠 Why is it useful?
- Easy way to **share logs, code, notes**.
- Just send a **link** instead of large text.
- Popular for **temporary sharing**.
---
# ✅ 2 Requirements
> *Requirements can be functional/non-functional/extended*  
> *Functional:* User can log in with email and password — *What the system does*  
> *Non-Functional:* Login response must be under 1 second — *How well it does it*  
> *Extended:* System must comply with GDPR — *External rules/constraints (e.g., legal, regulatory)*
## ⚙️ Functional Requirements
- Upload/paste text → get **unique URL**
- **Text only**, no images/files
- **Auto-expiry** + user-defined expiration time
- **Custom alias** support for URL (optional)
## 📈 Non-Functional Requirements
- **High reliability** → no data loss
- **High availability** → service always up
- **Low latency** → fast access to pastes
- **Unpredictable URLs** → secure + non-guessable
## 🧩 Extended Requirements
- **Analytics** → track paste views
- **REST API support** → external programmatic access
---
# 📌 3 Pastebin Design Considerations
- ✅ Similar to URL Shortener but with extra features
- 📏 **Max paste size**: limit to **10MB** to prevent abuse
- 🔗 **Custom URL limits**: apply length limit for consistency in DB
- 🆓 Custom URLs are **optional**, not required
---
# 📊 4 Capacity Estimation & Constraints
## 📈 Traffic Assumptions
- **Write (New Pastes/day)**: 1M ---> Assumption1
- **Read:Write Ratio** → 5:1 (R/W) ---> Assumption2
- **Reads per Day** -> 1M * R/W = 5M
## ⏱️ Per Second Estimates
- **Pastes/sec**:  = 10^6/(24*3600) = ~12 ---> Writes per Sec WPS
- **Reads/sec**: ~58  ----> WPS*R/W
##  Storage Estimates
- **Avg Paste Size**: 10KB  ----> ASsumption3 (SizeObj)
- **Daily Storage**: 1M * 10KB = 10 GB  
- **10 Years Total**: 36 TB  
- **Keys (Base64 encoded)**:  
  - 64⁶ = ~68.7B unique keys  
  - 3.6B * 6 bytes = ~22 GB  
- **Data for 10 years**:  
  `36 TB` (pastes) + `22 GB` (keys) ≈ `36.022 TB`
- **Target: Use only 70% of total capacity**  ----> Assumption4 we are using a 70% capacity model 
0.7 × Total_Storage = 36.022 TB
=> Total_Storage = 36.022 / 0.7 ≈ 51.46 TB
- **70% Capacity Model**: ~51.4 TB (buffered) 
## 📡 Bandwidth Estimates
> *📶 **Bandwidth** is the maximum amount of data that can be transferred over a network in a given time. Higher bandwidth means faster data transfer.*
> *📥 **Ingress** refers to data **coming into** the system (e.g., user uploads).*
> *📤 **Egress** refers to data **leaving** the system (e.g., user downloads or reads).*
- **Ingress (Writes)**:  
  - 12 * 10KB = 120 KB/s  ---> WPS * SizeObj
- **Egress (Reads)**:  
  - 58 * 10KB = 0.6 MB/s  ---> Ingress * R/W
## 🧠 Memory (Cache)
- **80-20 Rule**: Cache 20% of hot reads  
- **0.2 * 5M * 10KB** = ~10 GB cache

| **Metric**               | **Estimate**                                    | **Formula / Notes**                         |
|--------------------------|------------------------------------------------|----------------------------------------------|
| New Pastes/sec           | 12                                             | Writes Per Second WPS                        |
| Read/Write Ratio         | 5:1                                            | Read/Write R/W                               |
| Paste Reads/sec          | 58                                             | RPS = WPS × R/W                              |
| Size per Paste           | 10 KB                                          | Avg size of pasted text                      |
| Ingress                  | 120 KB/sec                                     | WPS × Size per Paste                         |
| Egress                   | 0.6 MB/sec                                     | RPS × Size per Paste                         |
| Storage (10 years)       | ~36 TB (raw) / ~51.4 TB (with buffer)          | 1M/day × 10KB × 365 × 10 / 0.7 (70% buffer)  |
| Cache Memory (per day)   | ~10 GB/day                                     | 20% × 5M × 10KB                              |
---
## 🧩 System APIs — Pastebin
### 🔸 addPaste()
- **Params**:  
  - `api_dev_key` → user auth / quota  
  - `paste_data` → text content  
  - `custom_url` → optional alias  
  - `user_name` → optional for vanity URLs  
  - `paste_name` → optional  
  - `expire_date` → optional expiry  
- **Returns**:  
  - URL (success)  
  - error code (failure)
### 🔸 getPaste()
- **Params**:  
  - `api_dev_key`  
  - `api_paste_key` → unique ID for the paste 
- **Returns**:  
  - paste text
### 🔸 deletePaste()
- **Params**:  
  - `api_dev_key`  
  - `api_paste_key`  
- **Returns**:  
  - `true` (success)  
  - `false` (failure)
---
