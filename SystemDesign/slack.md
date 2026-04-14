Model 1 — USERINFO#USER_ID
Looks fine for basic user metadata. One thing to add: a status field (online/offline/away) should live in Redis (TTL-based), not DynamoDB, since it changes constantly and you'd be hammering writes otherwise.


SADD & SREM (Redis Sets)


What you have — and it's solid
userId → connectionId — presence check, correct. One lookup to know if user is online and which WS server holds their connection.
userId → activeGroupId — smart optimization. If user is currently looking at group G1, don't increment unread counter, just push the message directly to their WebSocket. Real-time delivery without polluting the badge count.
Unread counter per user per group — standard pattern, resets to 0 on channel open.


Gaps
Gap 1 — userId → activeGroupId is device-specific
What if the user has Slack open on laptop (viewing #general) and also on phone (viewing #random)? One userId → activeGroupId can only hold one value. You need:
userId → [connectionId_1: activeGroupId_A, connectionId_2: activeGroupId_B]
So the structure becomes userId → connectionId → activeGroupId, not a flat mapping.

Gap 2 — message delivery vs counter increment are not mutually exclusive
Your current logic:

User is in group view → push directly, skip counter
But what about their other device? User is viewing G1 on laptop, but phone is on a different channel. Phone still needs the counter incremented for G1.
The logic should be per-connection:
for each connection of userId:
    if connection.activeGroupId == incomingGroupId:
        push message directly
    else:
        increment unread counter
        push notification/badge update


Gap 3 — what happens when counter resets to 0
When user opens a group, counter resets. But what lastSeenMsgID do you store in DynamoDB (Model 5)? You need to write the latest MSG_ID at that moment so that if Redis goes cold and you need to recompute the unread count from DynamoDB, you can.
Counter reset should always be paired with a lastSeenMsgID write to DynamoDB — async is fine, but it must happen.

Gap 4 — user is online but idle (different tab/backgrounded app)
userId → connectionId tells you the user is connected. userId → activeGroupId tells you what they're looking at. But what if they're connected but the app is in the background on mobile? You'd push the message directly (because connection exists) but the user never actually sees it in real time. The counter stays 0 but they have unread messages visually.
Most apps solve this with an explicit "app in foreground" signal — client sends a focus/blur event over WebSocket when app goes background/foreground. Server uses this to decide push-directly vs increment-counter.

Revised Redis structure
# Connection registry
HSET user_connections:{userId}   {connectionId}  {serverId}

# Active view per connection
HSET user_active_group:{userId}  {connectionId}  {activeGroupId}

# Unread counters
HSET unread:{userId}   {groupId}  {count}

# App focus state per connection
HSET user_focus:{userId}  {connectionId}  {foreground|background}
Everything keyed by userId → connectionId so multi-device works naturally.

One question for you
When the user has zero active connections (fully offline) and messages arrive — are you incrementing the Redis counter directly, or are you relying on recomputing from lastSeenMsgID in DynamoDB when they reconnect? Both work, but they have different tradeoffs on Redis memory vs DynamoDB read cost at reconnect.Sonnet 4.6

Redis is volatile
If Redis restarts or the key evicts, you lose all unread counts permanently. User reconnects and sees 0 unread everywhere, even though they have 500 unread messages across 10 groups.

How to handle this
Two options:
Option A — Redis as cache, DynamoDB as source of truth
Every time unread counter increments in Redis, also write lastSeenMsgID to DynamoDB Model 5 asynchronously. On reconnect:
1. Check Redis unread:{userId} — exists? serve it
2. Cache miss? → read lastSeenMsgID from DynamoDB Model 5
             → Query Model 3: count msgs after lastSeenMsgID
             → repopulate Redis
             → serve it
Redis is fast path, DynamoDB is recovery path. This is the standard pattern.
Option B — Redis persistence (AOF)
Enable Redis AOF (Append Only File) persistence. Every write is logged to disk. On restart, Redis replays the log and recovers state. Much simpler operationally but adds disk I/O overhead and recovery time on restart.

What most production systems do
Both together actually:

AOF for fast recovery from restarts
DynamoDB lastSeenMsgID as the ultimate fallback for catastrophic Redis failure or data corruption


The recomputation query — one concern
When you recompute from lastSeenMsgID:
Query Model 3 where PK = GROUPMSG#{groupId} 
and SK > lastSeenMsgID
This works but if the user was offline for 7 days across 20 groups, that's 20 DynamoDB queries on reconnect. Fine for occasional cold starts, painful if Redis goes down and 50,000 users reconnect simultaneously.
This is why AOF is worth enabling — reduces the "Redis cold restart" scenario from a DynamoDB stampede to a non-event.
Does this align with your thinking, or do you want to explore a different durability tradeoff?

Redis cold start is when Redis loses all its in-memory data and starts completely empty.

When does it happen?

Redis server crashes and restarts without persistence enabled
Redis instance is replaced (deployment, scaling event)
Memory pressure causes key eviction
Cloud provider restarts the Redis node


Why it's painful in your case
Imagine Redis goes down at 2am. 50,000 users wake up at 9am and open Slack simultaneously.
All 50,000 users hit Redis → cache miss
→ All 50,000 fall through to DynamoDB
→ Each user queries Model 3 for 20 groups
→ 50,000 × 20 = 1,000,000 DynamoDB queries in seconds
→ DynamoDB gets overwhelmed
→ App is slow or down for everyone
This is called a thundering herd — everyone hitting the database at the same time because the cache is empty.

It's not just Redis restarts
Even a partial cold start causes this. For example:

Redis evicts keys due to memory limit (maxmemory policy)
Those users get a cache miss
If many users' keys evict around the same time → same stampede problem


How to protect against it
Three layers:
1. AOF persistence — Redis replays its log on restart, comes back warm. Thundering herd mostly avoided.
2. Staggered reconnection — when clients detect disconnection, they reconnect with random jitter (e.g. wait 0–30 seconds before reconnecting). Spreads the load instead of all hitting at once.
3. Request coalescing — if 1,000 users all miss cache for the same group's messages simultaneously, only fire one DynamoDB query for that group and fan the result out to all 1,000 waiters. Libraries like p-limit or a simple in-flight request map handle this.

The core idea is — Redis cold start turns a cache problem into a database problem, and at scale that database problem becomes an outage. AOF is the cheapest insurance against it.


Two separate problems
Problem 1 — Connection thundering herd
50,000 users reconnect simultaneously → 50,000 WebSocket handshakes hit your servers at once → servers fall over.
Jitter solves this. Has nothing to do with Redis or cache state.

Problem 2 — Redis cold start (cache miss stampede)
Redis is empty → every user reconnects → every user gets a cache miss → every user hits DynamoDB → DynamoDB gets overwhelmed.
Jitter does NOT solve this. Even if reconnections are spread over 30 seconds, every single one still hits empty Redis and falls through to DynamoDB. You just get a slower stampede, not a solved one.

What actually solves Redis cold start

AOF persistence — Redis comes back with data already in memory, cache misses never happen in the first place
Lazy repopulation — partially helps for shared data (group messages) but useless for per-user data (unread counts)
Request coalescing — limits concurrent DynamoDB queries for the same key but again, unread counts are per-user so doesn't help there
