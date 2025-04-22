
Apache Spark (especially with Structured Streaming) doesn’t process each event as it arrives (like Flink does).
Instead, it buffers events into small batches, say every 500ms, 1s, or 2s, and then processes them together.

🧠 Think of it like:

"Wait for 1 second → gather all the events → run the logic on the whole batch."

That’s micro-batching — it's faster than traditional batch jobs, but not real-time per event.


Why Micro-Batching ≠ Sub-Second Latency?
Let’s say your goal is to respond within 200ms of an event happening (like a user clicking a listing).

If you use Spark with a 1-second micro-batch interval:

An event that arrives just after a batch starts will wait nearly a full second before it’s even seen.

Then Spark needs a few 100ms to run its job and emit results.
