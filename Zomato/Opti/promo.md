[Promo-Service Performance & Cost Optimizations Update] :rocket:

Timezone Loading Optimisation (Infra + Syscall Reduction)
While profiling promo-service, we discovered that a utility function GetTimeByTimezone was consuming disproportionate CPU due to repeated time.LoadLocation() on every execution, leading to repeated filesystem lookups and unnecessary syscalls across multiple hot request paths.

:hammer_and_wrench: What We Changed
• Introduced system-level timezone preloading at service startup
• Cached commonly used timezones (Asia/Kolkata, UTC) as reusable *time.Location instances
• Replaced direct time.LoadLocation() calls in hot paths with cached loader

:zap:Impact
• Total Syscall Time dropped from 18.59s to 12.46s (~33% CPU reduction)
• Peak running tasks reduced from 220 → 187 (~15% reduction)

Karma RPC Optimisation
We rely on Karma to identify fraudulent restaurants and block discounts accordingly — a critical safeguard in the promo flow. Previously, every campaign evaluation and discount application triggered a Karma RPC call to validate merchant eligibility.
On analyzing discount-blocking patterns, we observed that merchant fraud status doesn't not change frequently within short intervals. We introduced request-level caching for fraud evaluation results, storing merchant fraud status for a defined TTL.

:bar_chart: Impact

1.2M throughput reduction (~99%+) in Karma traffic from promo flows :chart_with_downwards_trend: 
Improved system efficiency without compromising fraud protection


Auto-Applied Discount Evaluation
While profiling, we identified three core inefficiencies in auto apply discount evaluation: redundant expression parsing, repetitive computation across identical items, and suboptimal concurrency placement.

:hammer_and_wrench:Here’s a breakdown of what we did to improve the performance:
 • Cached parsed expressions to eliminate repeated JSON unmarshaling, reflection, and struct allocations and memoized evaluation results by rule hash, shared across items and use-cases (e.g., Deal of the Day, Flash Sale)
 • Shifted parallelism from expression resolution → item resolution layer reducing goroutine churn and CPU scheduler contention

:fire: Impact

70% CPU Reduction of this function (5.01s → 1.52s) and around 26% reduction overall.
p99 Latency dropped from 78ms → 57ms (~27% improvement) in our campaigning RPC


Additional savings from the above optimizations ~$3500 per month :moneybag:
$6,406 /month total savings so far! :money_with_wings:
More performance and cost optimizations are in progress. Stay tuned
