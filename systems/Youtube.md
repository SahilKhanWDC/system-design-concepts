# System Design: YouTube at Scale (100M DAU)

This repository contains a structured breakdown of the system architecture, capacity planning, and bottleneck analysis for a global video-sharing platform.

---

## 1. Requirements

### Functional
* **Viewers:** Low-latency playback, adaptive bitrate streaming (compatibility), and high availability.
* **Creators:** Simple upload via **Presigned URLs**, efficient file transfer, and processing notifications.

### Non-Functional
* **Scale:** 100 Million Daily Active Users (DAU).
* **Read/Write Ratio:** 100:1 (Heavy Read).
* **Data Retention:** 10 Years.
* **Availability:** 99.99% (High Availability).

---

## 2. Capacity Planning & Estimation

### QPS (Query Per Second)
| Metric | Calculation | Result |
| :--- | :--- | :--- |
| **Write QPS** | $100\text{M} / 86400$ | **~1,157 writes/sec** |
| **Read QPS** | $10\text{B} / 86400$ | **~115,740 reads/sec** |

### Storage Requirements
* **Daily Video Data:** $100\text{M uploads} \times 500\text{MB} = 50\text{ PB/day}$.
* **10-Year Raw Storage:** $50\text{ PB} \times 365 \times 10 \approx$ **183 EB (Exabytes)**.
* **Total footprint (incl. Transcoding & Replication):** Likely **~1 Zettabyte** (considering 6x multiplier for redundancy and multiple resolutions).

---

## 3. High-Level Architecture



### The Write Path (Video Ingestion)
1.  **Direct Upload:** Client requests a **Signed URL** from the `Upload Service`.
2.  **Object Storage:** Video is uploaded directly to S3/GCS to save application server bandwidth.
3.  **Transcoding Pipeline:** * Triggered via **Message Queue** (Kafka).
    * Videos are split into chunks, transcoded into multiple resolutions (144p to 4K), and stored.
4.  **CDN Populating:** Final segments are pushed to Edge nodes.

### The Read Path (Video Consumption)
1.  **Metadata Fetch:** Client retrieves video URL and details from a **NoSQL Database** (sharded by `Video_ID`).
2.  **Streaming:** Client uses **MPEG-DASH** or **HLS** to stream segments from the nearest **CDN**.
3.  **Adaptive Bitrate:** The player dynamically adjusts quality based on network speed.



---

## 4. Critical Bottlenecks & Solutions

### A. The "Big Data" Storage Problem
* **Problem:** 183 EB is too expensive for standard SSDs.
* **Solution:** **Tiered Storage Strategy**.
    * **Hot Tier:** Trending/New videos on high-speed SSDs.
    * **Cold Tier:** 10-year-old videos with low views moved to **Erasure Coding** on HDDs or Archive storage.

### B. Global Latency
* **Problem:** Users in different continents experience buffering.
* **Solution:** Extensive use of **CDNs** and **ISP Peering** (Google Global Cache) to place data physically closer to the user.

### C. Database Performance
* **Problem:** Millions of concurrent "Like" and "View" updates.
* **Solution:** **Write-back Caching**. Buffer likes/views in **Redis** and flush to the main DB in batches to reduce I/O pressure.

---

## 5. Tech Stack Recommendations
* **Video Storage:** Amazon S3 / Google Cloud Storage.
* **Metadata DB:** Cassandra (High availability/Write-heavy) or MongoDB.
* **Caching:** Redis / Memcached.
* **Stream Processing:** Apache Flink / Spark (for real-time analytics).
* **Messaging:** Apache Kafka.

---

![alt](/images/systemImg/_-%20visual%20selection%20(8).png)
![alt](https://systemdesignschool.io/solutions/youtube/high-level-design.png)