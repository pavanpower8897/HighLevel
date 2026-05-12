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
- conflict resolutio
- retry handling
- conditional writes
- Monotonic Reads
  -  Sticky routing (“Same user → same replica”)
  -  Read-your-writes via leader
  -  Version-aware routing 
- consistent prefix reads 


** We should design for evolvability, not prematurely optimize for hypothetical requirements

Ex: Yes, future workflows may require online revision history. But today our confirmed operational requirement is current-state workflow execution, while immutable historical lineage already exists in append-only pipelines. We can evolve the online model toward versioning once real product workflows justify the additional complexity

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

