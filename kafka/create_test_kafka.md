# Create test kafka cluster

## Apache image without SSL

```bash
docker run -d  \
  --name broker -p 9092:9092 \
  -e KAFKA_NODE_ID=1 \
  -e KAFKA_PROCESS_ROLES=broker,controller \
  -e KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
  -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@localhost:9093 \
  -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1 \
  -e KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS=0 \
  -e KAFKA_NUM_PARTITIONS=4 \
  apache/kafka:latest
```

## Apache image with SSL

Create certificates (depends on java keytool and openssl)

```bash
git clone git@github.com:iliadmitriev/openssl-scripts.git certs
cd certs

# Generate CA
./create_ca.sh

# Generate Server key and certificate
./create_jks.sh kafka localhost

# Generate Client key and certificate
./create_jks.sh client localhost

cd ../

cp certs/kafka.* ./
cp certs/client.* ./
cp certs/root.pem ./client.ca.pem
```

Run instance

```bash
docker volume create broker-data
docker volume create broker-config

docker run -d  \
  --name broker-ssl -p 9092:9092 -p 9093:9093 \
  -v $(pwd):/etc/kafka/secrets \
  -v broker-data:/var/lib/kafka/data \
  -v broker-config:/mnt/shared/config \
  -e KAFKA_NODE_ID=1 \
  -e KAFKA_PROCESS_ROLES=broker,controller \
  -e KAFKA_LISTENERS=PLAINTEXT://:9092,SSL://:9093,CONTROLLER://:9094 \
  -e KAFKA_SSL_KEYSTORE_FILENAME=kafka.keystore.jks \
  -e KAFKA_SSL_TRUSTSTORE_FILENAME=kafka.truststore.jks \
  -e KAFKA_SSL_KEY_CREDENTIALS=kafka.key.pas \
  -e KAFKA_SSL_KEYSTORE_CREDENTIALS=kafka.keystore.pas \
  -e KAFKA_SSL_TRUSTSTORE_CREDENTIALS=kafka.truststore.pas \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092,SSL://localhost:9093 \
  -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@localhost:9094 \
  -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1 \
  -e KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS=0 \
  -e KAFKA_NUM_PARTITIONS=4 \
  apache/kafka:latest
```

### Kafka tools

Generate `client-connect.properties` file

```bash
cat > client-connect.properties << _EOF_
security.protocol=SSL
ssl.keystore.location=client.keystore.jks
ssl.keystore.password=$(cat client.keystore.pas)
ssl.key.password=$(cat client.key.pas)
ssl.truststore.location=kafka.truststore.jks
ssl.truststore.password=$(cat kafka.truststore.pas)
ssl.enabled.protocols=TLSv1.2,TLSv1.1,TLSv1
_EOF_
```

List and describe server topics with `kafka-topics`

```bash
kafka-topics --command-config client-connect.properties \
    --bootstrap-server "SSL://localhost:9093" --describe
```

Create a new topic `topic1`

```bash
kafka-topics --command-config client-connect.properties \
    --bootstrap-server "SSL://localhost:9093" --create --topic topic1
```

Consume messages from `topic1`

```bash
kafka-console-consumer --consumer.config client-connect.properties \
    --bootstrap-server "SSL://localhost:9093" --topic topic1
```

Send message to `topic1`

```bash
echo '{"message": "hello world"}' |
  kafka-console-producer --producer.config client-connect.properties \
    --bootstrap-server "SSL://localhost:9093" --topic topic1
```

### kcat tool

List topics with `kcat`

```bash
kcat -b ssl://127.0.0.1:9093 \
  -X security.protocol=SSL \
  -X ssl.key.location=client.key.pem \
  -X ssl.certificate.location=client.cert.pem \
  -X ssl.ca.location=client.ca.pem \
  -X ssl.endpoint.identification.algorithm=none \
  -L
```

Create properties file

```bash
cat > kcat.properties << _EOF_
security.protocol=SSL
ssl.ca.location=client.ca.pem
ssl.keystore.location=client.p12
ssl.keystore.password=$(cat client.keystore.pas)
_EOF_

# Or alternatively

cat > kcat.properties << _EOF_
security.protocol=SSL
ssl.ca.location=client.ca.pem
ssl.key.location=client.key.pem
ssl.certificate.location=client.cert.pem
_EOF_
```

```bash
kcat -b localhost:9093 -F kcat.properties -L
```

Consume `topic1` with `kcat`

```bash
kcat -b localhost:9093 -F kcat.properties -C -t topic1
```

Produce message to `topic1` with `kcat`

```bash
echo '{"message": "hello world"}' | kcat -b localhost:9093 -F kcat.properties -P -t topic1
```
