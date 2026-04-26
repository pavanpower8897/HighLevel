- Monotonic reads [23] is a guarantee that this kind of anomaly does not happen. It’s a
lesser guarantee than strong consistency, but a stronger guarantee than eventual con‐
sistency.

One way of achieving monotonic reads is to make sure that each user always makes
their reads from the same replica (different users can read from different replicas).
For example, the replica can be chosen based on a hash of the user ID, rather than
randomly. However, if that replica fails, the user’s queries will need to be rerouted to
another replica.

- consistent prefix
reads

Mrs. Cake
About ten seconds usually, Mr. Poons.
Mr. Poons
How far into the future can you see, Mrs. Cake?
To the observer it looks as though Mrs. Cake is answering the question before Mr.
Poons has even asked it. Such psychic powers are impressive, 

This guarantee says that if a sequence of writes happens in a certain order,
then anyone reading those writes will see them appear in the same order.
One solution is to make sure that any writes that are causally related to each other are
written to the same partition

When a new replica is added, naive hashing causes large-scale remapping, breaking monotonic reads and cache locality. To fix this, systems use consistent hashing to minimize remapping, combined with techniques like version-aware routing and gradual traffic shifting to maintain consistency and stability.
