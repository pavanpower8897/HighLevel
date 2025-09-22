Wrapper for dynamodb where we implemented a connection pool to optimize the latency of p99
If ur Connection Pooling for High-Throughput Upstreams!
If your service makes frequent upstream calls, be mindful of connection pooling to avoid unnecessary overhead and latency spikes.

In DynamoDB, read throttling and write throttling are independent. BatchWriteItem only consumes write capacity units (WCU).If your table or partition runs out of write throughput, you’ll see a ThrottlingException (like the one in your error).High reads elsewhere do not directly cause BatchWriteItem to fail.Read-heavy traffic consumes read capacity units (RCU).If RCUs are exhausted, read requests (GetItem, Query, Scan, BatchGetItem) get throttled, but writes are unaffected.

a tenant doesn’t mean “all customers using the app”, it’s more like a logical grouping of users under one account or organization. Let me clarify: Tenant = a logical customer unit, A tenant represents a customer or organization that uses your application, One tenant can have one or more users.

If you hammer one hot partition with both lots of reads + lots of writes, you can still hit throttling because the partition’s total capacity is exceeded.

 MemDS : distributed cache service

- Metadata Server stores the info in both persistent disk and local cache after that 
- Metadata store = control plane → it’s designed to manage correctness, durability, and relatively low-frequency updates (partition splits/merges, table creation, rebalancing).
- Request routing lookups = data plane → happens on every single request, millions per second.
- Even if you scale metadata service with 1000 ECS tasks:
- You’ve now turned your control plane into a hot data plane dependency.
- Any hiccup in the metadata service = global outage.
- The fan-in load will grow with traffic, making it hard to guarantee low tail latencies.
- Fault tolerance → Even if some nodes crash or act slow, the system still makes progress.


1. Why not just scale Metadata Store like using local cache instead of memcache?

Metadata Store = control plane.

It does keep the partition map in memory, but it must also persist updates to durable storage (disk or a replicated DB).

If you tried to scale Metadata Store to handle millions of requests/sec like Request Routers (RRs), then:

Every metadata replica would either need a big local disk (harder to keep in sync), or

They’d all be hitting a shared backing store (network/disk bottleneck).

Consensus + persistence = much higher coordination cost per request.

That’s why Metadata Store is not designed as a data-plane service.

So, you’re right — even if the partition map is cached in memory, scaling Metadata Store horizontally to RR-level throughput would increase the risk of global outages (more replicas, more leader elections, more chances of inconsistency).


In simple terms data plan and control plans are design concepts which containe tenchiques and strategies, We can apply these strategies to acheive the expected outcomes, like better scalability, reliability, availability, fault tolerance. 

Data plan -> Designed for handling high thorughput, low latency work , as forwarding packets, serving reads/writes, applying filters/ACLs, and enforcing policies at runtime. (Data plane components execute the real-time workload: serving queries, reading/writing to caches and databases, enforcing runtime rules, and staying optimized for throughput and latency. like our search service)

control plan -> Designed for determining the state such as configurations, polocies, data eetc. 

Scalability: the data plane scales with traffic; the control plane scales with configuration and coordination complexity.

Resilience: if the control plane is impaired, the data plane can often continue operating with the last known good configuration; faults are isolated across planes.



![WhatsApp Image 2024-11-27 at 6 28 00 PM](https://github.com/user-attachments/assets/5af5af3f-3673-4c3b-87b5-63c387477a1a)
![WhatsApp Image 2024-11-27 at 6 28 00 PM (1)](https://github.com/user-attachments/assets/ca09357a-1974-4660-acc7-75d1eddd16a3)
![WhatsApp Image 2024-11-27 at 6 28 01 PM](https://github.com/user-attachments/assets/a9370603-3cc7-4b19-9f82-e0625e5792f4)
![WhatsApp Image 2024-11-27 at 6 28 01 PM (1)](https://github.com/user-attachments/assets/a4eecd64-1089-4746-aa06-2ac6854f6167)
![WhatsApp Image 2024-11-27 at 6 28 01 PM (2)](https://github.com/user-attachments/assets/6f22730e-1a42-4ffc-ad8e-abee631de4c7)
![WhatsApp Image 2024-11-27 at 6 28 02 PM](https://github.com/user-attachments/assets/8996688b-0440-4b97-be3e-29754214f438)
![WhatsApp Image 2024-11-27 at 6 28 02 PM (1)](https://github.com/user-attachments/assets/da79b469-ef98-4f3b-b93d-f61602d5170c)
![WhatsApp Image 2024-11-27 at 6 28 02 PM (2)](https://github.com/user-attachments/assets/e4831afe-ae3b-41bb-9c7c-fb09a1d943bb)
![WhatsApp Image 2024-11-27 at 6 28 03 PM](https://github.com/user-attachments/assets/d1a917af-a671-42d4-a83c-75b73c3ab1ef)
![WhatsApp Image 2024-11-27 at 6 28 03 PM (1)](https://github.com/user-attachments/assets/3d758791-6834-4218-bfbd-8d88cf4828a9)
![WhatsApp Image 2024-11-27 at 6 28 03 PM (2)](https://github.com/user-attachments/assets/e46b2360-2d4c-4905-953e-fb2c9cbdbeae)
![WhatsApp Image 2024-11-27 at 6 28 03 PM (3)](https://github.com/user-attachments/assets/45b13394-1a20-4ab0-8fcd-7f57181d5f0c)
![WhatsApp Image 2024-11-27 at 6 28 04 PM](https://github.com/user-attachments/assets/e4341bb7-5929-4f46-ab79-d8a39b71f755)
![WhatsApp Image 2024-11-27 at 6 28 04 PM (1)](https://github.com/user-attachments/assets/e417ecf8-765c-408b-9c8f-0136366fb438)
