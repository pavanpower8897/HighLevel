1. Running the consumer with certain consumer group on certain topic it starts reading from beginning
   - docker exec -it kafka-2 kafka-console-consumer --bootstrap-server kafka-1:29092,kafka-2:29092 --topic test-topic --group test-group --from-beginning
   - docker exec -it kafka-2 kafka-console-consumer --bootstrap-server kafka-1:29092,kafka-2:29092 --topic test-topic --group test-group --from-beginning --property print.offset=true --property print.partition=true

2. For consuming from a partiticular offset 
   - docker exec -it kafka-2 kafka-console-consumer --bootstrap-server kafka-1:29092,kafka-2:29092 --topic test-topic --from-offset 5

3. Running producer which publishes messages on certain topic:
   - ``` docker exec -it kafka-1 kafka-console-producer --broker-list kafka-1:29092,kafka-2:29092 --topic test-topic ```

4.For checking which consumer is listening to which partition and the offset and lag values on a consumer group level
  - ``` docker exec -it kafka-2 kafka-consumer-groups --bootstrap-server kafka-1:29092,kafka-2:29092 --describe --group test-group ```

5. Checking how many paritions for a topic
   - ``` docker exec -it kafka-1 kafka-topics --bootstrap-server kafka-1:29092,kafka-2:29092 --describe --topic test-topic  ```
