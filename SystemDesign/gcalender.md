![IMG_0085](https://github.com/user-attachments/assets/e46038ab-59d1-45fa-8d16-3b76f500b746)
![IMG_0084](https://github.com/user-attachments/assets/08e0fc3c-a739-4937-94cd-d8af8df83280)


Critical issues you missed:

1. No read/write separation Your architecture routes everything through one path. At 1B RPM, you need separate read and write paths. Meeting reads (fetching calendars) vastly outnumber writes — a read replica layer or CQRS pattern is essential here.

2. Conflict detection is underspecified You wrote "notify conflict" but Google Calendar's hardest problem is conflict resolution — what happens when two users book the same room simultaneously? You need optimistic locking or a compare-and-swap strategy at the DB level, not just a notification after the fact.

3. No caching layer At 600M users, repeatedly fetching a user's calendar from DynamoDB on every view is prohibitively expensive. You need a Redis/Memcached layer in front of the DB for hot calendar data (today's events, upcoming reminders).

4. Timezone handling is missing entirely This is one of the most notoriously tricky parts of calendar systems. Do you store times in UTC and convert on read? What about recurring events that cross DST boundaries? This needs an explicit decision in your design.

5. Recurring events have no data model Your notes mention scheduling but don't address how recurring events are stored. Storing each instance is wasteful (a weekly meeting for a year = 52 rows). The standard approach is storing a rule (RRULE) + exceptions, which has significant query complexity implications.

6. Notification delivery guarantees You have Kafka noted, but there's no mention of at-least-once vs exactly-once delivery, retry logic, or what happens when a notification fails. For reminders (especially time-sensitive ones), this is critical.

7. No API gateway / auth layer Your diagram jumps straight from client to DynamoDB. In practice you need an API gateway + AuthN/AuthZ layer (who can see whose calendar? free/busy vs full details?).

8. Eventual consistency tradeoffs not addressed DynamoDB is eventually consistent by default. For calendar invites, if User A sends an invite and User B immediately opens their calendar, will they see it? You need to decide where strong consistency is required.



<img width="866" height="624" alt="Screenshot 2026-04-11 at 12 01 49 PM" src="https://github.com/user-attachments/assets/061aae22-b8c2-4237-a139-fd4348ee97d5" />
