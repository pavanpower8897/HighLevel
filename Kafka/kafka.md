# Kafka 

Kafka can act as message broker for asynchronous communication between services. With below advantages:
   - Reliability: Kafka offers durable message storage where messages published to kafka are stored in these persisted storage as a log with some offset and retention period which ensures high reliability and durability.
   - Scalability: 
   - Atomicity: 
   - Loose coupling: Kafka acts as a buffer between services, decoupling producer and consumer, Services can produce and consume independly without the availability of the service.
   - Idempotency: 

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
In Kafka broker we are going to write and read data to topic partitions.
<img width="1478" alt="Screenshot 2023-02-19 at 5 47 52 PM" src="https://user-images.githubusercontent.com/44682188/219947455-0da295ea-4d40-40c5-8247-922c7f14e417.png">

# Kafka cluster
Kafka cluster can have multiple kafka brokers, Basically the reason for having multiple kafka brokers is for load sharing and data replication.
<img width="1540" alt="Screenshot 2023-02-19 at 5 43 43 PM" src="https://user-images.githubusercontent.com/44682188/219947447-e7a6f085-944c-4bb8-a7fe-6bd239b1f469.png">

# Kafka Topic
 - In kafka data is classified into topics where publish can publish certain related data on certain topic and consumer will consume that data from that topic. These messages can stay on disk based on size or time strategy where in case of time strategy we can say how long a msg can retain in kafka or we can provide maximum parition size so once that size is reached old data will start evacuating.

### No Kafka server can run without a zookeeper server. It is mandatory to run the zookeeper server.

While publishing the kafka messages we can explicitly mention the partition if not then kafka broker user its default partitioner which uses a hash function on key level to decide which parittion it should write in a evenly distributed messages accorss the partitions. Sarama go package also provides a way to implement a custom paritioner where we can use customer algorithms for message partition distribution.

# Kafka consumer group 

Kafka consumer group is ntng but a list of consumers(In aws ec2 instances) who wants to share the partitions for consuming messages on kafka topic. Only one consumer can listen on certain partition so whenever a consumer is getting added to a consumer group partition assigner rebalance and assigns the
partitions based on the situation. 
Only one consumer can be assigned to one partition.

# Consumer Offset 
It refers to the position of consumer in a kafka topic partition.When a consumer group reads from a Kafka topic, each partition of the topic is assigned to one of the consumers in the group. The consumer group keeps track of the last offset that each consumer read in each partition. This offset indicates the position of the last record that was read by the consumer in that partition for a consumer group.

It make sures that same record not being consumed multiple times within a consumer group.

Kafka also provides a built-in topic called "__consumer_offsets" that stores the latest committed offset for each partition in a consumer group

# Consumer Lag Metrics


Whenever a new consumer is going up in a consumer group, partition assigner will rebalance the partitions and partitioner ensures that data is distributed evenly.

When a consumer group is created or rebalancing is triggered due to changes in the consumer group (e.g., addition or removal of consumers), the Group Coordinator is responsible for executing the partition assignment algorithm. This algorithm takes into account the current state of the group, the topic partitions, and the consumers' capabilities to determine the partition assignment.

For each partition, there is a leader and multiple replicas across the Kafka cluster. The leader is responsible for handling read and write requests for that partition, while the replicas replicate the data for fault tolerance.

It's worth noting that starting from Apache Kafka version 2.4, the default hashing algorithm for message partitioning has been changed to the Java default hashing algorithm (FNV-1a). However, the Murmur2 algorithm is still supported and can be explicitly configured if desired.

If size of the parition limit reaches then there is no auto resize or split policy , system administrator has to manually repartition or ading data retention policy so that old data will be archieved from the partition.
