# Stream Starter Project
### The first try to set up a sample quick start Kafka stream project.

## STEP 1: Get and Install Kafka
Go to [Apache Kafka](https://kafka.apache.org/) website and download Kafka version 3.0.0 (Scala 2.12).
Then extract the compressed file in a folder. Now Kafka is installed! :ok_hand: 

```shell
$ tar -xzf kafka_2.13-3.0.0.tgz
$ cd kafka_2.13-3.0.0
```


## STEP 2: Run The Kafka Environment
*NOTE: Your local environment must have Java 8+ installed* :wink: 
### Run ZooKeeper Service
```shell
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```
### Run Kafka Broker Service
```shell
$ bin/kafka-server-start.sh config/server.properties
```

## STEP 3: Create the needed topics 
```shell
$ bin/kafka-topics.sh --create --topic streams-input-topic --partitions 1 --replication-factor 1 --bootstrap-server localhost:9092
$ bin/kafka-topics.sh --create --topic streams-output-topic --partitions 1 --replication-factor 1 --bootstrap-server localhost:9092
```

#### Describe the topics
```shell
$ bin/kafka-topics.sh --describe --topic streams-input-topic --bootstrap-server localhost:9092
```

## STEP 4: Kafka in Java

It is the time to run your java application:
```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KTable;
import org.apache.kafka.streams.kstream.Named;
import org.apache.kafka.streams.kstream.Produced;

import java.util.Arrays;
import java.util.Properties;

public class StreamsStarterApp {

    public static void main(String[] args) {
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

        wordCountTable.toStream().to("streams-output-topic", Produced.with(Serdes.String(), Serdes.Long()));

        KafkaStreams streams = new KafkaStreams(builder.build(), config);
        streams.start();

        // printed the topology
        System.out.println(streams.toString());

        // Graceful Shutdown: shutdown hook to correctly close the streams application
        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

## STEP 5: Add some event to input topic
```shell
$ bin/kafka-console-producer.sh --topic streams-input-topic --bootstrap-server localhost:9092
> This is my first event
> This is my second event
> This is my third event
```

## STEP 6: Listen to the output topic
```shell
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic streams-output-topic    --from-beginning    --formatter kafka.tools.DefaultMessageFormatter --property print.key=true --property print.value=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
```
