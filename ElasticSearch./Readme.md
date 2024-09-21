
Elastic search stores the index data into segments which acts as sub indexes and whenever a doc is inserted to ES its going to crete a new small segment which contains the inverted index 
info and document field storage info and vector term info (like term frequency etc..)  , Here in inverted index it contians the map of terms to docIDs for example.

And important thing to remember here is segments are immutable

Now elastic search basically mergs the smaller segments is a comparitevly big segment through background processes asyn way for optimizing the number of parllel processing over all small segments.

Elastic search going to search on all these segments and merges the data and returns the info.


Whenever a document is deleted then lucine does this as a soft delete and it actully maintains a bitset of 1/0 which indicated whether a doc is deleted or not
And each bit indicates the document id inside a segment this is different than global document id, each bitset is mapped to the segment and this segment contain internal document id ranging 1 to ... 1000 ... etc.. 


When Lucene writes data it first writes to an in-memory buffer, When the data in the Buffer reaches a certain amount, it will be flushed to become a Segment, Before segments are flushed or committed, data is stored in memory and is unsearchable. This is another reason why Lucene is said to provide near-real time and not real-time queries

So when Lucene is searching internally, it makes a two-phase query. The first phase is to list the DocId’s found to contain the given Term, and the second phase is to find the Doc based on the DocId. Lucene provides functionality to search by Term as well as to query on the basis of DocId.

