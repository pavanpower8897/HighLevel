Snowflak sytle id:
Structure (64-bit integer)

| timestamp | machine_id | sequence |
Example (as a number):

1893472398472398472

🔢 Why this works perfectly

✅ Pure numeric (int64)
✅ Globally unique
✅ Works at millions of QPS
✅ Time sortable (very useful for queries)
✅ No DB/network dependency

Clock going backward

If system time shifts:
👉 Duplicate IDs possible

Fix:

Detect rollback and:
Wait OR
Fail fast


🧠 First: Can Snowflake sequence “break” at high QPS?

Short answer:
👉 No, if implemented correctly
👉 Yes, if you ignore limits

⚙️ What “sequence” actually means

In Snowflake:

sequence = counter within SAME millisecond per machine
Typical:

12 bits → 4096 IDs per ms per machine
🚨 When does it break?

Only in one case:

👉 A single machine generates >4096 IDs in the same millisecond

That means:

> 4 million IDs/sec per machine
👉 That’s extremely high, but possible in hot systems.

🔥 What happens then?

Sequence overflows:

sequence = 4096 → resets to 0
Now:

Same timestamp
Same machine ID
Same sequence (0)
👉 💥 Collision happens

✅ How Snowflake prevents this (IMPORTANT)

Correct implementation does:

if sequence == max {
    wait until next millisecond
}
👉 So instead of collision:

It blocks for ~1 ms
Then continues safely


Standard Snowflake bit allocation (64-bit)

Typical layout:

| 1 bit unused | 41 bits timestamp | 10 bits machine | 12 bits sequence |


Standard Snowflake uses:

41 bits timestamp (~69 years)
