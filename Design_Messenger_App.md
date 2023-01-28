
# Purpose
Messenger is a social media application used for text, audio, video communication between users.
# Functional requirements
- Sending text, images to other users (Friends)
- Showing list of friends with their active/inactive status
- Audio & Video call option.

# Non functional requirements 
- High availability 
- Low latency
- Durability 


# Scale estimation 

# Database 
If we observer chat apps contains huge amount of increasing data. So we need to perform queries on hugh data base. Assuming we need to build this application at high scale for large number of users NoSQL will be the best choice as its horizontal scalable by adding new nodes and particularly we can
use DynamoDB as its completely managed by AWS. 
The challenge here is to design data models in DynamoDB. For that first we need to identify all data access paterns and based on that we can build tables.
### Data access patterns
  - Fetch friends and groups from user_id 
  - Fetch one to one conversation data from user_id 
  - Fetch group conversation data
  - Send msg to group or friend(1:1) 

# Components involved 

# Design architecture 
