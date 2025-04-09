# Install confluent-kafka

Install openssl, librdkafka, pkg-config libraries

```shell
brew install openssl@1.1 librdkafka pkg-config
```

Install confluent-kafka package with pip

```shell
CFLAGS=$(PKG_CONFIG_PATH="/opt/homebrew/opt/openssl@1.1/lib/pkgconfig" pkg-config --cflags rdkafka) \
LDFLAGS=$(pkg-config --libs rdkafka) \
pip install confluent-kafka
```
