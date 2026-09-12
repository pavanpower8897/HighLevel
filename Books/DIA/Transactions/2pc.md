Normal transactions bottlneck analogy:
Easy analogy

Imagine one bathroom 🚽 and 10 people:

10 people
    ↓
1 bathroom
    ↓
Everyone competes for it
    ↓
Waiting

That's contention.

If there are 10 bathrooms:

10 people
    ↓
10 bathrooms
    ↓
Much less contention


2PC is needed when one transaction spans multiple independent databases/services, and we want all of them to commit or all of them to abort.

1. Start with two databases

Suppose an order involves:

Application
    │
    ├── DB A: Orders
    │
    └── DB B: Payments

When a user places an order:

DB A → Create order
DB B → Deduct payment

We want this invariant:

Either both happen, or neither happens.

Otherwise we could get:

DB A: Order created ✅
DB B: Payment failed ❌

Now we have an inconsistent business state.

2. Why can't we just use normal transactions?

Each database can have its own local transaction:

Transaction A:
DB A → BEGIN
DB A → create order
DB A → COMMIT

Transaction B:
DB B → BEGIN
DB B → deduct payment
DB B → COMMIT

But there is a problem.

Suppose:

DB A → COMMIT ✅
DB B → COMMIT ❌

Now we have:

Order exists
Payment doesn't exist

There is no automatic way for DB A's local transaction to know that DB B failed.

That's the problem 2PC solves.

3. Enter the coordinator

2PC introduces a coordinator:

                 Coordinator
                 /          \
                ↓            ↓
              DB A          DB B
             Orders        Payments

The coordinator manages the transaction across both databases.

It has two phases.

Phase 1: Prepare

The coordinator asks both databases:

"Are you ready to commit?"

Coordinator
    │
    ├──→ DB A: PREPARE?
    │       └──→ YES
    │
    └──→ DB B: PREPARE?
            └──→ YES

Each database does the necessary work to ensure:

"If you later tell me to commit, I can commit successfully."

They then enter a prepared state.

Phase 2: Commit

If everyone said YES:

Coordinator
    │
    ├──→ DB A: COMMIT
    │
    └──→ DB B: COMMIT

Now both commit.

DB A → COMMITTED ✅
DB B → COMMITTED ✅
What if one says NO?

Suppose:

DB A → YES
DB B → NO

The coordinator tells everyone:

ABORT

So:

DB A → ROLLBACK
DB B → ROLLBACK

Thus:

All commit or all abort.

4. Now you can see why 2PC is expensive

Compare it with a normal transaction.

Single database
Application
    ↓
DB
    ↓
COMMIT
2PC
Application
    ↓
Coordinator
    ↓
PREPARE
 ↙       ↘
DB A     DB B
 ↓        ↓
YES      YES
 ↘       ↙
  Coordinator
      ↓
   COMMIT
   ↙    ↘
 DB A   DB B

There is additional coordination and communication.

And more importantly, participants may have to stay prepared and hold resources while waiting for the coordinator's final decision.

5. The really important bottleneck

Imagine:

T1:
DB A → prepared
DB B → prepared

Coordinator → 💥 CRASH

Now:

DB A → "Should I commit?"
DB B → "Should I commit?"

They can't independently decide.

They may have to remain in the prepared state until the coordinator's decision can be recovered.

This can mean locks/resources remain occupied, causing other transactions to wait.

So:

2PC
 ↓
Extra coordination
 ↓
Participants may wait
 ↓
Resources remain occupied
 ↓
Contention
 ↓
Higher latency / lower throughput

That's the deeper reason the book talks about performance and availability problems.

The mental model

Normal transaction:

"One database, make these changes atomic."

2PC:

"Multiple independent databases, make this entire distributed transaction atomic."

And the trade-off is:

2PC gives you distributed atomicity, but you pay for it with coordination, latency, resource holding, and failure-related blocking.
