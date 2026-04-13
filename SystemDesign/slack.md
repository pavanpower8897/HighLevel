Model 1 — USERINFO#USER_ID
Looks fine for basic user metadata. One thing to add: a status field (online/offline/away) should live in Redis (TTL-based), not DynamoDB, since it changes constantly and you'd be hammering writes otherwise.

