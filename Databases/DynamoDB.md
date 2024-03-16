
When we create an item dynamoDB passes that partition key to internal hash function of dynamoDB.This hash function determains which partition it should be stored. 
The internal hash function of DynamoDB ensures data is spread evenly across available partitions.
<img width="1000" alt="Screenshot 2023-01-29 at 3 30 36 PM" src="https://user-images.githubusercontent.com/44682188/215318846-f6153d2a-2609-42b4-9dea-a9ec5c1a7a57.png">

Limits of a partition
The partition can contain a maximum of 10 GB of data. With size limit for an item being 400 KB, one partition can hold roughly more than 25000 (=10 GB/400 KB) items.

## When and How Partitions Are Created
#### Initial allocation of partitions

( readCapacityUnits / 3,000 ) + ( writeCapacityUnits / 1,000 ) = initialPartitions (rounded up)
Which means that if you specify RCUs and WCUs at 3000 and 1000 respectively, then the number of initial partitions will be ( 3_000 / 3_000 ) + ( 1_000 / 1_000 ) = 1 + 1 = 2.

After some time if RCUs and WCUs increased subsequently then existing partitions split into multiple partitions to handle the increased throughput. 
<img width="955" alt="Screenshot 2023-01-29 at 3 32 52 PM" src="https://user-images.githubusercontent.com/44682188/215318957-aaf016f7-771c-444f-b225-eef334978595.png">

And as soon as data size exceeds the maximum limit of 10 GB for the partition, DynamoDB splits the partition into two partitions.

<img width="952" alt="Screenshot 2023-01-29 at 3 33 31 PM" src="https://user-images.githubusercontent.com/44682188/215318978-dfad32fa-0b11-40e8-8e19-39e1c562771b.png">

Splitting happens in such a way that data and throughput capacity of an existing partition is evenly spread across newly created partitions.

One caveat to remember:
Items with the same partition key are stored within the same partition, and a partition can hold items with different partition keys.

## Hotspots
While designing data models we have to keep in mind about the hotspots which can be created. Like below
   - Social media posts: In a social media platform, a single post may receive a large number of likes, comments, and shares, resulting in a hot key for that post.
   - User profiles: In a user-based application, a popular user may receive a large number of requests for their profile information, resulting in a hot key for that user's profile.
   - Product catalog: In an e-commerce application, a popular product may receive a large number of requests for its details, resulting in a hot key for that product.

To prevent this we have distribute our parition keys as evenly as posisible like using post_id and user_id as partition keys which will prevent hot keys to some level and after that we can use caching and hot partition writes technique to reduce the severity further.
<img width="1179" alt="Screenshot 2023-01-29 at 3 57 36 PM" src="https://user-images.githubusercontent.com/44682188/215320285-340917b7-e498-424c-91ec-f25eb50c5312.png">

GSIs:
To speed up the queries of non key attributes we can use global secondary indexs. It contains selected items from the base table and not necessarily needed to have same partition/sort key.
Primary key attributes must be there as atleast non key attributes in the GSI Table.
In the GSI Query you can only fetch the attributes that are projected to GSI.
DyanamoDB only propagates data for which key attributes exists.
<img width="1122" alt="Screenshot 2023-07-03 at 11 47 30 PM" src="https://github.com/pavanpower8897/HighLevel/assets/44682188/81537b3f-47e4-4139-8914-f413f172b971">

Data will be updated in asynchronous manner from base table using eventually consistent model and GSIs inherit the read and write capacities from the base table.
In unlikely failure scenarios it might take some time for data to be populated on the GSIs, So your applications need to anticipate and handle situations where a query on a global secondary index returns results that are not up to date.
A table with many global secondary indexes incurs higher costs for write activity than tables with fewer indexes.


How scan exactly works ?
   - Does it sequentially checks all the partitions or some other way ?
   - There is a way to do parllel scan by spawning multiple child threads(workers) by defining some segment number(0 to n) and TotalSegments(n) to dynamodb, Which can utilize the max read throughput provisioned and do the parllel scan.
   - Generally scan retrieves 1MB of data to application and application requests again for next 1MB of data sequentially.
   - 
