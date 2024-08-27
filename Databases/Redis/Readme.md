Memory fragmentation paper:
https://medium.com/doubtnut/redis-fragmentation-and-optimization-3ddc95499382 

Redis is an in memory data store. 

Redis uses single threaded architecture. 

***What is single threaded arc ?***

A single thread proceses all the incoming requests in that instance. Here redis uses single threaded event loop where it processes commands in a loop and if there is any IO operation happening it processes other command meantime.

**Why redis uses single threaded arc ?** 

Because multi threading has some performance overheads like 

- Data synchronisation needs to be handled through mutex locks or samephores for ensuring the data integrity as dead locks can occur when two process trying to access same shared resource. And there will be some complexity and performance overhead
- Context switching: In multi threading env cpu switches the context between threads for concurrent processing , so switching the threads includes saving the previous instruction pointer info etc which can decrease the performance.
- CPU Caching: Where CPU Caches the frequently accessed data into L1,L2, L3 Layers, So data will cleared more frequently in multi threaded env compared to single threaded , Decrease the performance.

To increase the scalability and availability of redis we use redis clustering concept. Added in above doc.

Below CPU Cache layers present closer to CPU Cores than the main memory, So providing faster access times than from main memory.

- L1 Cache: The Level 1 cache is the closest and fastest cache to the CPU Cores. It has the lowest capacity but offers lowest latency and highest bandwidth. It split into below two parts
    - Instruction cache→ Holds instruction fetched from memory
    - Data cache → Stores frequently accessed data.
- L2 Cache: It is larger than L1 Cache and present further away from CPU Cores. It serves as a secondary cache providing additional capacity to store frequently used data and instructions. L2 Cache typically shared amount multiple cores.
- L3 Cache: It is a higher level cache shared amount multiple CPU Cores or processor modules.Its larger and slower than L1 and [L2.It](http://L2.It) tries to reduce the need to access main memory frequently.
