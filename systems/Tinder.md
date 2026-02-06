# System Design: Tinder Architecture

A high-level conceptual overview of the core components required to build a scalable dating application.

---

## 1. System Requirements
To ensure a seamless user experience, the architecture focuses on four primary pillars:
* **Profile Management:** Storing user data and media.
* **Recommendation Engine:** Suggesting relevant users based on location.
* **The Matcher:** Handling swipes and identifying mutual interests.
* **Direct Messaging:** Real-time chat functionality between matched users.

---

## 2. Profile & Media Storage
While text-based metadata is stored in traditional databases, image storage requires a specialized approach.

### BLOB vs. Distributed File Storage
* **BLOB (Binary Large Objects):** Storing images directly in a relational database often leads to performance bottlenecks and massive backup sizes.
* **Distributed File Architecture:** We utilize services like **Amazon S3** or **Google Cloud Storage**. This is more cost-effective and allows for the use of a **Content Delivery Network (CDN)** to serve images with low latency globally.



---

## 3. Recommendation Engine & Data Partitioning
To handle millions of users, the system must be able to query candidates near a specific location quickly.

### Geospatial Sharding
We partition user data based on geography (**City-wise partitioning**) using **NoSQL** databases like **Cassandra** or **Amazon DynamoDB**. 
* **Horizontal Partitioning (Sharding):** Ensures that as the user base grows in a specific region, we can add more nodes to handle the load without affecting global performance.
* **Geofencing:** Queries are limited to specific latitude/longitude bounds to filter the "pool" of potential matches.



---

## 4. Swiping & The Matcher Microservice
The system must remain decoupled to ensure high availability.

### Client-Side Optimization
To prevent showing the same user twice:
* **Bloom Filters:** Used on the client or edge to efficiently track "seen" or "rejected" profiles without taxing the primary database.

### The Matcher Service
1.  When a user swipes, the action is sent to the **Matcher Microservice**.
2.  The service validates the action against a fast-access cache (e.g., **Redis**).
3.  If a mutual "Right Swipe" is detected, a **Match Event** is triggered, enabling the chat functionality between the two users.

---

## 5. Direct Messaging (Chat Service)
Real-time communication is built on persistent connections rather than standard HTTP polling.

* **Protocol:** **XMPP** or **WebSockets** over **TCP**.
* **Mechanism:** WebSockets allow for a bi-directional, persistent connection between the client and the server.
* **Session Mapping:** The system maintains a mapping of `User_ID` to `Active_Connection_ID`. When a message is sent, the **Session Microservice** routes it directly to the receiver's active socket.



---

## 6. High-Level Summary
* **Architecture:** Microservices-based.
* **Communication:** Decoupled via Message Queues (for matches) and WebSockets (for chat).
* **Scalability:** Achieved through Geospatial sharding and Distributed File Systems.

> **Note:** This is a conceptual design for interview purposes and does not represent the actual internal implementation of Tinder.

---

![Alt Text](/images/systemImg/_-%20visual%20selection%20(4).png)