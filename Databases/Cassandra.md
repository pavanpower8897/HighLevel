### Cassandra 

- cassandra is an open source column-oriented data store.
   - What is column oriented data store: Data is stored in such a way where columns are grouped together where as in traditional row-oriented data store rows are stored together.
Ex: In a customers table name, address , email fields present 
In column oriented data store: All name fields will be stored together etc.
In row oriented data stroe: Each customer data stored together.

- In cassendra there is no concept of master replic or primary replica where the first commit happens, Instead all nodes are equal and commits can happen on any
replica.
  - When a write request is made it first writes the request to one node and returns successful instead of waiting for data to be propagated to other replica nodes, 
as it wants to be write-first  which prioritizes writes over read consistency. This allows for high write availability and low latency.However its worth noting that
reads are eventually consistent using 'Hinted Handoff' technique.
  - While doing writes we can specify consistency level as 'ONE' or 'QUORUM' where 'ONE' means write request will be successful after writing to one replica node where as
'QUORUM' write request will be successful after writing to majority of replicas like if we have 3 nodes write will be sucecssfull after writing to 2 nodes its to 
handle balance between write latency and read consistency.

- Cassendra is said to be column but its actually a hybrid as we can also store all row attributes together, There is a concept called super column which group together multiple columns in one column and helps in retrieving specific attribute fields more efficiently.

- In DynamoDB partition and sort key contains only one attribute where as in cassendra can contain more than one attribute Ex: In users model partition key can be both first name and last name. Parition key in both dynamodb and cassendra used to identifty physical location of the partition node where data is stored.
- In cassendra with the increase in number of partitions the overhead to manage the partitions increase as cassenra is a eventually consistent data store where data will be 
