Replication:
In databases, locking means preventing changes to data while a certain operation (like a copy or backup) is happening.

Generally when the data is being replicated to replicas using replication log or streams we dont take locking as it affects the availability of the whole system.

Multi leader configuration:
There are some disadvantages with single leader model 
Single point of failure for writes even in multi data center approach
Performance of write compared to multi as now the data has to go to one datacenter where leader is there and from there it has to be replicated to others
Tolerance to network disruptions 
Like Only the Delhi teacher can make updates (write), and the Bangalore and Mumbai teachers only get copies of the changes (read-only followers).
✅ Simple, consistent
 ❌ If Delhi teacher is offline or unreachable, no one can accept new students or update records.

- Although multi leader configurations have certains benefits, those rarely outweighs the complexity it generally introduces. So in very rare situates if we need it then only it make sense to use it

"Retrofitted" means something new has been added to an existing system that wasn't originally designed for it.

As multi-leader replication is a somewhat retrofitted feature in many databases 

The challenges are handling write conflicts, Auto incrementing keys, triggers and integrity constraints


Here’s a simple explanation of the two use cases mentioned: offline clients and collaborative editing, both using multi-leader replication, followed by a bullet summary.

🔌 Clients with Offline Operation (like calendar apps)
Think of your phone or laptop calendar.


Even if you are offline, you can:


See events (read)


Add/edit events (write)


When you're back online, your device syncs changes to the server and your other devices.


So each device behaves like a leader that accepts writes.


The syncing between devices is multi-leader replication, done asynchronously (after delay).


It’s like each device is its own mini-datacenter, and syncing is unreliable and delayed.


Calendar sync issues in real life show how hard it is to get this right.


Some tools like CouchDB are built specifically for this offline + sync use case.



🧑‍💻 Collaborative Editing (like Google Docs)
Multiple people can edit the same document at the same time.


Everyone's changes first go to their local browser (local replica).


Then changes are replicated asynchronously to others.


If you use locking, only one person edits at a time → this is like single-leader replication.


But if you allow everyone to type at once, it’s like multi-leader replication.


This leads to conflicts — e.g., two people editing the same word at once.


So collaborative editing apps need smart conflict resolution algorithms to merge changes correctly.


Locking = single-leader style, avoids conflicts.
No locking = faster, but requires conflict resolution, like in multi-leader setups.



Sync vs Async conflict resolution:
Mostly in multi leader architecture writes will be replicated in an async way only so we will get to know about the conflicts at the time of replicating the data to replicas from leader , If we make the writes wait till the data is replicated to all the replicas then we are not leveraging the advantages of multi leader architecture

Conflict avoidance:
The simplest way to handle the conflicts is to avoid it by redirecting the users writes to same write data center but it can also have issues like user changes his location or multiple users writing from multiple locations which is not optimised and still the issue persists if the data center fails which we need to handle anyways. 

Converging toward a consistent state
In conflict state multiple leaders have multiple states of data because of different sequential execution in each data center, So eventually its important to achieve a consistent state where same data reflects everywhere 


Version clocks or Vector Clocks:
DS used to detect the conflicts in the distributed write world. Its a simple map that keeps the replica ID to counter Map

These vector clocks or maps passes across the replicas along with the data, So incoming replica takes the version map and update its own counter,Stores the updated vector clock alongside the new version of the value.
{ "A": 1, "B": 0, "C": 0 } (Replica A increments its own counter)
{ "A": 1, "B": 1, "C": 0 } (Replica B makes its own write:)

Conflict detection 
Example:
Clock 1: { A: 2, B: 0, C: 0 }
Clock 2: { A: 1, B: 0, C: 0 }
→ Clock 1 dominates Clock 2 (because 2 ≥ 1 for A, others same) → no conflict.
But:
Clock 1: { A: 1, B: 1, C: 0 }
Clock 2: { A: 0, B: 2, C: 0 }
→ Neither dominates → conflict (concurrent writes from A & B).

How does it resolve the conflicts ?

Using latest timestamp technique which maps along with the version clock 

LWW -> Here there is risk of loosing the data latest updates but still lot of applications uses its because of simplicity and its converging consistent state
We can’t use the greater number of updates as it can be equal also if 2 writes happens on Replica A and 2 on on B then they both have sample number
So, DynamoDB Is Durable BUT:
It doesn't "lose" writes accidentally — every write is safely stored.
But it can discard writes later on purpose during conflict resolution (due to design).
That’s why it’s sometimes called “durably stored but logically lost”

Dynamodb has two modes of writes: eventually consistent and strong consistent ,In eventually consistent mode only any replica can be picked instead of master node for write and eventually all the replicas will be reached to same state. So here version clocks used in this mode 
In strong consistency mode anyways the leader will be picked for writes where there is no need of version clocks. 


LWW -> Simple, fast, Works well for use-cases like caching or analytics where occasional data loss is okay.

Merge Conflicting Values Together (Automatic Conflict-Free Resolution)
System merges values automatically without conflicts:
Shopping cart: Union of all items.
Counters: Sum.
Sets: Union.
Guaranteed no data loss.
Two nodes add different items to a shopping cart:
Node A: Adds "apple".
Node B: Adds "banana".
Merged result: { "apple", "banana" }.
Why used:
Excellent for some use-cases like:
Collaborative apps (Google Docs-like tools).
Shopping carts (additive only).
No conflicts, no manual resolution needed.
⚠️ Limitation:
Only works with data that can be meaningfully merged (e.g., numbers, sets).
Doesn’t fit all data models.

Record Conflicts Explicitly — App Resolves Later

-> System keeps both the conflicts as siblings and communicate to clients for maunal resolution.

Example (Cassandra):
Writes conflict, system keeps both versions.
App gets both and resolves:
[
  { "name": "Alice", "email": "alice@example.com" },
  { "name": "Alice Smith", "email": "alice@example.com" }
]



Ensures no data loss.
Flexible — applications can decide the best merge strategy.


✅ How DynamoDB (or Dynamo-style systems) Keep Multiple Versions:
They store multiple versions of the same item within the same key entry in the underlying storage engine.

{
  "Key": "123",
  "Versions": [
    { "Value": "Alice", "VectorClock": { "A": 2, "B": 0 } },
    { "Value": "Bob", "VectorClock": { "A": 0, "B": 2 } }
  ]
}

Both versions are attached to the same key internally.

What Happens Later:
When a read happens, DynamoDB sees multiple versions.
Now it depends on system behavior:
LWW (Last Write Wins): It may pick the highest timestamped version and discard others.
Custom Conflict Resolution: Some systems expose both versions to the app to merge manually.
Background Repair: Sometimes background processes try to automatically resolve them.


Yes, memory/storage usage grows temporarily for conflicted keys.
But this trade-off is intentional and well-accepted because:
It preserves durability and availability.
Conflicts are rare in most healthy systems.
Versions are resolved & cleaned up over time.

Partition:

For very large datasets, or very high query throughput, that is not sufficient: we need to break the data up into partitions, also known as sharding.


What we call a partition here is called a shard in MongoDB, Elas‐
ticsearch, and SolrCloud; it’s known as a region in HBase, a tablet
in Bigtable, a vnode in Cassandra and Riak, and a vBucket in
Couchbase. However, partitioning is the most established term, so
we’ll stick with that

Reason: large dataset can be distributed
across many disks, and the query load can be distributed across many processors.

Difference between transactions vs analytics:

Database broadly serves two different types of workloads (Transactional and analytics) 

Transaction workloads (OLTP) - Online transaction processing. 
