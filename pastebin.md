# 📝 Pastebin System Design — Short Notes

## 📌1.  What is Pastebin?
<img width="955" height="570" alt="image" src="https://github.com/user-attachments/assets/daf2a14a-536f-4804-b035-6f1d62d38df3" />
https://pastebin.com/QhTBy7Gw

*Pastebin is a simple service where users can store and share plain text via a unique URL.*

### 🔗 Example Workflow:
1. User pastes some text (e.g., code snippet or notes).
2. System stores it in a database.
3. System returns a **short unique URL** (e.g., `pastebin.com/xyz123`).
4. Anyone with that URL can **view the text**.

---

## 🧠 Why is it useful?
- Easy way to **share logs, code, notes**.
- Just send a **link** instead of large text.
- Popular for **temporary sharing**.
