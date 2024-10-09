
**Yes, in an Elasticsearch cluster, there is only one active master node at any given time. Here’s a more detailed explanation:


Elastic search stores the index data into segments which acts as sub indexes and whenever a doc is inserted to ES its going to crete a new small segment which contains the inverted index 
info and document field storage info and vector term info (like term frequency etc..)  , Here in inverted index it contians the map of terms to docIDs for example.

And important thing to remember here is segments are immutable

Whenever we are updating the documents new segments will be created and older segments will be marked as deleted, so we will observe lot of delete operations.

Now elastic search basically mergs the smaller segments is a comparitevly big segment through background processes asyn way for optimizing the number of parllel processing over all small segments.

Elastic search going to search on all these segments and merges the data and returns the info.


Whenever a document is deleted then lucine does this as a soft delete and it actully maintains a bitset of 1/0 which indicated whether a doc is deleted or not
And each bit indicates the document id inside a segment this is different than global document id, each bitset is mapped to the segment and this segment contain internal document id ranging 1 to ... 1000 ... etc.. 


When Lucene writes data it first writes to an in-memory buffer, When the data in the Buffer reaches a certain amount, it will be flushed to become a Segment, Before segments are flushed or committed, data is stored in memory and is unsearchable. This is another reason why Lucene is said to provide near-real time and not real-time queries
** But because we are writing to in memory buffer if there is any crash there is chance of data loss and it creates inconsistent system, So Elastic search handles this by first writing to trans log to in memory and flushes the translog to disk (Here its configurable to wait for translog to be flushed to disk or asyn it can and return the ack for faster processing , performance vs durability) 
ES on system crash and its recovery it reads the last commit point and reply the trans log operattion which are stored after the commit point. 
Last Commit Point: Elasticsearch maintains a commit point or marker that indicates the last operation that was successfully flushed to disk and associated with a segment. This is essential for recovery after a crash.
The commit point is updated every time a flush occurs. Essentially, this is a record of the last operation that has been safely persisted in the segments.
Translogs on the Primary: Each primary shard in Elasticsearch has its own translog that logs all write operations. When a write operation occurs, it is recorded in the translog of the primary shard.
Translogs on Replicas: The replicas also have their own translogs, which are used to ensure data consistency. When a primary shard performs a write operation, that operation is sent to its replicas to be logged in their respective translogs.


So when Lucene is searching internally, it makes a two-phase query. The first phase is to list the DocId’s found to contain the given Term, and the second phase is to find the Doc based on the DocId. Lucene provides functionality to search by Term as well as to query on the basis of DocId.


- Why more index commits increases the latency ?
Because during index commits ES flushes in memory buffer where it gathers the latest updates/insertions/deletions data to disk, Now lucine will create the segments in the disk along with some meta data
So here important thing to note here is disk operations contains I/O and generally takes more time access the data / write compared to RAM even with latest SSDs and more over its a shared resource, So if there are more frequent commits its going to consume more disk resources which effects the queries happening on that disk.

-> Why fsync is required after commiting from ES application ?
Because aftering commit the data the OS generally cache its it in memory for waiting to write the  batch data to disks, So inbetween if the system crashes the data will be lost which for application it thinks data is already phycically stored but in reality that is not that case.
so fsync forcefully pushes or flushes in memory OS buffer to dish and it also increase the Disk IO and affeects other operations.

-> How elastic cache invalidates the query cache which it cached for queries ?
Elastic cache do two type of query caching
1. Node caching (Ntng but segment caching) : it caches the segments which are involved in the queries so next time if the same segment involved in the query it checks in the cache and if it
   misses then it checks in the disk.


-> How elastic search executes a filter query ?
  - Its generally a two step process one is DISK I/O operation where it fetches the segments to RAM
  - And CPU loads the information from RAM and applies the filter
  - And then the filtered segments are cached in the memory on segment level
  - Elasticsearch uses a combination of memory mapping and segment loading:

     Suppose range query on created at field is there
    - It first tries to fetch the created_at field inverted index info (Maybe it uses the bucketing to minizes the info or in naive case its going to load all the inverted index created at field values
    - Now CPU nows the possible values now its going to do the range query and then shortlist the docIDs
    - Now its going load the segments for those docIDs from memory/ OS page cache/ Disk (If the segment is getting frequently accessed between filters etc.. then whole segment will be fetched and cached)

   Multiple filter execution like field: active, name: pavan
   - In this cases it fetches the docIDs by direct term matching the inverted indexing using memory map for each filter individually
   - CPU takes the intersection or UNION or bitset operation for logic operations to shortlist the docIDs
   - Fetch the docIDs info for further result gather

   - Uniqueness: The uniqueness of the key comes from the specific combination of query structure and parameters. Even a small change in the query (e.g., changing the range values) would result in a different cache key.
   - When the same query is run again, Elasticsearch checks the cache for the cache key.
If a cache hit occurs, it retrieves the results. However, before returning the cached results, it will check the segment metadata to ensure that the segments associated with the cache entry have not changed.
If the relevant segments have been updated (e.g., their version numbers have changed), the cache entry is invalidated, and the query is re-executed against the current segments.

  - <img width="898" alt="Screenshot 2024-09-23 at 1 31 49 AM" src="https://github.com/user-attachments/assets/6f9e4fee-361b-493f-b8cc-4abd41ee8378">

  Distributed elastic search query execution
<img width="991" alt="Screenshot 2024-09-23 at 1 56 47 AM" src="https://github.com/user-attachments/assets/46d7afd2-c12c-43e6-87df-823e66f2f9eb">

<img width="826" alt="Screenshot 2024-09-23 at 2 00 38 AM" src="https://github.com/user-attachments/assets/b28f4fe1-ab46-44cc-9aef-c0dee07ecb9c">

If any master node fails another node will be elected by voting and one who gots majority of votes will be assigned as master
I was curious and asked doesnt it become a single point of failure, but ES master doesnt do much execpt cluster management redirection and node health and other coordination jobs
mainly data processing indexing will be done by data nodes 
And at the end merging and pagination also does by coordination nodes which can be more thatn one , Master node delegates the whole thing to coordination node. And on master node typically no data will be stored and no query processing happens 


Yes, that's correct! In Elasticsearch, when a query involves aggregation (such as a "group by" operation), the final grouping and consolidation of results are handled by the coordinating node after it receives partial results from the data nodes. Here’s how this process works:


The coordinating node does not maintain its own separate query cache. Instead, it relies on the query results cached at the data nodes when routing requests and aggregating results.

In addition to the node query cache, there is also a shard query cache at the data node level that stores the results of aggregations. This cache is specifically for the results of aggregation queries and can also speed up subsequent identical aggregation queries.

Understant the exact difference between node cache and shard cache ??? 


The design allows for fault tolerance. If the active master node fails, the cluster will automatically elect a new master, ensuring continuous operation.

But yes temperoary desruption is there till another master node will be up after the election process.


So all are ES servers only just using lucine libraries in the ES Server code ?

<img width="1050" alt="Screenshot 2024-09-23 at 2 14 49 AM" src="https://github.com/user-attachments/assets/71ac3b0c-7235-45bc-bd6a-8cbb9214abc9">

Overall Execution Flow
- Filters (narrow down the documents).
- Query (applies scoring if necessary).
- Sorting (sorts the filtered and/or scored documents).
- Aggregations (group by or calculate metrics after filtering, querying, and sorting).


- Scoreing functions will be applied after query filters are evaluated on matching documents.
- Query and filters execution are Interleaved Execution , ES executes the filters parllely or sequentially which is optimized with least expensive operation first
- Term queries are the most efficient queries that Elasticsearch supports: their matches are pre-computed in the inverted index structure. (https://www.elastic.co/blog/elasticsearch-query-execution-order) , Based on the query type like term / conjuction / disjunction it estimates the cost and then executes the queries in the way in which it will cost less cost and ES doesnt expose the information and here the ordering of my queries/filter doesnt matter as ES automitically reorder according to its score.
- Do filters get executed before or after queries? -> We cant say everything is interleaved 
- In summary, Elasticsearch executes queries and filters in an interleaved manner based on cost optimization rather than strictly one after another

-> Total hits in the query response means total documents matching your query
---**For aggegation cardinality of field also matters as it ES uses the data structure like global cardinals to maintain the map of unique strings and it will be done in coordinator node also at further reduce phase which can have incremental latency issue.n 
-> Size of the field also matters in the case of aggregations...

***
numeric and string data are stored, indexed, and processed in Lucene’s internal data structures.
String fields are stored as inverted index which is optimized for exact match searches but less efficient for sorting and aggregations
Numeric Fields(integers, longs, floats, doubles) Stored in optimized block k-d tree (BKD tree) which is efficient for range queries and space
Why keyword fields are less efficient than numeric fields for aggregations ? Aggregations will be performed by lucene and it depends on how the fields are internally structured
When aggregating on keyword fields, Elasticsearch internally hashes the string values to keep track of unique values. -> Hashing takes more time
Elastic search has to keep track of all these unique hashes in memory for maintaining the group which memory and CPU Intensive compared to numeric fields
Lexicographical Comparisons when strings matched for aggregations (his is less efficient than numeric comparisons due to the way string sorting works:)
Memory Overhead During Aggregation

***
