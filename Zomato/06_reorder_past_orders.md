-> We created a new ES Cluster to power the reorder past orders listing where we have provided different new filters like Open now, dishes, Meal time, Relevancy which has aggregation , rating etc..

- We create two listners one is backfill listener and another one is real time listener, First we enabled the real time listener and its started indexing the real time orders to new ES index
- After this first we migrated the zoman orders using backfill listener through ETL job which publishes the data to offline kafka
- After this we enabled the new filters which we are directly queries on ES like rating, dishIds, meal time etc.
- And after that we introduced relevancy here we used ES gauss function for orderedAt and field value factor for countSoFar to give more weightage to most ordered and recent orders
      - But we have obsereved that lot of orders are showing one after another which felt like redundant, So we thought of doing aggregation and here initially we thought of group by using `orderIdentity` Hash field which is the combination of catalogue, chain info which can uniquely identify group of similar orders , But we observed around 500ms time for this aggregation queryes as orderIdentity is strings and strings are not optimized for group by in ES due to various internally handling like string char matching and memory issues, So then we thought of adding a new integer field which represents orderIdentity and on which we can aggregate then if we create a new numeric field generator then there are higher chances of collision, So we decided to use firstOrderId of that orderIdentity Hash as firstLinkedOrderId and started using that for groupBy then we observed the latencies within in 20ms 
- After this change we ran indexing for zomans and powered the aggregation in relevancy
- Once everything is stable for zomans we started migrating the actual user data , Initially we started with 100k and gradually increased it to 1million while monitoring the upstream data sources and services like dynamoDB, Redis and Elastic search and composite service CPU, We havent observed any significant limits, Actually here we have kept dynamodb under provisioning only which saved lot of cost like merely 30 to 40dollors , So when increasing the throughput suddenly like few 100k rpm we suddently freaked out as the dynamodb consumption reaches the provisioning limits but luckily it didnt crossed it, So after that once dynamo provision increased to significant buffer we increased it.
- Here at peak time we also observed CPU alerts on redis, as during lunch peak more containers were added and due to this more consumptions was consumed and it increased the throughput to 1.5 to 2million which is not guessed before so alert was triggered and it havent crossed 50% even though we have scaling post 50%
- And around 1.5 to 2milion we have observed that ES IOPS also reaching its limit around 4500IOPS, so we havent scaled it further
- And also we havent estimated the node sizes properly so have intially added as 200GB and then increased it to 450GB and total 5 nodes.
- We were doing userID Level routing where all the user document will be found in one index node and also one thing is we have created different indexes on year level, and doing parllel queries on those indexes.And creating next year index using a cron which runs everyday and ignores if already next year node created
- And after sometime the previous aggregation queries and remaining query latencies has increased significantly which we havent observed before, So we thought its due to indexing and switched of the aggregation but as without aggregation we were getting similar orders at same place which again few folks raised that concern so we disabled the new filter flow and decided to enable the new flow once everything is stable and indexing migration is done.
- As we gaussed as the reads were not there at the time of indexing, So the latencies were increased  but after that we enabled for 1% of users and as the  nodes started warmingup the latencies started decreasing from more than 500ms to less than 100ms
- And in between we also observed that on search keyword we were using wildcard query, For this query also latency has shooted up during indexing and then we releized that this is not optimized and rolled back to previous search index, Ideally we should have used different catalogue word key words ins searchTerms but we have used the slugName which is just one combination word, As search is not critical we thought of reindexing everything with seperate word search terms later
- And later aggregation query also we optimized as its giving a bit un expected released, By smoothening the mealtime hour using gauss and
- Total around 2.4 billion records were indexed

    <img width="1428" alt="Screenshot 2024-11-05 at 1 05 00 AM" src="https://github.com/user-attachments/assets/327ef7fb-5525-47ba-a541-e808b95db97e">

-> Below is the final aggregation we have used with mealtime hour crazy 24hour logic as the gauss has to take the min diff for handling times like 11pm and 2am the gap should be 3 hours but as we storing orderHourMeal whici 2300, its not directly possible so we have used below logic

```
if currentHourMinute > 1200 {
		// Time is after 12 PM
		orderHourBucket1Origin = -int(2400 - currentHourMinute)
		orderHourBucket2Origin = currentHourMinute
		orderHourBucket1FilterOperators[ese.GreaterThanEqual] = 0
		orderHourBucket1FilterOperators[ese.LessThan] = timeAfter12

		orderHourBucket2FilterOperators[ese.GreaterThanEqual] = timeAfter12
	} else {
// Time is before 12 PM

		// For next 12 hours from the current hour minute we can consider the origin as current hour minute as its within 12hr range
		orderHourBucket1Origin = currentHourMinute
		orderHourBucket1FilterOperators[ese.GreaterThan] = 0
		orderHourBucket1FilterOperators[ese.LessThanEqual] = timeAfter12

		//After 12hrs from the current hour we need to consider below origin for handling time to mealtime hour handling
// For example if currentHourMinute is 0200(2AM) and orderHourMinute is 2300(11PM) then gauss should consider the distance as 300 (Using below example:(2400+200)-2300=300)
		orderHourBucket2Origin = 2400 + currentHourMinute
		orderHourBucket2FilterOperators[ese.GreaterThan] = timeAfter12
		orderHourBucket2FilterOperators[ese.LessThan] = 2400
	}
```

```
 "function_score": {
      "query": {
        "bool": {
          "filter": [
            {
              "term": {
                "userId": 417372
              }
            }
          ],
          "must_not": [
            {
              "terms": {
                "visibilityState": [
                  "Hidden"
                ]
              }
            }
          ],
        "should": [
            {
              "exists": {
                "field": "userId"
              }
            }
          ],
          "minimum_should_match": 1
        }
      },
      "functions": [
           {
            "filter": {
              "range": {
                "orderedHourMinute": {
                  "gte": 0,
                  "lt": 1300
                }
              }
            },
            "gauss": {
              "orderedHourMinute": {
              "origin": 100,
              "scale": 90,
              "decay": 0.2
            }
            },
            "weight": 6
        },
        {
            "filter": {
              "range": {
                "orderedHourMinute": {
                  "gte": 1300,
                  "lt": 2359
                }
              }
            },
            "gauss": {
               "orderedHourMinute": {
                "origin": 2500,
                "scale": 90,
                "decay": 0.2
            }
            },
            "weight": 6
        },
        {
          "gauss": {
            "orderedAt": {
              "origin": "now",
              "scale": "30d",
              "decay": 0.3
            }
          },
          "weight": 4
        },
        {
            "filter": {
              "range": {
                "countSoFar": {
                  "gt": 50
                }
              }
            },
            "weight": 0.6
        },
        {
          "filter": {
            "range": {
              "countSoFar": {
                "lte": 50
              }
            }
          },
           "field_value_factor": {
              "field": "countSoFar",
              "factor": 0.01,
              "modifier": "sqrt"
          }
        }
      ],
      "score_mode": "sum"
    }
```


We also create a new seperate RPC to fetch all user ordered resIds and dishIds add the query pattern here for that. 
Initially we used to derive everything from PS which is very unoptimized and then we migrated to this approach.
