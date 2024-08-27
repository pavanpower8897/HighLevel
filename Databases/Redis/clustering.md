# Redis Clustering

Modern application requirements:

→ Scalability: Ability to scale up or down based on the requirement

→ High availability: Ability to available highly by data replication in different multiple zones with automatic failover

→ Security

How aws elastic cache scale reads ?

**Cluster mode is disabled**  

There is only one shard going to accept requests where in that shard there will be primary node and replica nodes where primary node accepts writes and reads and replica nodes will power reads, we can have replicas upto 5 and its good to have read replicas for heavy read cases as it will reduce the load on primary node so that it can take care of writes and updates.

One thing to not here is that primary node cpu load can increase a bit because of data replication to replicas and it can take some network capacity of primary node also. 

**Cluster mode is enabled** 

Then we can have more than one shard where data will be distributed to multiple shards which contains primary nodes and replica nodes. Each shar handles subset of data.

To scale ready heavy workloads we can simply add read replicas but if we want to scale write replicas horizontally we need to add new shards. 

Cluster continues to server requests even when we are adding new shards to the cluster

Scaling vertically: We can scale horizontally or vertically when our cluster mode is enabled/disabled, Its ntng but upgrading to a new instance type with larger provisions. 

Redis → In memory data structure store (What is in memory ? What all data structures it can store) 

Redis has the ability to scale horizontally using clustering. (How ?)

**Gossip protocol:** 

It is used for node discovery and message propagation in redis cluster

Questions:

- What is in memory ?
- How horizontal scaling acheived using clustering ?
- What is fault tolerance  ?
- How data will be copied from master to replica nodes in redis ?
- What is key space and how redis rebalance that key space ?
- What is redis cluster protocal ?
- How many master nodes can be present in a cluster and how to decide the replicas ?
- And who is going to know whether a node is master or replica and how this transformation happens ?
- Where is the slot and node mapping presents ? And only 16, 384 slots present in a single cluster ?
- What is cluster topology ?
- Redis uses which hash technique for distributing data evenly ?
- What are hash slots ?
- How the network bandwidth is distributed between shards and replicas in redis cluster and what kind of best practices we need to follow for not choking the bandwidths ?
