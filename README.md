# Kafka Streams Word Count

This is a tutorial project that demonstrates how to build and run a Kafka Streams application to count words from a stream of input text.

## Project Structure

```
Kafka Root Directory
├── bin
├── config
├── libs
├── licenses
├── logs
├── site-docs
└── streams-quickstart/
    ├── pom.xml
    └── src/
        └── main/
            └── java/
                └── myapps/
                    └── WordCount.java
```

---

## Getting Started

### 1. Create Kafka Topics

Start the Kafka server and create the input and output topics:

```bash
# Start the Kafka Server
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
bin/kafka-storage.sh format --standalone -t $KAFKA_CLUSTER_ID -c config/server.properties
bin/kafka-server-start.sh config/server.properties

# Create input and output topics
bin/kafka-topics.sh --create \
    --bootstrap-server localhost:9092 \
    --replication-factor 1 \
    --partitions 1 \
    --topic streams-plaintext-input
bin/kafka-topics.sh --create \
    --bootstrap-server localhost:9092 \
    --replication-factor 1 \
    --partitions 1 \
    --topic streams-wordcount-output \
    --config cleanup.policy=compact
```

---

### 3. Build the Project

```bash
cd streams-quickstart
mvn clean package
```
---

### 4. Run the WordCount Application

```bash
mvn exec:java -Dexec.mainClass="myapps.WordCount"
```

---

### 5. Produce Sample Input

Use Kafka CLI to send text data:

```bash
bin/kafka-console-producer.sh --topic streams-plaintext-input --bootstrap-server localhost:9092
```

Type input lines like:

```
hello world
kafka streams word count
hello kafka
```

---

### 6. Consume Output
Open a new terminal, execute:
```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic streams-wordcount-output \
    --from-beginning \
    --property print.key=true \
    --property print.value=true \
    --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
    --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
```

You should see output like:

```
hello    1
world    1
kafka    1
streams  1
word     1
count    1
hello    2
kafka    2
```

---

## How It Works
In WordCount.java
```java
KStream<String, String> source = builder.stream("streams-plaintext-input");

source
  .flatMapValues(value -> Arrays.asList(value.toLowerCase(Locale.getDefault()).split("\W+")))
  .groupBy((key, word) -> word)
  .count(Materialized.as("counts-store"))
  .toStream()
  .to("streams-wordcount-output", Produced.with(Serdes.String(), Serdes.Long()));
```

- **flatMapValues**: Splits input lines into individual words.
- **groupBy**: Regroups by word.
- **count**: Aggregates word counts.
- **toStream().to(...)**: Writes the result to the output topic.

---

## Cleanup

To delete topics:

```bash
bin/kafka-topics.sh --delete --topic streams-plaintext-input --bootstrap-server localhost:9092
bin/kafka-topics.sh --delete --topic streams-wordcount-output --bootstrap-server localhost:9092
```
To shutdown the application:
```bash
ctrl + c 
```

---

## Troubleshooting

- Match your Java version with `pom.xml` configuration.
- Add Kafka Streams dependencies in `pom.xml` if missing.

