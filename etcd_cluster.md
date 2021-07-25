# Install etcd

## Mac OS

```shell
brew install etcd
```

## Alpine Linux 
```shell
case $(uname -m) in \
  "aarch64") \
      wget https://github.com/etcd-io/etcd/releases/download/v3.5.0/etcd-v3.5.0-linux-arm64.tar.gz \
        -O /tmp/etcd-v3.5.0.tar.gz \
      ;; \
  "amd64") \
      wget https://github.com/etcd-io/etcd/releases/download/v3.5.0/etcd-v3.5.0-linux-amd64.tar.gz \
        -O /tmp/etcd-v3.5.0.tar.gz \
      ;; \
esac \
  && mkdir /opt/etcd \
  && tar -xvf /tmp/etcd-v3.5.0.tar.gz -C /opt/etcd --strip-components=1
```

## Create hosts and IPs

```shell
sudo ifconfig lo0 alias 192.168.11.1
sudo ifconfig lo0 alias 192.168.11.2
sudo ifconfig lo0 alias 192.168.11.3
sudo ifconfig lo0 alias 192.168.12.4
sudo ifconfig lo0 alias 192.168.13.5
```

```shell
sudo bash -c 'cat >> /etc/hosts << EOF__ 
192.168.11.1 etcd1
192.168.11.2 etcd2
192.168.11.3 etcd3
192.168.11.4 etcd4
192.168.11.5 etcd5
EOF__'
```