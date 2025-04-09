# etcd cluster

## Install Mac OS

```shell
brew install etcd
```

## Install Alpine Linux

```shell
case $(uname -m) in \
  "aarch64") \
      wget https://github.com/etcd-io/etcd/releases/download/v3.5.0/etcd-v3.5.0-linux-arm64.tar.gz \
        -O /tmp/etcd-v3.5.0.tar.gz \
      ;; \
  "x86_64") \
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
sudo ifconfig lo0 alias 192.168.11.4
sudo ifconfig lo0 alias 192.168.11.5
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

## Run first node in cluster

Run first instance without data directory.

Data directory should be empty.

⚠️ Check `rm -rf data/etcd1`

```shell
etcd --data-dir=data/etcd1 --name=etcd1 --enable-v2=true \
     --listen-client-urls 'http://192.168.11.1:2379' \
     --advertise-client-urls 'http://192.168.11.1:2379' \
     --listen-peer-urls 'http://192.168.11.1:2380' \
     --initial-advertise-peer-urls 'http://192.168.11.1:2380' \
     --initial-cluster-state 'new' \
     --initial-cluster-token 'secrettoken'
```

## Add member to cluster

Create new member in cluster first

```shell
etcdctl --endpoints=192.168.11.1:2380 member add \
      etcd2 --peer-urls http://192.168.11.2:2380
```

It will output:

```shell
Member 59c7383825355e6a added to cluster f6c6734aee51507b

ETCD_NAME="etcd2"
ETCD_INITIAL_CLUSTER="etcd1=http://192.168.11.1:2380,etcd2=http://192.168.11.2:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.11.2:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
```

Where,

- `59c7383825355e6a` - is etcd2 member id
- `ETCD_INITIAL_CLUSTER` - is cluster advertise peers
- `ETCD_INITIAL_CLUSTER_STATE` - cluster state (should be `existing`, or etcd will attempt to create new cluster with new ID)

Check if there is no data directory from previous run

```shell
rm -rf data/etcd2
```

Put `ETCD_INITIAL_CLUSTER` to `--initial-cluster` and `ETCD_INITIAL_CLUSTER_STATE` to `--initial-cluster-state`

Run instance with name etcd2

```shell
etcd --data-dir=data/etcd2 --name=etcd2 --enable-v2=true \
     --initial-cluster "etcd1=http://192.168.11.1:2380,etcd2=http://192.168.11.2:2380" \
     --listen-client-urls 'http://192.168.11.2:2379' \
     --advertise-client-urls 'http://192.168.11.2:2379' \
     --listen-peer-urls 'http://192.168.11.2:2380' \
     --initial-advertise-peer-urls 'http://192.168.11.2:2380' \
     --initial-cluster-state 'existing' \
     --initial-cluster-token 'secrettoken'
```

## Check cluster members

```shell
etcdctl --endpoints=192.168.11.1:2380 member list -w table
# or
etcdctl --endpoints="192.168.11.1:2380,192.168.11.2:2380" member list -w table
```

It will output:

```
+------------------+---------+-------+--------------------------+--------------------------+------------+
|        ID        | STATUS  | NAME  |        PEER ADDRS        |       CLIENT ADDRS       | IS LEARNER |
+------------------+---------+-------+--------------------------+--------------------------+------------+
| 5022e613524bd2ec | started | etcd1 | http://192.168.11.1:2380 | http://192.168.11.1:2379 |      false |
| 59c7383825355e6a | started | etcd2 | http://192.168.11.2:2380 | http://192.168.11.2:2379 |      false |
+------------------+---------+-------+--------------------------+--------------------------+------------+
```

## Delete member from cluster

1. [Get member IDs](#check-cluster-members)
2. Run `remove` command:

```shell
etcdctl --endpoints=192.168.11.1:2380 member remove 59c7383825355e6a
```

3. Delete data directory:

```shell
rm -rf data/etcd2
```
