1. Running the consumer with certain consumer group on certain topic it starts reading from beginning
   - docker exec -it kafka-2 kafka-console-consumer --bootstrap-server kafka-1:29092,kafka-2:29092 --topic test-topic --group test-group --from-beginning
   - docker exec -it kafka-2 kafka-console-consumer --bootstrap-server kafka-1:29092,kafka-2:29092 --topic test-topic --group test-group --from-beginning --property print.offset=true --property print.partition=true
   
   For publishing json payload we should use below one.
   docker exec -it kafka-1 kafka-console-producer \
  --broker-list kafka-1:29092,kafka-2:29092 \
  --topic test-topic \
  --property "parse.key=true" \
  --property "key.separator=:"
  
  Sample input: 1:{"id": 1, "name": "Alice"} (Remembers 1: before json payload)

2. For consuming from a partiticular offset 
   - docker exec -it kafka-2 kafka-console-consumer --bootstrap-server kafka-1:29092,kafka-2:29092 --topic test-topic --from-offset 5

3. Running producer which publishes messages on certain topic:
   - ``` docker exec -it kafka-1 kafka-console-producer --broker-list kafka-1:29092,kafka-2:29092 --topic test-topic ```

4.For checking which consumer is listening to which partition and the offset and lag values on a consumer group level
  - ``` docker exec -it kafka-2 kafka-consumer-groups --bootstrap-server kafka-1:29092,kafka-2:29092 --describe --group test-group ```

5. Checking how many paritions for a topic
   - ``` docker exec -it kafka-1 kafka-topics --bootstrap-server kafka-1:29092,kafka-2:29092 --describe --topic test-topic  ```


bin/zkServer.sh  start-foreground
bin/kafka-server-start.sh config/server.properties
echo dump | nc localhost 2181 | grep brokers
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic firstTopic --partitions 1 --replication-factor 1
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic firstTopic
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic firstTopic
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic firstTopic --from-beginning
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list (List of consumer groups)
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group console-consumer-57317
bin/kafka-reassign-partitions.sh --bootstrap-server localhost:2181 --topics-to-move-json-file topics-to-move.json --broker-list "2,3" --generate (Generates the json file for partition reassignment)

