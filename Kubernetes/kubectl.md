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

* `Deployment` a higher-level concept of `ReplicaSet`
* `StatefulSet`
* `DaemonSet`
* `Job` and `CronJob`

### Deployment (deploy)

#### Get all deployments
```shell
kubectl get deployments
kubectl get deploy
kubectl get deployments -n namespace
kubectl get deployments --all-namespaces
```

#### Create a new deployment

Let's create deployment of nginx:1.20-alpine image of 3 replicas with selector `app=nginx` with port 80 inside container
`nginx-deployment.yaml` file:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20-alpine
        ports:
        - containerPort: 80
```

```shell
kubectl apply -f nginx-deployment.yaml
```

An alternative way to create this deployment with command line
```shell
kubectl create deployment deployment-nginx --image=nginx:1.20-alpine --replicas=3
```
but in this case selector label will be created by the name of deployment `deployment-nginx`.

Also you can add `-dry-run` and `-o yaml` to a `create`-command to output yaml template

Check deployment is created
```shell
# get deployment by name 
kubectl get deploy nginx-deployment
# get associated replicasets by selector app=nginx
kubectl get rs -l app=nginx 
# get pods by selector 
kubectl get po -l app=nginx
```

#### Edit deployment

Edit deployment using standard editor 
```shell
kubectl edit deploy nginx-deployment
```
using nano
```shell
KUBE_EDITOR="nano" kubectl edit deploy nginx-deployment
```

#### Delete deployment

```shell
kubectl delete deploy nginx-deployment
```

### Pods (po)

#### Get pods

```shell
# get all pods in current namespace (long form)
kubectl get pods
# get all pods in all namespaces
kubectl get po --all-namespaces
# get all pods in designated namespace 
kubectl get po -n namespace
# get all pods by selector label
kubectl get po -l app=nginx
# get pod by name pod-name
kubectl get po pod-name
```

Show labels of pods 
```shell
kubectl get po --show-labels
```

#### Execute shell command inside a pod

Interactively run shell `sh` inside `pod-name` pod
```shell
kubectl exec -ti pod-name -- sh
```

#### Watch logs of a pod

watch logs of a `pod-name` pod
```shell
kubectl logs -f pod-name
```

you can specify a deployment
```shell
kubectl logs -f deployment/nginx-deployment
```
then kubernetes will choose one of pods available for this deployment