
- DynamoDB is a key-value and document oriented store which is fully managed by AWS, where Apache cassendra is a column oriented data store.
- In DynamoDB partition and sort key contains only one attribute where as in cassendra can contain more than one attribute Ex: In users model partition key can be both first name and last name. Parition key in both dynamodb and cassendra used to identifty physical location of the partition node where data is stored.
- In cassendra with the increase in number of partitions the overhead to manage the partitions increase as cassenra is a eventually consistent data store where data will be 
