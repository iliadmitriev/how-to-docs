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
```

Run instance

```bash
docker run -d  \
  --name broker-ssl -p 9092:9092 -p 9093:9093 \
  -v $(pwd):/etc/kafka/secrets \
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
