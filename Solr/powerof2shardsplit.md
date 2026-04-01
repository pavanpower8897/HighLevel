SPLITSHARD -> Command to split the shard on production 
CLUSTERSTATUS

Hexadecimal -> One hexi digit represent a 4 bit info
Total possible Hexadecimal symbols are -> 0, 1,…9 , A, B, C, D,E,F (Total 16 values)
So to present the larget symbol here we need max 4 bits (2POWER 4 -> 16)

Solr is built on Apache Lucene, and Lucene internally uses documentIDs, Each document gets an ID with 2*32 -1 range, So one segment holds 4 billions docs

Power-of-2 does NOT guarantee

“customer always in ONE shard”

👉 It guarantees:

“customer range will NOT be broken incorrectly during splitting”



Setup: Use ONLY 4 bits (instead of 32 bits)

Total address space:

0000 → 1111   (0 → 15)
Now assume:

👉 Customer range rule (same idea as Solr):

Customer X → XXXX
Customer occupies → XXXX0 → XXXX1   (last bit varies)
So each customer always owns 2 continuous value



Case 1: Power of 2 shards (4 shards)

We split into 4 shards (2²)

Each shard size:

16 / 4 = 4 values
Shards:

Shard 1: 0000 → 0011
Shard 2: 0100 → 0111
Shard 3: 1000 → 1011
Shard 4: 1100 → 1111
Now take a customer

Customer hash = 10XX

Customer range:

1000 → 1011
👉 Entirely inside:

Shard 3 ✅

ase 2: Non-power of 2 shards (3 shards)

Now split into 3 shards

Each shard ≈ 5–6 values:

Shard 1: 0000 → 0100
Shard 2: 0101 → 1001
Shard 3: 1010 → 1111

Step 3: What Went Wrong with 10 Shards

Initially:

Solr handled 10 shards okay (smart allocation)
But later they did:

👉 SPLITSHARD (auto split in half)

Example:

Original shard:
80000000 → 9998FFFF

After split:
Shard A: 80000000 → 8CCC7FFF
Shard B: 8CCC8000 → 9998FFFF
⚠️ Problem:

Customer range:

8CCC0000 → 8CCCFFFF
👉 Gets split across both shard

Step 4: Why This Broke Everything

Their system relied on:

1. 🚀 Routing optimization

They used:

_route_ = <customer_id>
👉 This assumes:

Customer is in one shard only
Rule 1: Use Power-of-2 shards

Because:

Enables clean binary splits
Prevents entity fragmentation
✅ Rule 2: Know your sharding key deeply

They assumed:

customer_id guarantees locality

👉 But hashing + splitting broke that assumption





e 3: SPLITSHARD is dangerous

Default behavior:

Splits blindly in half
Doesn’t respect logical boundaries
👉 Fix:

Manually define shard ranges
✅ Rule 4: Avoid cross-shard dependencies

Things that break badly:

Routing assumptions
Join queries
Aggregations

Analogy (to remember forever)

Think of shards like lockers:

Power of 2 → clean lockers per user ✅
Non-power + splitting → one user’s clothes in 2 lockers ❌
Now imagine:

You search only 1 locker → missing clothes
Or search both → slow

8 shards 
2 values 
