# kubectl

## Namespaces (ns)

List namespaces
```shell
kubectl get namespaces
# or 
kubectl get ns
```

Create a new namespace named `namespace-name`
```shell
kubectl create ns namespace-name
```

Delete namespace named `namespace-name`
```shell
kubectl delete ns namespace-name
```

Set default namespace used in context
```shell
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# check
kubectl config view --minify | grep namespace:
```

## Workloads

Kubernetes provides several built-in workload resources:

* `Deployment` (a higher-level concept) and `ReplicaSet`
* `StatefulSet`
* `DaemonSet`
* `Job` and `CronJob`

### Deployment (deploy)

Get all deployments
```shell
kubectl get deployments
kubectl get deploy
kubectl get deployments -n namespace
kubectl get deployments --all-namespaces
```