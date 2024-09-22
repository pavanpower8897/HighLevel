
Elastic search stores the index data into segments which acts as sub indexes and whenever a doc is inserted to ES its going to crete a new small segment which contains the inverted index 
info and document field storage info and vector term info (like term frequency etc..)  , Here in inverted index it contians the map of terms to docIDs for example.

And important thing to remember here is segments are immutable

Now elastic search basically mergs the smaller segments is a comparitevly big segment through background processes asyn way for optimizing the number of parllel processing over all small segments.

Elastic search going to search on all these segments and merges the data and returns the info.


Whenever a document is deleted then lucine does this as a soft delete and it actully maintains a bitset of 1/0 which indicated whether a doc is deleted or not
And each bit indicates the document id inside a segment this is different than global document id, each bitset is mapped to the segment and this segment contain internal document id ranging 1 to ... 1000 ... etc.. 


When Lucene writes data it first writes to an in-memory buffer, When the data in the Buffer reaches a certain amount, it will be flushed to become a Segment, Before segments are flushed or committed, data is stored in memory and is unsearchable. This is another reason why Lucene is said to provide near-real time and not real-time queries

So when Lucene is searching internally, it makes a two-phase query. The first phase is to list the DocId’s found to contain the given Term, and the second phase is to find the Doc based on the DocId. Lucene provides functionality to search by Term as well as to query on the basis of DocId.


- Why more index commits increases the latency ?
Because during index commits ES flushes in memory buffer where it gathers the latest updates/insertions/deletions data to disk, Now lucine will create the segments in the disk along with some meta data
So here important thing to note here is disk operations contains I/O and generally takes more time access the data / write compared to RAM even with latest SSDs and more over its a shared resource, So if there are more frequent commits its going to consume more disk resources which effects the queries happening on that disk.

-> Why fsync is required after commiting from ES application ?
Because aftering commit the data the OS generally cache its it in memory for waiting to write the  batch data to disks, So inbetween if the system crashes the data will be lost which for application it thinks data is already phycically stored but in reality that is not that case.
so fsync forcefully pushes or flushes in memory OS buffer to dish and it also increase the Disk IO and affeects other operations.
