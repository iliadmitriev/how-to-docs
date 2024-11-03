# Kafka tools

Create some data

```bash
kafka-topics --bootstrap-server localhost:9092 --create --topic test-topic --partitions 1
echo '{"message": "test message 1"}' | kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic
```

## Get Offsets for partitions

```bash
kafka-get-offsets --bootstrap-server localhost:9092 --topic test-topic
# or 
kafka-run-class kafka.tools.GetOffsetShell --bootstrap-server localhost:9092 --topic test-topic
```

```
test-topic:0:1
```

## Parse log file

Parse a log file and dump its contents to the console.

```bash
kafka-run-class kafka.tools.DumpLogSegments --help
```

Read and dump log file

```bash
kafka-run-class kafka.tools.DumpLogSegments  --files test-topic-0/00000000000000000000.log --deep-iteration --print-data-log
```

```
Dumping test-topic-0/00000000000000000000.log
Log starting offset: 0
baseOffset: 0 lastOffset: 0 count: 1 baseSequence: 0 lastSequence: 0 producerId: 1 producerEpoch: 0 partitionLeaderEpoch: 0 isTransactional: false isControl: false deleteHorizonMs: OptionalLong.empty position: 0 CreateTime: 1729573472640 size: 96 magic: 2 compresscodec: none crc: 3570448100 isvalid: true
| offset: 0 CreateTime: 1729573472640 keySize: -1 valueSize: 28 sequence: 0 headerKeys: [] payload: {"message": "test message1"}
```

## List producers

```bash
kafka-transactions --bootstrap-server localhost:9092 describe-producers --topic test-topic --partition 0
```

## Measure e2e Latency

```bash
kafka-e2e-latency localhost:9092 test-topic 5 1 100
```

## Delete messages

Add some more data

```bash
echo '{"message": "test message 2"}' | kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic
echo '{"message": "test message 3"}' | kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic
```

Create delete file

```bash
cat > __del.json << _EOF_
{"partitions":[{"topic":"test-topic","partition":0,"offset":1}]}
_EOF_
```

Delete (shift low watermark)

```bash
kafka-delete-records --bootstrap-server localhost:9092 --offset-json-file __del.json 
```

```bash
Executing records delete operation
Records delete operation completed:
partition: test-topic-0	low_watermark: 1
```

```bash
rm __del.json
```

check deleted

```bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic --from-beginning
```
