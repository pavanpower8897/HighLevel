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
