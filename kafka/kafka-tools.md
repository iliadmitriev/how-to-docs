# Kafka tools

Parse a log file and dump its contents to the console.

```bash
kafka-run-class kafka.tools.DumpLogSegments --help
```

Read and dump log file
```bash
kafka-run-class kafka.tools.DumpLogSegments  --files /bitnami/kafka/data/<topicName>-0/00000000000000096852.log --deep-iteration --print-data-log
```

