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



