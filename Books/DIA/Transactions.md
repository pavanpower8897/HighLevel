Some authors have claimed that general two-phase commit is too expensive to support,
because of the performance or availability problems that it brings. We believe it is better to
have application programmers deal with performance problems due to overuse of transac‐
tions as bottlenecks arise, rather than always coding around the lack of transactions.

James Corbett et al., Spanner: Google’s Globally-Distributed Database (2012)

The most dangerous transaction is often not the one doing the most computation. It is the one holding a popular resource while waiting on remote work.

The strategic comparison is not:
expensive distributed transaction
versus
free alternative
It is:
database-managed transaction
versus
application-managed partial failure
Avoiding 2PC does not remove coordination. It pushes coordination into application code.
Without transactions, developers may need to implement:
idempotency keys;
retries;
deduplication;
compensating operations;
reconciliation jobs;
intermediate state machines;
event ordering;
recovery after crashes;
monitoring for inconsistent states.

That alternative may improve availability or scalability, but it is not automatically simpler or cheaper.

Fault tolerance is the engineering work that anticipates, contains, and recovers from those faults. Reliability is the user-visible result: the system continues to meet its stated correctness and service guarantees despite them.

A **fault** is one component going wrong:
A server crashes during a write.
A network message is lost.
An application process restarts.
Two clients update the same record concurrently.
A disk becomes unavailable.
A request is repeated after a timeout.

In order to be reliable, a system has to deal with these faults and ensure that they
don’t cause catastrophic failure of the entire system. However, implementing fault-
tolerance mechanisms is a lot of work. It requires a lot of careful thinking about all
the things that can go wrong, and a lot of testing to ensure that the solution actually
works.

**server crashes during a write.**
- Crash before COMMIT: Database aborts the transaction; client starts a new transaction and safely retries.
- Crash during COMMIT, before durable commit: Database aborts on recovery; client treats the result as unknown and retries using the same idempotency key.
- Crash after durable commit, before data pages finish writing: Database replays the log and preserves the commit; client checks status or retries idempotently.
- Crash after durable commit, before response reaches client: Transaction succeeded but client sees a timeout; client must query status or retry with the same idempotency key.
- Crash after success response reaches client: Transaction is committed and durable; client proceeds without retrying.
Revision rule: A timeout during COMMIT means unknown outcome—not failure; check status or retry idempotently.

With transactions, error
handling becomes much simpler for an application, because it doesn’t need to worry
about partial failure—i.e., the case where some operations succeed and some fail 

Pure hyperbole means an exaggerated, overly absolute claim that ignores real-world trade-offs.
The author is criticizing two opposite viewpoints:
“Distributed transactions are always too expensive.”
This exaggerates their performance and availability costs; transactions can be perfectly reasonable when used within suitable boundaries.

“Every serious application with valuable data must use transactions.”
This exaggerates their necessity; some systems can maintain correctness using idempotency, conditional writes, asynchronous workflows, compensation, and reconciliation.

The balanced viewpoint is:
Transactions are neither universally harmful nor universally necessary; choose them according to the required correctness guarantees, failure model, performance needs, and operational complexity.


Giving up partition instead of Consistency and available in CAP theorem:
One ATM. One single database, in one location, no replica anywhere else. There's no second copy to fall out of sync with, so there's nothing to partition.
Consistency is the application's responsibility.

C is fundamentally different from A, I, and D.

Think of it this way:

Property	Who primarily provides it?
Atomicity	Database
Isolation	Database
Durability	Database
Consistency	Application + database constraints
Database guarantees safe execution.
Application guarantees correct meaning.

Or even simpler:

ACID doesn't mean "the database makes my data correct."

It means:

If I define my transaction correctly and my starting state is valid, the database's transactional guarantees help me move safely to another valid state.


What is Consistency ?
In d systems Consistency is a concept which explains how quickly we can fetch the results which we have written, 
S con -> Once the data has updated, We always see the latest update results
E con -> Eventually we will be able to see but not immediate 

CAP Theorem 
Can we acheive C, A by giving up P ?
Giving up P means there shouldnt be any network tolerance that means we cant power the system using multiple servers which effects the scalability like 
As we are forced to power it using single server

Above example can be applicable only to databases or application servers aswell ?


What is isolation ?
Isolation in the sense of ACID means that concurrently executing transactions are
isolated from each other: they cannot step on each other’s toes. The classic database
textbooks formalize isolation as serializability, which means that each transaction can
pretend that it is the only transaction running on the entire database. The database
ensures that when the transactions have committed, the result is the same as if they
had run serially (one after another), even though in reality they may have run con‐
currently


Ex: Two clients who are trying to increment same record, Then we can see incorrect count values 

Dynamodb aswell provides isolation gurantees

Like if when updating the items 
Even when concurrent requests lands parllely , It will execute them serially inorder to acheive the valid state
UpdateExpression:
SET counter = counter + :counterValue,
    balance = balance - :balanceValue

But is it only true for increment/decremetnals operatiosn ?
No we can also acheive this on normal fields updates liek status etc but we need to use condition writes

"Once APPROVED, nobody should be able to change it back to REJECTED."

then you need an additional concurrency-control mechanism, such as a ConditionExpression:

ConditionExpression:
status = :pending
Otherwise based on the ordering simiply status will be overrided 

ncrement/decrement is just one common concurrency problem.

Isolation matters whenever concurrent operations access overlapping data:

Operation	Concurrency concern
counter += 1	Lost updates
status = APPROVED	Conflicting writes
stock -= 1	Overselling
balance -= 100	Concurrent balance changes
SET field = X	Last-write-wins conflicts (Database overides based on the ordering)
Multiple items	Cross-item consistency


