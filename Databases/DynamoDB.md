
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
