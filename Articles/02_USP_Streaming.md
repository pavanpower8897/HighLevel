
Apache Spark (especially with Structured Streaming) doesn’t process each event as it arrives (like Flink does).
Instead, it buffers events into small batches, say every 500ms, 1s, or 2s, and then processes them together.

🧠 Think of it like:

"Wait for 1 second → gather all the events → run the logic on the whole batch."

That’s micro-batching — it's faster than traditional batch jobs, but not real-time per event.
