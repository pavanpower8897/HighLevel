Reward Service – DynamoDB Cost Optimization
The primary cost driver for the Reward Service DynamoDB table was Timed Storage, costing approximately $5,200/month (including Timed Storage + PITR).
After analyzing the dataset, we identified documents that could either be deleted or managed using TTL to reduce storage costs.
:gear: Actions Taken
• Deleted documents older than 6 months
• Added TTL (with jitter) to documents newer than (current time – 6 months) to ensure automatic expiration
:chart_with_downwards_trend: Cost Impact
• Data deletion savings: ~$2,250/month
• TTL-based future savings: ~$1,050/month (once items start expiring)
:dollar: Total expected monthly savings: ~ $3,300/month
:receipt: One-Time Operational Cost
• Running the deletion and TTL update operations incurred a one-time cost of ~$5,000
:rocket: Net Impact
• Payback period: ~1.5 months, after which the savings continue at ~$3,300/month
Attaching before [ss1] vs after [ss2] screenshots of day-wise timed storage costs.

$9,706/month total savings so far! :money_with_wings:
More performance and cost optimizations are in progress. Stay tuned
