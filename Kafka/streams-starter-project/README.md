# Stream Starter Project
### The first try to set up a sample quick start Kafka stream project.

## STEP 1: Get and Install Kafka
Go to [Apache Kafka](https://kafka.apache.org/) website and download Kafka version 3.0.0 (Scala 2.12).
Then extract the compressed file in a folder. Now Kafka is installed! :ok_hand: 

```shell
tar -xzf kafka_2.13-3.0.0.tgz
cd kafka_2.13-3.0.0
```


## STEP 2: Run The Kafka Environment
*NOTE: Your local environment must have Java 8+ installed* :wink: 
### Run ZooKeeper Service
```shell
bin/zookeeper-server-start.sh config/zookeeper.properties
```
### Run Kafka Broker Service
```shell
bin/kafka-server-start.sh config/server.properties
```

## STEP 3: Create the needed topics 
```shell
bin/kafka-topics.sh --create --topic streams-input-topic --bootstrap-server localhost:9092
bin/kafka-topics.sh --create --topic streams-output-output --bootstrap-server localhost:9092
```

#### Describe the topics
```shell
bin/kafka-topics.sh --describe --topic streams-input-topic --bootstrap-server localhost:9092
```

## STEP 4: Kafka in Java

```java
Properties config = new Properties();
config.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-starter-app");
config.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
config.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
config.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

StreamsBuilder builder = new StreamsBuilder();

KStream<String, String> wordCountInput = builder.stream("streams-input-topic");

KTable<String, Long> wordCountTable = wordCountInput
        .mapValues(value -> value.toLowerCase())
        .flatMapValues(value -> Arrays.asList(value.split(" ")))
        .selectKey((key, value) -> value)
        .groupByKey()
        .count(Named.as("Counts"));

wordCountTable.toStream().to("streams-output-output", Produced.with(Serdes.String(), Serdes.Long()));

KafkaStreams streams = new KafkaStreams(builder.build(), config);
streams.start();
```

## STEP 5: Listen to the output topic
```shell
bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```
