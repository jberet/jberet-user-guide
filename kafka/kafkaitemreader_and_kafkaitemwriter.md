# KafkaItemReader and KafkaItemWriter

Apache Kafka is a distributed publish-subscribe messaging system. jberet-support module includes `kafkaItemReader` and `kafkaItemWriter` to read from or write to Kafka topics. `kafkaItemReader` reads and binds data to instances of custom POJO bean provided by the batch application.

`kafkaItemReader` keeps track of the current read position, including current topic name, topic partition number, and topic partition offset. Therefore, it is recommended to disable Kafka auto commit in Kafka consumer properties, e.g., `enable.auto.commit=false`. Kafka consumer properties are specified in batch property `configFile`.

This reader class supports retry and restart, using the tracked read position as checkpoint info. It is also recommended to turn off Kafka consumer automatic group management; instead manually assign topics and partitions for the consumer. See batch property `topicPartitions`.

The following dependencies are required for `kafkaItemReader` and `kafkaItemWriter`:

```xml
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>kafka-clients</artifactId>
  <version>0.9.0.0</version>
</dependency>
```

## Batch Configuration Properties in Job XML

`kafkaItemReader` and `kafkaItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml. All properties are of type String, unless noted otherwise. The following is an example job xml that references `kafkaItemReader` and `kafkaItemWriter`:

```xml
<chunk item-count="5">
  <reader ref="kafkaItemReader">
    <properties>
      <property name="configFile" value="kafka-consumer.properties"/>
      <property name="topicPartitions" value="#{jobParameters['topicPartitions']}"/>
      <property name="pollTimeout" value="#{jobParameters['pollTimeout']}"/>
    </properties>
  </reader>
  ...
</chunk>
```

```xml
<chunk item-count="5">
  ...
  <writer ref="kafkaItemWriter">
    <properties>
      <property name="configFile" value="kafka-producer.properties"/>
      <property name="topicPartition" value="#{jobParameters['topicPartition']}"/>
      <property name="recordKey" value="#{jobParameters['recordKey']}"/>
    </properties>
  </writer>
</chunk>
```

### Batch Properties for Both `kafkaItemReader` and `kafkaItemWriter`

#### configFile

The file path or URL to the Kafka configuration properties file. See Kafka docs for valid property keys and values.

### Batch Properties for `kafkaItemReader`

#### topicPartitions

`java.util.List<String>`

A list of topic-and-partition in the form of "topicName1:partitionNumber1, topicName2:partitionNumber2, ". For example, "orders:0, orders:1, returns:0, returns:1".

#### pollTimeout

`long`

The time, in milliseconds, spent waiting in poll if data is not available. If `0`, returns immediately with any records that are available now. Must not be negative.


### Batch Properties for `kafkaItemWriter`

#### topicPartition

A topic partition in the form of `<topicName>:<partitionNumber>`. For example, "orders:0". Unlike KafkaItemReader, which accepts multiple TopicPartition as source, this writer class only accepts 1 TopicPartition as destination.

#### recordKey

The key used when sending `ProducerRecord`.