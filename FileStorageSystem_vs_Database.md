# Difference Between File Storage System and Database

Both **file storage systems** and **databases** are used to store data, but they serve **very different purposes** and are optimized for **different types of data** and **access patterns**:
<img width="1169" alt="Screenshot 2025-05-17 at 1 00 14 AM" src="https://github.com/user-attachments/assets/b07d2381-c19f-4984-8abc-175574387493" />
| Feature | **File Storage System** | **Database** |
|--------|-------------------------|--------------|
| **Purpose** | Stores **unstructured** data like images, videos, and documents | Stores **structured** data (organized in rows/columns or key-value pairs) |
| **Data Format** | Files (e.g., `.jpg`, `.mp4`, `.pdf`) | Tables (SQL), documents (MongoDB), key-value pairs (Redis), etc. |
| **Querying** | Cannot be queried natively; usually accessed via file paths or APIs | Supports complex querying (e.g., SQL queries, search, filtering) |
| **Access** | Accessed via filesystem paths or object storage APIs (e.g., S3, HDFS) | Accessed via query languages (e.g., SQL, NoSQL queries) |
| **Examples** | Amazon S3, Google Cloud Storage, HDFS, local disk | MySQL, PostgreSQL, MongoDB, Cassandra |
| **Use Cases** | Storing photos, videos, logs, backups, large media files | Storing user data, tweet metadata, follower graphs, likes, etc. |

---

### 🔹 In the Context of Twitter

- **Database**: Stores all tweet metadata (text content, timestamps, likes, retweets, user info).
- **File Storage System**: Stores photos, GIFs, or videos attached to tweets, as they are too large and unstructured for typical databases.
