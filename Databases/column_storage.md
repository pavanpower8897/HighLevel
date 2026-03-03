Simple Mental Model

Think of it like this:

Row storage = optimized for writing transactions

Column storage = optimized for reading analytics

Analytics = scan big data → aggregate → filter
Columnar storage was built exactly for that.



5️⃣ The Big Takeaway

Columnar storage exists because:

Analytics workloads read a few columns across many rows.

So storing data by column:

Reduces I/O

Improves compression

Speeds up queries

Lowers cost

