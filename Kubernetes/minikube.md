# Contents

- [Install minikube](#install-minikube)
- [Configuration](#configuration)
  * [Get list of keys](#get-list-of-keys)
  * [View current config keys and values](#view-current-config-keys-and-values)
  * [Get config key value](#get-config-key-value)
  * [Set config key a new value](#set-config-key-a-new-value)
- [Usage](#usage)
  * [Create cluster and start](#create-cluster-and-start)
  * [Addons](#addons)
    + [List available addons](#list-available-addons)
    + [Install addon](#install-addon)
  * [Dashboard addon](#dashboard-addon)
    + [Create and run dashboard](#create-and-run-dashboard)
  * [Network and connection](#network-and-connection)
    + [Connect via service](#connect-via-service)
    + [Connect using tunnel](#connect-using-tunnel)
  * [Add nodes to minikube cluster](#add-nodes-to-minikube-cluster)
- [Cleanup](#cleanup)

# Install minikube

Get the latest release for your operating system
https://github.com/kubernetes/minikube/releases

# Configuration

## Get list of keys

```shell
minikube config
```

## View current config keys and values

```shell
minikube config view
```

## Get config key value

```shell
minikube config get driver
```


## Set config key a new value

```shell
minikube config set driver docker
```

# Usage

## Create cluster and start

To create and start earlier created cluster
Run
```shell
minikube start --driver=docker --nodes=1
```

Cluster will be automatically added to your kubectl config as a default context

Get status of cluset
```shell
minikube status
```

Stop cluster
```shell
minikube stop
```

## Addons

### List available addons

```shell
minikube addons list
```

### Install addon

```shell
minikube addons enable metrics-server
```

## Dashboard addon

### Create and run dashboard 
```shell
minikube dashboard
```

## Network and connection

### Connect via service

List available services in namespace `default`
```shell
minikube service list -n default
```

Connect service `service-name` to localhost port
```shell
minikube service -n default service-name
```

### Connect using tunnel

You can create tunnel to a minikube service.
Service should be only `loadBalancer` type

List available services in namespace `default`
```shell
minikube tunnel list -n default
```

Connect
```shell
minikube tunnel -n default
```

## Add nodes to minikube cluster

Before adding be aware of:
* https://github.com/kubernetes/minikube/issues/8055
* https://github.com/kubernetes/minikube/issues/10382

```shell
minikube node add
```

# Cleanup

```shell
# Stop cluster nodes
minikube stop

# Delete nodes resources 
minikube delete
# or
docker rm minikube
docker rm minikube-m02
docker network prune
docker volume prune

# Delete local config and keys
rm -rf ~/.minikube/
```
