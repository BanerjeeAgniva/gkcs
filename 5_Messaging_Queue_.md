# Messaging Queues - System Design

## **Introduction**
Messaging queues help in handling asynchronous communication, allowing tasks to be distributed and processed efficiently without making clients wait for immediate responses.

messaging queues are durable components stored in memory that support asynchronous communication 
---

## **Example: Pizza Shop**
### **Order Handling in a Pizza Shop**
- A pizza shop takes orders from multiple clients without stopping pizza-making.
- Customers receive an immediate response (e.g., "Please sit down" or "Come back later") instead of waiting for the pizza to be made.
- Orders are stored in a **queue** (list of pending orders).
- Orders are processed based on priority, and pizzas are made asynchronously.
- The customer pays after receiving the pizza, ensuring smooth operations.

### **Benefits of Asynchronous Processing**
- Clients are free to do other tasks instead of waiting (e.g., checking their phone, going out).
- Pizza makers can prioritize tasks (e.g., making urgent orders first, serving drinks quickly).
- More efficient resource allocation for both clients and the business.

---

## **Scaling the Shop Horizontally**
- As the shop becomes successful, it expands to multiple locations (e.g., **Pizza Shop 1, 2, and 3**).
- Each shop serves its own set of customers.

### **Fault Tolerance in Shops**
- If a shop fails (e.g., power outage), takeaway orders are canceled.
- Delivery orders can be reassigned to other shops.
- Need for **persistent storage** (database) to track orders.

---

## **Message Queue Architecture**
### **Database for Order Persistence**
- Orders are stored in a database with details like:
  - Order ID
  - Order Contents
  - Completion Status (Done/Not Done)
- Helps in redistributing orders if a server fails.

### **Failure Handling**
- **Servers (S1, S2, S3, S4)** process orders.
- If **S3 crashes**, orders assigned to it must be reassigned.
- Database helps retrieve unfinished orders.
- A **Notifier** checks server status (heartbeat mechanism every 15s).
- If a server doesn’t respond, its orders are reassigned to other servers.

### **Avoiding Order Duplication**
- Orders reassigned from a failed server **must not** be duplicated.
- **Load Balancing** ensures even distribution and prevents duplicate processing.
- **Consistent Hashing** is used for efficient load balancing:
  - Each server handles a fixed set of orders.
  - If a server crashes, its orders are redistributed without affecting others.

---

## **Messaging Queues as a Solution**
### **Features of a Messaging Queue (Task Queue)**
- Stores tasks (orders) persistently.
- Assigns tasks to available servers.
- Waits for acknowledgment of task completion.
- If a server fails, it reassigns tasks to another server.
- Encapsulates complexity, making task management simpler.

### **Encapsulation in System Design**
- Using a **task queue** simplifies server-side logic.
- **Example of Messaging Queues:**
  - **RabbitMQ** (widely used message broker)
  - **ZeroMQ** (lightweight messaging library)
  - **JMS (Java Messaging Service)**
  - **Amazon SQS (Simple Queue Service)**

<img width="1416" alt="Screenshot 2025-02-21 at 3 05 48 AM" src="https://github.com/user-attachments/assets/740df9d6-d379-4693-9a4f-f80580b204d1" />

<img width="1345" alt="Screenshot 2025-02-21 at 3 11 23 AM" src="https://github.com/user-attachments/assets/5bded38a-585b-4813-8c3b-a20db986503a" />

- Producer vs Consumer 
   - producer -> customer -> produces requests.
   - consumer -> pizzashop -> consumes requests.
   - message queues -> queues these requests from producer and then sends to consumer based on PUSH/PULL (read below)
     
---

<img width="1323" alt="Screenshot 2025-02-21 at 3 24 16 AM" src="https://github.com/user-attachments/assets/3bc5d1c6-0d68-4155-bb02-b992de55818f" />

## Order Processing in a Message Queue

1. When an order is placed:  
   - The order details are encapsulated into a **message**.  
   - This message is then added to the **queue**.  

2. The queue contains messages with **order details** inside them.  

3. Separate processes, called **workers**, handle these messages:  
   - Workers **pull** messages from the **front** of the queue.  
   - Each worker processes the assigned task.  

4. Processing flow:  
   - The server **acknowledges** that it has received and processed the message.  
   - The queue then **removes** this message to prevent duplicate processing.
  
   ---


<img width="1382" alt="Screenshot 2025-02-21 at 3 12 17 AM" src="https://github.com/user-attachments/assets/b006c281-d2fe-4beb-8d60-aa1729e25f1c" />

### **decoupling**

<img width="1315" alt="Screenshot 2025-02-21 at 3 13 30 AM" src="https://github.com/user-attachments/assets/a373822e-7db5-4747-9231-3ce399dd65d4" />
<img width="1331" alt="Screenshot 2025-02-21 at 3 13 56 AM" src="https://github.com/user-attachments/assets/d2e3a530-a668-4a23-9d47-227ee31c926b" />
<img width="1308" alt="Screenshot 2025-02-21 at 3 14 18 AM" src="https://github.com/user-attachments/assets/ca7adf55-f524-483a-bb6b-fe3755cd1fe6" />

data is stored not in a ram, but in a disk which is persistent 

### **Types of Queues**
<img width="748" alt="Screenshot 2025-02-21 at 3 14 44 AM" src="https://github.com/user-attachments/assets/601f4919-50d1-4384-9462-e9c98eec4415" />

### **Push Pull Legs**
some queues are pool based meaning they wait for workers or consumers to pull messages from this queue and on the other hand some of them are push based meaning they actively send messages to the workers to be processed 
<img width="1283" alt="Screenshot 2025-02-21 at 3 15 21 AM" src="https://github.com/user-attachments/assets/d0391541-4393-4d75-86ae-4c6c19a77f5d" />


