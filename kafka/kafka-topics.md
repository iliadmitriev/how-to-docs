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

### Kafka topics with SSL

Create config file `connect.properties` with SSL options

```properties
security.protocol=SSL
ssl.keystore.location=keystore.jks
ssl.keystore.password=secret
ssl.key.password=secret
ssl.truststore.location=truststore.jks
ssl.truststore.password=secret
ssl.enabled.protocols=TLSv1.2,TLSv1.1,TLSv1
```

Create key and truststore using [KeyStore Explorer](https://keystore-explorer.org/) or [Openssl](https://github.com/iliadmitriev/openssl-scripts) and convert it to jks

Use connect.properties as command config with  `kafka-topics` and `kafka-configs`
```bash
kafka-topics --command-config connect.properties --bootstrap-server "SSL://localhost:9093" --list
```

```bash
kafka-configs --command-config connect.properties --bootstrap-server "SSL://localhost:9093" --describe --all --topic new_topic
```