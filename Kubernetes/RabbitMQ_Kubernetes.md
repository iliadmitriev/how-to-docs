# Before start

We use minikube deployment
```shell
# create minikube with two nodes, ingress, metrics-server and dashboard
minikube start --nodes=2 --addons=ingress,metrics-server,dashboard
```
Open dashboard
```shell
minikube service -n kubernetes-dashboard kubernetes-dashboard
```

# install RMQ operator

Install operator
```shell
kubectl apply -f "https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml"
```
It will create 
* new namespace `rabbitmq-system`
* new custom resource `RabbitmqCluster`
* role, cluster role, role bindings, cluster role binding and service account
* deployment of `rabbitmq-cluster-operator`

Check if custom resource `RabbitmqCluster` is installed
```shell
kubectl get customresourcedefinitions.apiextensions.k8s.io
# NAME                                   CREATED AT
# rabbitmqclusters.rabbitmq.com   2021-08-08T07:11:06Z
```

# Install RabbitMQ cluster

Features
* the latest alpine version of community image
* 2 replicas
* 500Mi memory
* 0,5 vCPU core
* 2Gb of persistent storage
* securityContext for containers and initContainers
* default user `guest` with password `guest` with administrative privileges and default vhost `guest`
```yaml
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  labels:
    app: rmq-taskcamp
  annotations:
    some: annotation
  name: rmq-taskcamp
spec:
  replicas: 2
  image: rabbitmq:3.9.1-management-alpine
  rabbitmq:
    additionalConfig: |
      default_user = guest 
      default_pass = guest
      default_vhost = guest
      default_user_tags.administrator = true
  resources:
    requests:
      cpu: 500m
      memory: 500Mi
    limits:
      cpu: 500m
      memory: 500Mi
  persistence:
    storage: 2Gi
  override:
    statefulSet:
      spec:
        template:
          spec:
            containers: []
            securityContext: {}
            initContainers:
            - name: setup-container
              securityContext: {}
```

# Cluster administrative web UI

Create self-signed certificates using [this script](https://github.com/iliadmitriev/openssl-scripts#usage)
```shell
./create_cert.sh rmq.taskcamp.info
```

Create TLS secret
```shell
kubectl create secret tls rmq-taskcamp-tls \
  --key rmq.taskcamp.info.key \
  --cert rmq.taskcamp.info.pem
```

Add `127.0.0.1 rmq.taskcamp.info` to `/etc/hosts`
```shell
sudo sed -i '' -e '$a\'$'\n''127.0.0.1 rmq.taskcamp.info'$'\n'  /etc/hosts
```

Install service of type NodePort and Ingress 
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    some: annotation
  labels:
    app: rmq-taskcamp
    app.kubernetes.io/component: rabbitmq
    app.kubernetes.io/name: rmq-taskcamp
    app.kubernetes.io/part-of: rabbitmq
  name: rmq-management-nodeport
spec:
  ports:
  - name: management
    port: 15672
    protocol: TCP
    targetPort: 15672
  selector:
    app.kubernetes.io/name: rmq-taskcamp
  type: NodePort
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rmq-management-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: rmq.taskcamp.info
    http:
      paths:
      - backend:
          service:
            name: rmq-management-nodeport
            port:
              number: 15672
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - rmq.taskcamp.info
    secretName: rmq-taskcamp-tls
```

Start tunnel
```shell
minikube tunnel
```

Get default username and password from rabbitmq cluster secret
```shell
kubectl get secrets rmq-taskcamp-default-user \
   -o jsonpath='{.data.default_user\.conf}' | base64 -D
```
Output
```
default_user = aSAQQWBBK7jb-_J0YYX5o8zojA7iWVwi
default_pass = _iVFRZWlP6PMBhMhwYvAW8TKP3dPyGkV
```

Go to `https://rmq.taskcamp.info`
and use default username and password

# Test cluster

Create pod which produce benchmark load to service `rmq-taskcamp` (user, password and vhost are `guest`)
```shell
kubectl run perf-test --image=pivotalrabbitmq/perf-test -- \
   --uri "amqp://guest:guest@rmq-taskcamp/guest"
```

Watch load
```shell
kubectl logs -f perf-test
```

Stop load pod
```shell
kubectl delete pod perf-test
```

# References
1. [RabbitMQ Operator](https://www.rabbitmq.com/kubernetes/operator/quickstart-operator.html)
2. [Using RabbitMQ Operator](https://www.rabbitmq.com/kubernetes/operator/using-operator.html)
3. [Cluster formation and peer discovery](https://www.rabbitmq.com/cluster-formation.html#peer-discovery-aws)
4. [Deploying RabbitMQ on Kubernetes using RabbitMQ Cluster Operator](https://medium.com/nerd-for-tech/deploying-rabbitmq-on-kubernetes-using-rabbitmq-cluster-operator-ef99f7a4e417)