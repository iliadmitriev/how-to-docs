## Kafka topics


List topics.

```bash
kafka-topics --bootstrap-server localhost:9092 --list
```

Create topic.

```bahs
kafka-topics --bootstrap-server localhost:9092 --create --topic new_topic
```

Read topic config.

```bash
kafka-topics --bootstrap-server localhost:9092 --describe --topic new_topic
```

Add more partitions to topic

Partitions can be added but canot be deleted. Operations is irreversable.

```bash
kafka-topics --bootstrap-server localhost:9092 --alter --topic new_topic --partitions 4
```

## Kafka Config


List all configs for topic

```bash
kafka-configs --bootstrap-server localhost:9092 --describe --all --topic new_topic
```

Set retention time in msecs `retention.ms`.

```bash
kafka-configs --bootstrap-server localhost:9092 --alter --topic new_topic --add-config retention.ms=10000
```

Remove retention time config option

```bash
kafka-configs --bootstrap-server localhost:9092 --alter --topic new_topic --delete-config retention.ms
```