# Kafka producer

```shell
kcat -P -b 10.1.1.1:9093 \
     -t my-topic \
     -X security.protocol=SSL \
     -X ssl.key.location=key.pem \
     -X ssl.certificate.location=cert.pem \
     -X ssl.ca.location=ca.pem -e | wc -l
```

# Kakfa consumer

## Count messages

```shell
kcat -C -b 10.1.1.1:9093 \
     -t my-topic \
     -X security.protocol=SSL \
     -X ssl.key.location=key.pem \
     -X ssl.certificate.location=cert.pem \
     -X ssl.ca.location=ca.pem -e | wc -l
```

## Start consuming from end

```shell
kcat -C -b 10.1.1.1:9093 \
     -t my-topic \
     -X security.protocol=SSL \
     -X ssl.key.location=key.pem \
     -X ssl.certificate.location=cert.pem \
     -X ssl.ca.location=ca.pem -e \
     -o end
```

## Start consuming from end with group of consumers autorebalancing partitions
```shell
kcat -C -b 10.1.1.1:9093 \
     -X security.protocol=SSL \
     -X ssl.key.location=key.pem \
     -X ssl.certificate.location=cert.pem \
     -X ssl.ca.location=ca.pem \
     -G my-group my-topic
```
