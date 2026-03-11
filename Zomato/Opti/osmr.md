Platform Services Cost Optimisation Thread] :take_my_money:  @channel 

We've completed multiple infra, storage, throughput, profiler related optimisations leading to significant monthly savings:

Completed :white_check_mark:

Deprecated redundant calls from DeliveryCharge service to LocationService
Reduction of 450K RPM i.e. 30% of total service throughput resulting in reduction of peak running ECS tasks 36 → 24 
:moneybag: ~$800/month saved

Profiler-driven optimisations on LocationService
Optimisations in memory allocations in DA jumbo logging, redundant function calls removed from POI flows, resulting in reduction in 3 ECS tasks on peak
:moneybag:~$200/month saved

OpenSearch descale on Eternal Location Service
c7g.2xlarge.search to c7g.xlarge.search along with reduction of number of node from 6 to 3, without any increase in latency and with CPU util in bounds
:moneybag: ~$1,500/month saved

Salience Calculation Consumer Deprecation
Figured out alternative sources of salience calculation, and skipped DDT computations for all restaurants on HP
Peak ECS tasks of this worker: 47 → 6 
:moneybag: ~$1,600/month saved

ElasticCache cache.r7g.xlarge shards will be reduced from 25 to 20 
:moneybag: ~$1,600/month : to be saved

Further to deprecate the related jumbo table 
:moneybag: $1,474.09/month : to be saved


OSRM Throughput Reduction
Blinkit side caching (at hex 11) and a 70% promo/demand manager throughput reduction have decreased OSRM peak tasks by 25–30%, with further reductions expected from District. Peak running ECS tasks 450 → 290 
:moneybag: ~$1,700/month saved



------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Ongoing:hourglass_flowing_sand:

GSI deprecation on LocationService DDB
Old DA computation GSI with almost no reads, and only writes
:moneybag: ~$500/month : to be saved  (storage + pitr)

TimeService DDB Data archival
Current cost and entries:
$1700 (storage) + $1400 (pitr)
Entries to be deleted 3,119,222,376 (avg size 1,900 bytes) - 95% of the total entries present in table

Savings:
:moneybag: Estimated monthly cost savings ~ $2800
One time cost for deletion ($0.71 per million write request units + $0.1425 per million read request units) - $4,875 + $200 (ECS cost) : Payback period-2 months


OSRM Redis shard count reduction
Autoscaling threshold to be increased from 50-60%, along with min shard count reduction + reduced throughput : resulting in ~4 chache.m6g.large shards reduction.

User service profiler-driven optimisations
Optimised redundant ser-deserialisation and added system-level timezone preloading at service startup, expected 15% CPU profile opti resulting in reduction of around 5 ECS tasks at peak



:moneybag:$5,800/month total savings so far
