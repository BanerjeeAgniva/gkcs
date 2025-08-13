# Facebook Messenger 

## 1 Requirements 

## a Functional Requirements
- **One-on-one conversations** between users.
- Track **online/offline statuses** of users.
- **Persistent storage of chat history**.

## b Non-functional Requirements
- Real-time **chat** with **minimum latency**.
- **Highly consistent** chat history across all devices.
- **High availability** is desirable, but can **tolerate lower availability** for better consistency.

## c Extended Requirements
- **Group Chats**: Support multiple participants in a conversation.
- **Push notifications**: Notify users of **new messages** when they are **offline**.

---

## 2 Capacity Estimation & Constraints
<img width="307" height="251" alt="image" src="https://github.com/user-attachments/assets/0b4a4fe0-3b23-465c-89b1-08ca3595bd59" />

### Assumptions
- **Daily Active Users (DAU)**: 500 million.  
- **Avg Messages per User/day**: 40.  
- **Avg Message Size**: 100 bytes.  
- **Retention Period**: 5 years of chat history.  
- **Data Transfer Rule**: Each incoming message is delivered to one other user.  
- **Exclusions**: No compression or replication factored in.  
- **Additional Storage Needs**: User profile data + message metadata (ID, timestamp, etc.). 

### Calculations
- **Total Messages/day**: 500M × 40 = **20 billion**.  
- **Storage/day**: 20B × 100B = **2TB/day**.  
- **5-year Storage**: 2TB × 365 × 5 ≈ **3.6PB**.  

### Bandwidth
- **Incoming Data Rate**: 2TB ÷ 86,400s ≈ **25MB/s**.  
- **Outgoing Data Rate**: Same as incoming (25MB/s).

---

<img width="1462" height="662" alt="image" src="https://github.com/user-attachments/assets/857541bc-9edc-45ee-8325-b40a13cb9b28" />

## 3 High-Level Chat Flow (Steps 1–9)

1. **User A → Server A**: Sends message to User B.  
2. **Server A → User A**: Sends ACK (message received).  
3. **Server A → DB**: Stores message in database.  
4. **Server A → Server B**: Passes message to server handling User B.  
5. **Server B → Server A**: Sends acknowledgement.  
6. **Server B → User B**: Sends message to User B.  
7. **User B → Server B**: Sends message receive acknowledgement.  
8. **Server B → Server A**: Confirms delivery to User B.  
9. **Server A → User A**: Sends delivery confirmation to User A.

---

## 4 Detailed Component Design — Single Server

### Use Cases
a. Receive and deliver messages.
b. Store/retrieve messages from DB.
c. Track user online/offline status & notify relevant users.

## a. Message Handling

**Sending/Receiving**
- Users connect to server to send messages.
- **Two models:**
  1. **Pull**: Users periodically check server → frequent empty responses, high resource waste.
  2. **Push**: Keep open connection (low latency, instant delivery).

**Maintaining Open Connections**
- Use **HTTP Long Polling** or **WebSockets**.
- Long polling: client request held until data available; reconnect on timeout/disconnect.

**Tracking Connections**
- Server keeps hash table: `UserID → connection object`.

**Offline Users**
- If disconnected, notify sender of delivery failure.
- Temporary disconnect → expect reconnect & retry.
- Optionally store message for later delivery.

**Scaling Connections**
- Plan for **500M** connections.
- One server handles ~50K connections → need **10K servers**.
- **Software load balancer** maps UserID to server.

**Message Delivery Steps**
1. Store in DB (can be async).
2. Send to receiver via server holding connection.
3. Acknowledge sender immediately.

**Maintaining Sequence**
- Store timestamp on arrival (not enough for global ordering).
- Maintain **per-user sequence numbers** for consistent device ordering.
