# How Instagram Handled 2 Billion Users with a Single PostgreSQL

In its early days, Instagram launched its service with just a single PostgreSQL database. Today, it has grown into a massive system handling over 2 billion users.

There is a common perception that relational databases (RDBMS) like PostgreSQL are not suitable for massive scale-outs. So how did Instagram overcome this limitation? The process of their database expansion is summarized below in a sequential Q&A format.

---

### Q1. Is PostgreSQL inherently difficult to scale out?

PostgreSQL is fundamentally optimized and designed for a single-node environment. Therefore, when traffic increases, the primary approach is vertical scaling (Scale-up) by upgrading the hardware performance (CPU, RAM). However, as a service experiences explosive growth, it inevitably hits a physical limit where even the highest-spec instances provided by cloud providers cannot handle the traffic. This physical ceiling is why large-scale horizontal scaling (Scale-out) is often perceived as difficult.

### Q2. When the vertical scaling limit (Max Instance) was reached, why wasn't the database immediately distributed across multiple servers?

Splitting a database across multiple servers (Sharding) is a highly complex and risky operation that requires a complete overhaul of the application architecture. Before blindly partitioning the servers, Instagram's engineers analyzed the "root cause" of the database's struggles with microscopic precision.

### Q3. Was the true cause of the database performance degradation the data size or disk bottlenecks?

The culprit wasn't the size of the data at all; it was "memory evaporation due to connections."
Instagram's web servers (Django) were opening countless connections to communicate with the DB. By design, Postgres consumes about 1.3MB of RAM per connection.

50 Django servers × 30 connections per server × 1.3MB = About 2GB of RAM wasted just keeping the communication pipes open (Connection Overhead).

### Q4. How was this severe memory waste (Connection Overhead) resolved?

A connection pooler named **PgBouncer** was introduced between the web servers and the database.
Instead of the Django servers establishing numerous direct connections to the DB, PgBouncer intercepted the requests and multiplexed them, maintaining only a small number of essential communication pipes to the actual DB. This freed up the wasted memory, allowing the RAM to be fully utilized for the database's core tasks, such as query processing and caching.

### Q5. Were there no further issues after introducing PgBouncer?

While the immediate memory leak issue was resolved, as the user base grew into the tens of millions, they faced a fundamental physical limit. The disk storage space and CPU computing power of a single server were simply no longer enough to store all the photos and data pouring in from around the world.

### Q6. Once physical limits were reached, wouldn't migrating to a NoSQL database be better for scaling? Why was PostgreSQL maintained?

This decision was driven by the engineering philosophy that **"Boring Tech Wins."**
Abandoning PostgreSQL—a database with decades of proven stability—to introduce a new NoSQL solution would consume massive amounts of time and "Complexity Budget." Instead of switching infrastructure technologies based on trends, Instagram decided to stick with their robust, existing technology. They focused their saved budget and resources on solving "real new business problems" that delivered actual value to their users.

### Q7. If PostgreSQL was kept, how was that massive amount of data distributed (Scale-out)?

Instagram implemented **Logical Sharding**.
Rather than simply partitioning data by physical servers, they first divided the data into 4,096 "logical shards" based on identifiers like PhotoID. They then designed a decoupled architecture that mapped these logical shards to a small number of physical servers.
Through this approach, even if a specific physical server reached full capacity, they could seamlessly move logical shards to other servers just by updating the addresses in the mapping table, ensuring high scalability without modifying a single line of application code.

---

## Architecture Evolution Timeline Summary

* **Phase 1: Single Postgres**
* Started with just one PostgreSQL node. Addressed traffic growth through vertical scaling (Scale-up) while maintaining the simplest possible architecture.


* **Phase 2: Add PgBouncer**
* Experienced Connection Overhead and memory depletion due to the increase in app servers. Resolved the bottleneck by introducing PgBouncer for connection pooling.


* **Phase 3: Shard Logically**
* Reached the disk/CPU limits of a single physical server. Built a decoupled sharding architecture that partitioned data into logical shards and flexibly allocated them to physical servers.


* **Phase 4: Snowflake-style IDs**
* To guarantee unique Primary Keys in a distributed sharded environment, implemented a technique that generates collision-free unique IDs using a combination of (Timestamp + Shard ID + Sequence) without relying on a central server.


* **Phase 5: Planet Scale**
* Overcame the limits of proven technology (PostgreSQL) through relentless analysis and structural optimization, completing a system capable of handling over 2 billion global users.
