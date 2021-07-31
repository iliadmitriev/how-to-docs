# Install minikube

Get the latest release for your operating system
https://github.com/kubernetes/minikube/releases

# Usage

## Create cluster / start cluster

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
