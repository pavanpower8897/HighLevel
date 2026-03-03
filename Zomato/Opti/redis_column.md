https://seattledataguy.substack.com/p/back-to-the-basics-what-is-columnar
Hey team!
We’ve just rolled out a major Redis Memory Optimisation on Serviceability, and the results look impressive :rocket:

:brain: What changed
We revamped how we store serialized protobuf payloads in Redis:

Before: Row-oriented payload storage + snappy compression
After: Columnar storage layout for proto fields + Zstd compression


:chart_with_downwards_trend: Optimisation Results:

User Level Response caching memory footprint reduced by ~40% (26KB -> 16KB)
Total Redis DB memory usage reduced by ~30% during peaks (49% -> 36%)
No observable regression in latency or CPU Utilization.


What is columnar storage? why this worked?

Columnar storage stores values of the same field together in contiguous columns instead of row-wise blobs.
It provides high compression ratio for repetitive payloads.
Zstd provides better memory density than Snappy while maintaining competitive decompression speeds, leading to an additional ~10% reduction.


Columnar storage is traditionally used in OLAP systems, but this change shows the same principles can be effective at the application/cache layer when payloads are highly repetitive


Adding to the above points, our access pattern wasn’t columnar, so we added a wrapper that converts columnar data into a row format for access. Even with this extra step, overall latency still improved because columnar compression is significantly faster compared to row-based storage.

That said, columnar storage isn’t always the best approach. The gains depend heavily on the payload and access patterns. Before moving ahead, we should benchmark this using realistic payload to understand the impact on latency, CPU, and overall memory usage.

<img width="698" height="418" alt="Screenshot 2026-03-03 at 1 27 40 PM" src="https://github.com/user-attachments/assets/d1723a45-cf60-400a-bf2b-bcc431f26109" />
