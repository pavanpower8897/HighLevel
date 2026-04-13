👉 What is a Set?

A Set is:

Unordered collection
No duplicates allowed
Optimized for membership checks

✅ SADD (Add to Set)
SADD users:online user1 user2 user3

👉 Result:

{user1, user2, user3}
If you try adding duplicate:
SADD users:online user1

👉 Nothing changes (no duplicates)

SREM (Remove from Set)
SREM users:online user2

👉 Result:

{user1, user3}

⏱️ Time Complexity of SADD and SREM in Redis
🔹 1. SADD (Add element to Set)

Time Complexity:

✅ O(1) → for each element added
So if adding multiple elements:
👉 O(N) (where N = number of elements)


🔹 2. SREM (Remove element from Set)

Time Complexity:

✅ O(1) → per element
Removing multiple:
👉 O(N)
Example:

Why O(1)? (Important Insight)

Redis Sets are implemented using:

Hash table (hashtable) internally (most common case)

👉 So operations like:

Add (SADD)
Remove (SREM)
Check (SISMEMBER)


2. Why Threshold = 512?

⚠️ Compare with intset:
Structure	Memory for 3 elements
intset	~14 bytes
hashtable	
This is NOT random — it’s based on memory vs CPU tradeoff curves.
Hidden factor for 512: CPU Cache

This is VERY important.

intset:
Stored contiguously
Fits in CPU cache (L1/L2)

👉 blazing fast
