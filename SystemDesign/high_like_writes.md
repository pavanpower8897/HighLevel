Async aggregation (this is where real scale comes from)
Now let’s go deep—this is a core system design pattern.



Problem we are solving
If every “like” does:

UPDATE posts SET like_count = like_count + 1

👉 At scale:
* Hot rows
* Lock contention
* Leader overload


p 2: Write event to log (Kafka-style)
Send to a system like:
* Apache Kafka
Now you have:


Why this is powerful
* Writes are append-only
* No contention
* Extremely scalable


Step 3: Stream processing (aggregation layer)
A consumer processes events:
* Apache Flink
* Apache Spark Streaming


Aggregation logic
For each event:

(post_id=456) → increment counter

Maintain state:

post:456:likes_count = 101


Step 4: Store aggregated result
Write to:
* Redis (fast reads)
* or DB (materialized view)


🧠 Important: This is NOT real-time
There is delay:

User click → event → queue → processing → update

Delay = few ms to seconds



Problem 1: Duplicate events
User retries → multiple LIKE events
Solution:
Make events idempotent
Store:

(user_id, post_id) unique


🧩 Problem 2: Unlike
User:
* Like
* Unlike
Now aggregation must handle:


LIKE → +1
UNLIKE → -1


Better approach
Instead of blind increments:
👉 Maintain state, not just counts

(user_id, post_id) → liked/unliked

Then recompute counts correctly


🧩 Problem 3: Ordering
Events may arrive out of order
Example:

UNLIKE arrives before LIKE


Solution
Use:
* timestamps
* versioning
* or partitioning by key (Kafka partition per pos


🧩 Problem 4: Hot keys (viral posts)
One post gets millions of likes

Solution: Sharded counters
Instead of:

post:456:likes = 1 counter

Use:

post:456:likes:shard1
post:456:likes:shard2
...

Then sum them at read time

✅ Benefits
1. Massive write scalability
* Millions of events/sec
2. No DB contention
* No row-level locking
3. Replay capability
* Rebuild counts anytime
4. Decoupling
* Write path ≠ read path

❌ Tradeoffs
1. Eventual consistency
* Counts may lag
2. Complexity
* Stream processors
* event schemas
* failure handling
3. Debugging difficulty
* Harder than CRUD systems


For idempotency:
API → Kafka (source of truth)
       ↓
   Consumers:
     → Redis (serving layer)
     → DB (offline / backup)

Or for idempotency we can relay on kafkaesque partition processing guarantee 
How stream processing fixes this
1) Partitioning guarantees single-threaded ordering per key
With Apache Kafka:

partition_key = hash(user_id, post_id)

👉 All events for (u, p) go to the same partition

2) One consumer processes that partition
Inside something like Apache Flink:

Partition 7 → handled by exactly ONE task/thread

So execution becomes:

LIKE → UNLIKE → LIKE → UNLIKE

Processed strictly in order:

state = LIKED
state = UNLIKED
state = LIKED
state = UNLIKED

👉 Final state is always correct

🧠 Key realization
We didn’t “solve ordering with logic” We solved it by eliminating concurrent writers

🔁 What about duplicates? (idempotency)
Even with Kafka, duplicates can happen (retries, replays).
Example

LIKE
LIKE   (duplicate)
UNLIKE

Processing:

state = LIKED
state = LIKED   // no change
state = UNLIKED

👉 Still correct

Why this works
Because operations are:

SET state = X

NOT:

INCREMENT / DECREMENT


Edge case: network race
Two servers send:

LIKE
UNLIKE

But due to network:

UNLIKE arrives first

👉 Kafka order becomes:

UNLIKE → LIKE

Final state = LIKED

Is this wrong?
👉 Depends on requirement:
* If you need real-time exact ordering → you need stronger coordination (expensive)
* Most systems accept: “Order = arrival order”

Kafka = memory of the system
Flink = temporary brain (can forget and relearn)
Redis = face shown to users



How Flink actually handles offsets

When using Apache Kafka with Apache Flink:

Important:

For ensuring exactly once processing gurantee:
We have to maintain attomicity between kafka commit and state checkpoint 

👉 Flink stores Kafka offsets inside its checkpoint

Checkpoint C1:
  state = ...
  offsets = partition-3 → offset 100


100
🔄 What happens during processing
Step 1: Process events
E1 → E2 → E3

State updated accordingly.

Step 2: Checkpoint happens

Flink captures:

state = after E3
offset = 3
Step 3: Kafka commit (optional / async)

Flink may commit offsets to Kafka:

commit offset = 3

👉 BUT this is just for monitoring / external tools


Why plain INCR/DECR breaks

Your earlier failure scenario still applies:

Before crash
count = 100
LIKE → INCR → 101
(crash before checkpoint)
After restart
event replayed → INCR again → 102 ❌

👉 Redis doesn’t know it already processed that event

🧠 Root problem

Redis INCR is not idempotent

It blindly applies deltas.


So here instead of redis incr we should use flink state count and sets to redis for making the aggregated counts consistent

✍️ Short answer

Apache Flink does NOT store checkpoints in Apache Kafka.

It stores them in a separate, durable storage system (called a state backend / checkpoint storage).

🧠 Where checkpoints are actually stored

Flink writes checkpoints to external persistent storage, typically:

Common options
Filesystem (local or network)
HDFS (Hadoop Distributed File System)
Object storage:
AWS S3
GCS
Azure Blob

What gets stored there?

Each checkpoint contains:

Checkpoint C123:
  ├── Operator state (your (user_id, post_id) → state)
  ├── Aggregates (post_id → count)
  ├── Kafka offsets (per partition)
  └── Metadata (topology, versions)


We are eliminated the need to take care of concurrent aggregarted count writes by moving to  post_id level partition processing


Redis vs async aggregation thrugh apach kafka Lets talk about the limiting factors interms of scalability & reliability with above approachs, At what scales we should what ?

🧠 Two archetypes
1) Redis-centric (direct writes)
API → Redis (SET / INCR) → Read
2) Async aggregation (event-driven)
API → Kafka → Flink → Redis (SET projection) → Read


Limiting factors
1) Hot keys (biggest killer)

A viral post:

post:123:likes_count

All traffic hits ONE key → ONE shard → ONE CPU core

👉 Even with Redis cluster:

Key cannot be split automatically
Throughput capped by a single node
2) Multi-writer contention

Many API servers:

Server A → INCR
Server B → INCR
Server C → INCR

👉 Redis handles atomicity, but:

CPU bound
network saturation
latency spikes

📈 Practical ceiling (rough intuition)

Redis-only starts struggling when:

> 50k–100k writes/sec on a single hot key
or millions of concurrent writers
or strict correctness is required


Why it scales better
1) Partitioned parallelism
partition_key = post_id

👉 Each post handled independently

100 partitions → 100 parallel processors
linear scalability
2) No hot-key contention

Even viral post:

post:123 → partition 7 → single thread

👉 No lock, no contention, just a queue


Limiting factors
1) Partition bottleneck for a single key

One post → one partition → one thread

👉 Throughput per post is bounded

(you can push this far, but it’s not infinite)

2) Processing lag

If ingestion > processing:

Kafka lag increases

👉 Users see delayed counts

3) Operational complexity
Kafka cluster
Flink jobs
checkpoint tuning
monitoring lag

3) Write amplification handled

Millions of writes:

append-only log (Kafka
