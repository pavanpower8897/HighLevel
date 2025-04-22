
Apache Spark (especially with Structured Streaming) doesn’t process each event as it arrives (like Flink does).
Instead, it buffers events into small batches, say every 500ms, 1s, or 2s, and then processes them together.

🧠 Think of it like:

"Wait for 1 second → gather all the events → run the logic on the whole batch."

That’s micro-batching — it's faster than traditional batch jobs, but not real-time per event.


Why Micro-Batching ≠ Sub-Second Latency?
Let’s say your goal is to respond within 200ms of an event happening (like a user clicking a listing).
- If you use Spark with a 1-second micro-batch interval:
- An event that arrives just after a batch starts will wait nearly a full second before it’s even seen.
- Then Spark needs a few 100ms to run its job and emit results.

In Contrast: Event-at-a-Time (True Streaming)
Frameworks like Apache Flink, Kafka Streams, or Apache Beam (on Dataflow) process each event as soon as it arrives — this is true streaming, enabling:
Sub-second end-to-end latencies.


```
// This would be called on every incoming event
func HandleMessage(event Event) {
	// Step 1: Read state from RocksDB or Flink-managed KV store
	state := ReadState(event.UserID) // load from persistent KV store

	// Step 2: Process and update state
	now := CurrentTimestampMillis()
	updatedState := ProcessEvent(event, &state, now)

	// Step 3: Write state back
	WriteState(event.UserID, updatedState)

	// Optional: Emit updated state to downstream
	EmitToTopic("user-activity", updatedState)
}


func ProcessEvent(event Event, state *UserState, now int64) UserState {
	const windowDuration = 15 * 60 * 1000 // 15 minutes in millis

	// Clean old events
	filtered := []Event{}
	for _, e := range state.Events {
		if now-e.Timestamp <= windowDuration {
			filtered = append(filtered, e)
		}
	}

	// Add new event
	filtered = append(filtered, event)

	return UserState{Events: filtered}
}

```
