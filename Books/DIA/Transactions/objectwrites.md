Single-Object Writes — Simple Notes

Goal: A database should make a write appear all-or-nothing, even if the write is interrupted or the database crashes.

1. Atomicity

A single object should never be exposed in a partially updated state.

Either the old value or the new value — never half-old/half-new.

2. Two common ways to achieve this

A. In-place update + WAL

Update the existing pages directly.
Use a lock so concurrent operations don't see the intermediate state.
Use WAL (Write-Ahead Log) so the database can recover if it crashes.
After a crash, the DB uses the log to undo or redo the operation.

B. Copy-on-write / versioning

Write the new object somewhere else.
Once complete, switch the pointer/reference to the new object.
Old version remains untouched until the switch.

You don't have to use versioning. In-place updates + WAL are also common.

Lock = protects concurrent readers/writers
WAL = helps recover from crashes
Atomicity = makes the whole write appear all-or-nothing


A vs B — when each makes sense
	A: In-place + WAL	B: Copy-on-write / Versioning
How it works	Modify existing pages	Write new pages, then switch reference
Main advantage	Efficient updates; less extra space	Easier isolation/recovery model
Main cost	More complex crash recovery	Extra writes/storage
Common idea	PostgreSQL-style WAL/in-place page updates	LSM/tree variants, snapshot systems, some storage engines
Good when	Frequent updates, efficient space usage	Snapshots/versioning are valuable
Simple mental model

Imagine a notebook.

Approach A — edit the existing notebook

Old page
   ↓
Erase/change it
   ↓
Write new content

If something goes wrong halfway, your backup log tells you how to recover.

Approach B — create a new notebook

Old notebook → still untouched

New notebook → write everything

When complete:
CURRENT → New notebook

If something goes wrong, simply keep using the old notebook.

So why would a database choose B?

Copy-on-write can make things like snapshots, rollback, and crash recovery easier because the old version remains intact.

But it can require more storage and additional writes.

The key decision

Think of it as:

Approach A: “Modify efficiently and use a log to recover.”
Approach B: “Keep the old version safe and switch to the new version when ready.”

For most applications, you don't manually implement this choice. You select a database/storage engine whose internal design fits your workload.


Locking approach:

"We're changing the menu. Nobody can read it until we're finished."

Bad when 1,000 customers want to read the menu. 😄

Versioning/MVCC approach:

"Keep the old menu available while we prepare the new menu. Once the new menu is ready, new customers get it."

Much better for read-heavy workloads


You're actually discovering an important database design trade-off:

Read-heavy workload → minimize reader/writer contention.

That's why techniques such as MVCC, snapshots, read replicas, caching, and lock-free/read-optimized structures are valuable.

So your intuition is right — contention is often the real performance problem, not the mere existence of transactions or locks.
