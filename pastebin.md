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
