# HighLevel

- Contention collapse
- Latency spikes
- Throughput collapse
- Wasted CPU work
- Retry storm (amplification)
- optimistic locking
- Write amplification
- Operational complexity
- Processing lag
- Multi-writer contention
- Async aggregation
- idempotent
- Exactly once gurantee
- High contention
- Thundering herd
- Clock synchronisations

System design good resources:

https://medium.com/coders-mojo/quick-roundup-solved-system-design-case-studies-6ad776d437cf

https://www.enjoyalgorithms.com/system-design/

https://www.uber.com/en-IN/blog/real-time-push-platform/


🧠 How to Think About It as a System Designer

Don’t assume:
- Events will come once and on time.
- Your processor will never crash.
- Your data pipeline will always be consistent.

Instead, ask:

- "If I get this event twice, what will happen?"
- "If this event is late by 10 minutes, will it still be useful?"
- "If I reprocess yesterday’s data, will I corrupt today’s?"

