# Kafka tools

Create some data

```bash
kafka-topics --bootstrap-server localhost:9092 --create --topic test-topic --partitions 1
echo '{"message": "test message1"}' | kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic
```

## Get Offsets for partitions

```bash
kafka-run-class.sh kafka.tools.GetOffsetShell --bootstrap-server localhost:9092 --topic test-topic
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
