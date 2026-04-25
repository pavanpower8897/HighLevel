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
