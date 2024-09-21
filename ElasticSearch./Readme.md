
Elastic search stores the index data into segments which acts as sub indexes and whenever a doc is inserted to ES its going to crete a new small segment which contains the inverted index 
info and document field storage info and vector term info (like term frequency etc..)  , Here in inverted index it contians the map of terms to docIDs for example.

And important thing to remember here is segments are immutable

Now elastic search basically mergs the smaller segments is a comparitevly big segment through background processes asyn way for optimizing the number of parllel processing over all small segments.

