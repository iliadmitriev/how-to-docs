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

Create client, server keystores and truststore using [KeyStore Explorer](https://keystore-explorer.org/) or [Openssl](https://github.com/iliadmitriev/openssl-scripts) and convert it to jks
1. Root CA key pair (key and certificate) adding CA extention 
2. Server key pair signed with Root CA (Sing new key pair) and added extention *TLS Web Server Authentication (1.3.6.1.5.5.7.3.1)*
3. Client key pair signed with Root CA and added extention *TLS Web Client Authentication  (1.3.6.1.5.5.7.3.2)*
Export Root CA to trustsore,  save server and client key pairs to keystores

Use connect.properties as command config with  `kafka-topics` and `kafka-configs`
```bash
kafka-topics --command-config connect.properties --bootstrap-server "SSL://localhost:9093" --list
```

```bash
kafka-configs --command-config connect.properties --bootstrap-server "SSL://localhost:9093" --describe --all --topic new_topic
```


## Kafka consumer groups

```bash
./kafka-consumer-groups --bootstrap-server "localhost:9092" --group worker-group --describe
```
