
# Zookeper
Open source apache service which provides centralized service for managing configurations, naming, synchronization over a large clusters in
distributed systems

For example following tasks:
- Cluster coordination: Helps kafka brokers coordinates with each other by maintainer the state, If a new brokers added or removed zookeeper updates the state of the cluster.
   - What is the need to mantain the state ? There are different advantages of maintaining kafka broker state cuz if a node failure occured we should pick another kafka broker for doing reads/writes on parition level and also it will be used for load balancing
   - 
- Leader election: Kafka broker uses zookeeper to elect a leader for each partition of a topic.The leader is responsible for all writes and reads on certain partitions while followers replicate the data from leaders to to ensure fault tolerance. 
   - What is fault tolerance ?
   - Zooker container the partition to leader mapping ?
   - 
- Configuration management: Zookeeper helps kafka broker manage configuration information such as topic configuration, broker configuration etc. 

# Kafka broker
<img width="1540" alt="Screenshot 2023-02-19 at 5 43 43 PM" src="https://user-images.githubusercontent.com/44682188/219947447-e7a6f085-944c-4bb8-a7fe-6bd239b1f469.png">

# Kafka cluster
Kafka cluster can have multiple kafka brokers, Basically the reason for having multiple kafka brokers is for load sharing and data replication.
<img width="1478" alt="Screenshot 2023-02-19 at 5 47 52 PM" src="https://user-images.githubusercontent.com/44682188/219947455-0da295ea-4d40-40c5-8247-922c7f14e417.png">
