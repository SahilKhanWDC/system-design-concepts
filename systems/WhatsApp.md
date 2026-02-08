# System Design: WhatsApp Architecture

Designing a WhatsApp-style architecture is a classic "High-Level Design" (HLD) challenge. This breakdown focuses on scalability, real-time communication, and data integrity.

---

## 1. High-Level Architecture
WhatsApp uses a **Client-Server-Client** model. Because mobile devices aren't always online, the server acts as a reliable middleman.



* **Communication Protocol:** Use **WebSockets** for bi-directional, real-time communication. Unlike HTTP, WebSockets keep a connection open, which is essential for low-latency "typing..." indicators and instant message delivery.
* **Gateway Service:** Manages millions of active WebSocket connections and routes messages to the correct user sessions.

---

## 2. Core Feature Implementation

### Message Delivery Flow
1.  **User A** sends a message to the Chat Service.
2.  The service persists it in the database and attempts to push it to **User B**.
3.  If **User B** is offline, the message is queued in a **Message Store** (e.g., Cassandra) until they reconnect.

### Read Receipts (The "Ticks")
These are handled as "control messages." 
* **Single Tick:** Server received the message.
* **Double Tick:** Server delivered the message to the recipient's device.
* **Blue Tick:** The recipient's app sent an `ACK` (acknowledgment) back to the server indicating the message was rendered on screen.

### Presence Service (Last Seen/Online)
Managed by a heartbeat mechanism where the client pings the server every few seconds. This data is stored in an in-memory cache like **Redis** for sub-millisecond lookups.

### Group Messaging
A **Group Service** maintains the membership list. When a message is sent to a group, the server replicates that message for every active member in that list.

---

## 3. Non-Functional Requirements (NFRs)

### Message Ordering & Sequence
In distributed systems, messages can arrive out of order.
* **Solution:** Use **Client-side Timestamps** combined with a **Server-side Sequence Number**. 
* The database uses a sort key based on these timestamps to ensure the UI renders them in the correct order.

### Idempotency & Retrials
* **Idempotency:** To prevent duplicate messages during network flickers, the client generates a **UUID (Message ID)**. The server checks this ID; if it already exists, the second request is ignored.
* **Retrials:** The client uses **Exponential Backoff** to retry failed sends without DDOSing the server.

### Load Balancing & Scalability
* **Consistent Hashing:** Distributes WebSocket connections across multiple servers. If one server fails, only a specific subset of users is disconnected.
* **Database Sharding:** Shard the database by `User_ID` or `Chat_ID` to prevent any single node from becoming a bottleneck.

### Image/Media Sharing
Large files are never sent over WebSockets.
1.  **Upload:** Client uploads the image to an **S3 Bucket** and gets a URL.
2.  **Transfer:** The client sends only the **URL/Metadata** as a text message.
3.  **Download:** The receiver fetches the image from the URL via a **CDN** (Content Delivery Network).

---

## 4. The Tech Stack Summary

| Component | Technology | Reason |
| :--- | :--- | :--- |
| **Database** | Cassandra / DynamoDB | Optimized for high write throughput and horizontal scaling. |
| **Cache** | Redis | Fast lookups for "Last Seen" and session status. |
| **Real-time** | WebSockets (WSS) | Persistent, low-latency, bi-directional connection. |
| **Storage** | Amazon S3 + CloudFront | Efficiently handles globally distributed media files. |

---
![alt_text](/images/systemImg/_-%20visual%20selection%20(6).png)