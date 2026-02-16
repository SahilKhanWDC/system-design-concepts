# System Design Revision: Netflix Video Onboarding & Delivery

Netflix manages billions of requests by utilizing a highly parallelized ingestion pipeline and a localized delivery network. Below are the key components for a system design interview.

---

## 1. Video Onboarding & Processing
To handle massive file sizes and diverse device requirements, Netflix uses a **split-and-conquer** approach.

* **Video Chunking:** Raw source files are broken into smaller segments.
* **Shot-to-Scene Collation:** * Chunks are analyzed as **4-second shots**.
    * Shots are collated into **scenes** to optimize encoding recipes (e.g., complex action scenes get higher bitrates than static shots).
* **Storage:** **Amazon S3** serves as the primary object storage for all processed video chunks.

---

## 2. Global Content Delivery (Open Connect)
Netflix avoids network congestion by moving content as close to the user as possible.

* **Open Connect Appliances (OCAs):** Custom-built hardware servers provided to Internet Service Providers (ISPs).
* **Edge Caching:** These servers act as a local cache for popular movies and shows.
* **Request Handling:**
    * **Cache Hit:** Most requests are served directly from the ISP’s local Open Connect server, bypassing the broader internet.
    * **Cache Miss:** Rare or "long-tail" content is fetched over the main network.

---

## 3. Key Engineering Benefits
* **Reduced Bandwidth:** ISPs save money by not having to fetch the same movie thousands of times from Netflix's main servers.
* **Lower Latency:** Users experience faster "start-to-play" times and less buffering.
* **High Scalability:** Parallel processing of chunks allows Netflix to onboard new content rapidly regardless of the file's original size.

> **Summary:** The "synergy" lies in using **AWS** for heavy-duty computation and storage, while using **Open Connect** for high-performance, localized distribution.

---

![](/images/systemImg/_-%20visual%20selection%20(7).png)