# kubectl

- [kubectl](#kubectl)
- [Namespaces](#namespaces)
- [Workloads](#workloads)
    * [Deployments](#deployments)
        + [Get all deployments](#get-all-deployments)
        + [Create a new deployment](#create-a-new-deployment)
        + [Edit deployment](#edit-deployment)
        + [Delete deployment](#delete-deployment)
    * [Pods](#pods)
        + [Get pods](#get-pods)
        + [Execute shell command inside a pod](#execute-shell-command-inside-a-pod)
        + [Watch logs of a pod](#watch-logs-of-a-pod)
        + [Create interactive temporary pod with shell](#create-interactive-temporary-pod-with-shell)
    * [StatefulSet](#statefulset)
        + [Create StatefulSet](#create-statefulset)
        + [Network Identities of Stateful Set](#network-identities-of-stateful-set)
- [ConfigMaps](#configmaps)
    * [Get ConfigMap](#get-configmap)
    * [Create ConfigMap](#create-configmap)
    * [Edit ConfigMaps](#edit-configmaps)
    * [Delete ConfigMaps](#delete-configmaps)
    * [Using ConfigMaps](#using-configmaps)
        + [Environment variables from ConfigMap with envFrom](#environment-variables-from-configmap-with-envfrom)
        + [Single environment variable from ConfigMap with valueFrom](#single-environment-variable-from-configmap-with-valuefrom)
        + [Mount ConfigMap as a volume directory](#mount-configmap-as-a-volume-directory)
        + [Mount single file from ConfigMap](#mount-single-file-from-configmap)
- [Labels](#labels)
    * [Get resource labels](#get-resource-labels)
    * [Use labels for filtering resources](#use-labels-for-filtering-resources)
        + [Filtering condition operations](#filtering-condition-operations)
    * [Add labels to pod](#add-labels-to-pod)
    * [Remove labels from pod](#remove-labels-from-pod)
    * [Using labels as a resource selectors](#using-labels-as-a-resource-selectors)
        + [Matching condition types](#matching-condition-types)
- [Scaling](#scaling)
    * [Manual scaling](#manual-scaling)
- [Network](#network)
    * [Services](#services)
        + [Get services](#get-services)
        + [Create ClusterIP](#create-clusterip)
        + [Create nodePort](#create-nodeport)
        + [Create LoadBalancer](#create-loadbalancer)
- [Resources](#resources)


# Namespaces

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

Set current namespace used in context
```shell
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# check
kubectl config view --minify | grep namespace:
```

# Workloads

Kubernetes provides several built-in workload resources:

* `Deployment` a higher-level concept of `ReplicaSet`
* `StatefulSet`
* `DaemonSet`
* `Job` and `CronJob`

## Deployments

### Get all deployments
```shell
# get all deployments in current namespace
kubectl get deployments
# get all deployments in current namespace using short reference
kubectl get deploy
# get all deployments in a namespace
kubectl get deployments -n namespace
# get all deployments in all namespaces
kubectl get deployments --all-namespaces
# get deployment named nginx in current namespace
kubectl get deployment nginx
```

### Create a new deployment

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
kubectl create deployment deployment-nginx \
     --image=nginx:1.20-alpine --replicas=3 --port 80
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

### Edit deployment

Edit deployment using standard editor 
```shell
kubectl edit deploy nginx-deployment
```
using nano
```shell
KUBE_EDITOR="nano" kubectl edit deploy nginx-deployment
```

### Delete deployment

```shell
kubectl delete deploy nginx-deployment
```

## Pods

### Get pods

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

### Execute shell command inside a pod

Interactively run shell `sh` inside `pod-name` pod
```shell
kubectl exec -ti pod-name -- sh
```

### Watch logs of a pod

watch logs of a `pod-name` pod
```shell
kubectl logs -f pod-name
```

you can specify a deployment
```shell
kubectl logs -f deployment/nginx-deployment
```
then kubernetes will choose one of pods available for this deployment

### Create interactive temporary pod with shell

```shell
kubectl run test-pod --image=nginx:1.20-alpine --rm -it \
          --restart=Never -- sh
```
* test-pod - name of temporary pod
* nginx:1.20-alpine - image

## StatefulSet

StatefulSet is the workload API object used to manage stateful applications, that:
* provide guarantees about ordering and uniqueness of pods
* provide unique network identifiers for pods
* have persistent storage
* have graceful deployment and scaling
* provide ordered, automated rolling updates

Components: Service, StatefulSet, Persistent Volume, Persistent Volume Claim, Pods

`sts` - short for StatefulSet

### Create StatefulSet

Let's create a `nginx-stateful-set.yaml` file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nginx-svc # should match .spec.serviceName
  labels:
    app: web-nginx
spec:
  ports:
  - port: 80
    name: web-service
  clusterIP: None
  selector:
    app: web-nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web-nginx-stateful
spec:
  selector:
    matchLabels:
      app: web-nginx # should match .spec.template.metadata.labels
  serviceName: web-nginx-svc # should match .metadata.name
  replicas: 3
  template:
    metadata:
      labels:
        app: web-nginx # should match .spec.selector.matchLabels.app
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: web-nginx-pod
        image: nginx:1.20-alpine
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: web-pvc
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: web-pvc
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
* A Headless Service, named `web-nginx-svc`, is used to control the network domain.
* The StatefulSet, named web, has a Spec that indicates that 3 replicas of the nginx container will be launched in unique Pods.
* The volumeClaimTemplates, named `web-pvc`, will provide stable storage using PersistentVolumes provisioned by a PersistentVolume Provisioner.

```shell
kubectl apply -f nginx-stateful-set.yaml
```

Watch pods created one by one
```shell
kubectl get po -w -l app=web-nginx
```

### Network Identities of Stateful Set

Hostname consists of
```shell
${StatefulSet.metadata.name}-${order}.${Service.metadata.name}.${namespace}
```
* ${StatefulSet.metadata.name} - `web-nginx-stateful`
* ${order} - number 0..N-1 replicas count
* ${Service.metadata.name} - `web-nginx-svc`
* ${namespace} - namespace stateful created in

for example full:
```shell
web-nginx-stateful-1.web-nginx-svc.test-sandbox.svc.cluster.local
```
and short:
```shell
web-nginx-stateful-1.web-nginx-svc
```


Get hostname of each pod
```shell
for i in $(kubectl get po -o name -l app=web-nginx);
do
   kubectl exec "$i" -- sh -c 'hostname -f';
done
```
Will output
```
web-nginx-stateful-0.web-nginx-svc.test-sandbox.svc.cluster.local
web-nginx-stateful-1.web-nginx-svc.test-sandbox.svc.cluster.local
web-nginx-stateful-2.web-nginx-svc.test-sandbox.svc.cluster.local
```

Create an empty container 
```shell
kubectl run test-pod --image=alpine --rm -it \ 
          --restart=Never -- sh
```

```shell
nslookup web-nginx-svc

    Server:		10.96.0.10
    Address:	10.96.0.10:53
    
    Name:	web-nginx-svc.test-sandbox.svc.cluster.local
    Address: 172.17.0.11
    Name:	web-nginx-svc.test-sandbox.svc.cluster.local
    Address: 172.17.0.13
    Name:	web-nginx-svc.test-sandbox.svc.cluster.local
    Address: 172.17.0.12

nslookup 172.17.0.11

  Server:		10.96.0.10
  Address:	10.96.0.10:53

  11.0.17.172.in-addr.arpa	name = web-nginx-stateful-0.web-nginx-svc.test-sandbox.svc.cluster.local

```

# ConfigMaps

A ConfigMap is an API object used to store non-confidential data in key-value pairs.

cm - is short for config maps

## Get ConfigMap

```shell
# get configmaps from current namespace
kubectl get configmap
# get configmaps from all namespaces
kubectl get configmap -A
# get configmaps from test-sandbox namespaces
kubectl get configmap -n test-sandbox
# get configmap postgres-config contents in yaml
kubectl get configmap postgres-config -o yaml 
```

## Create ConfigMap

Create ConfigMap with command line
```shell
kubectl create cm test-config-map \
    --from-literal KEY1=VALUE1 \
    --from-literal KEY2=VALUE2
```

Create ConfigMap from environment file `test.env` containing key=value pairs
```shell
KEY1=VALUE1
KEY2=VALUE2
KEY3=LONG VALUE WITH SPACES
```
```shell
kubectl create cm test-config-map \
    --from-env-file=test.env
```

Create ConfigMap from defined configuration file `test-config-map.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-config-map
data:
  KEY1: VALUE1
  KEY2: VALUE2
```

Apply ConfigMap configuration 
```shell
kubectl apply -f test-config-map.yaml
```

Create ConfigMap from file `public-cert.pem`
All file data will be value of PUBLIC_CERT variable
```shell
kubectl create cm test-config-map \
    --from-file PUBLIC_CERT=public-cert.pem
```

## Edit ConfigMaps

Edit ConfigMap named test-config-map from current namespace
```shell
kubectl edit configmap test-config-map
```
Current pods using this ConfigMap will not be affected

## Delete ConfigMaps

Delete ConfigMap named test-config-map from current namespace

```shell
kubectl delete configmap test-config-map
```

Current pods using this ConfigMap will not be affected


## Using ConfigMaps

### Environment variables from ConfigMap with envFrom

Let's create a test pod `config-test-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-test-pod
spec:
  containers:
    - name: test-container
      image: nginx:1.20-alpine
      envFrom:
      - configMapRef:
          name: test-config-map
  restartPolicy: Never
```
Run pod
```shell
kubectl create -f config-test-pod.yaml
```
Check the environment variables
```shell
kubectl exec -it config-test-pod  -- env | grep KEY
```
Should output
```
KEY1=VALUE1
KEY2=VALUE2
```

### Single environment variable from ConfigMap with valueFrom

Let's create a test pod `config-test-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-test-pod
spec:
  containers:
    - name: test-container
      image: nginx:1.20-alpine
      env:
        - name: KEY1
          valueFrom:
            configMapKeyRef:
              name: test-config-map
              key: KEY1
  restartPolicy: Never
```
```shell
kubectl create -f config-test-pod.yaml
```
Check the environment variable KEY
```shell
kubectl exec -it config-test-pod  -- env | grep KEY
```
Should output
```
KEY1=VALUE1
```

### Mount ConfigMap as a volume directory

Let's create a test pod `config-test-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-test-pod
spec:
  containers:
    - name: test-container
      image: nginx:1.20-alpine
      volumeMounts:
        - name: config-volume
          mountPath: /volume
  volumes:
    - name: config-volume
      configMap:
        name: test-config-map
  restartPolicy: Never
```
```shell
kubectl create -f config-test-pod.yaml
```

Check files
```shell
kubectl exec -it config-test-pod  -- sh

ls /volume/
  KEY1  KEY2
  
cat /volume/KEY1
  VALUE1

cat /volume/KEY2
  VALUE2
```

### Mount single file from ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-test-pod
spec:
  containers:
    - name: test-container
      image: nginx:1.20-alpine
      volumeMounts:
        - name: config-volume
          mountPath: /volume/KEY1
          subPath: KEY1
  volumes:
    - name: config-volume
      configMap:
        name: test-config-map
  restartPolicy: Never
```
```shell
kubectl create -f config-test-pod.yaml
```
Check files
```shell
kubectl exec -it config-test-pod  -- sh

ls /volume/
  KEY1
  
cat /volume/KEY1
  VALUE1
```

# Labels

## Get resource labels

User `--show-labels key` to show labels
```shell
kubectl get po --show-labels
```
Output
```
NAME            READY   STATUS    RESTARTS   AGE     LABELS
pg-etcd-0       1/1     Running   1          3m53s   app=pg-etcd,controller-revision-hash=pg-etcd-78dbf7b6d5,statefulset.kubernetes.io/pod-name=pg-etcd-0
pg-etcd-1       1/1     Running   0          3m30s   app=pg-etcd,controller-revision-hash=pg-etcd-78dbf7b6d5,statefulset.kubernetes.io/pod-name=pg-etcd-1
pg-etcd-2       1/1     Running   3          15h     app=pg-etcd,controller-revision-hash=pg-etcd-78dbf7b6d5,statefulset.kubernetes.io/pod-name=pg-etcd-2
pg-postgres-0   1/1     Running   2          15h     app=pg-postgres,controller-revision-hash=pg-postgres-674879f847,role=master,statefulset.kubernetes.io/pod-name=pg-postgres-0
pg-postgres-1   1/1     Running   2          15h     app=pg-postgres,controller-revision-hash=pg-postgres-674879f847,statefulset.kubernetes.io/pod-name=pg-postgres-1
pg-postgres-2   1/1     Running   2          15h     app=pg-postgres,controller-revision-hash=pg-postgres-674879f847,statefulset.kubernetes.io/pod-name=pg-postgres-2
terminal-pod    1/1     Running   3          41h     run=terminal-pod
```

Get labels for other type of resources
```shell
# services
kubectl get svc --show-labels
# persistent volume claim
kubectl get pvc --show-labels 
```

## Use labels for filtering resources

Use key `-l` or `--selector` for filtering only matching labels
```shell
kubectl get po -l app=pg-postgres
```
For multiple conditions use comma separated format
```shell
kubectl get po -l app=pg-postgres,role=master 
```
### Filtering condition operations

* equality =
* inequality !=
* inclusion (in)
* exclusion (notin)

```shell

# equality label app is equal to pg-postgres
# and label role is equal to master
kubectl get po -l role=master 

# inequality
kubectl get po -l role!=slave
 
# inclusion (user single quotes)
# get all resources which
# key app includes values both pg-postgres or pg-etcd
kubectl get po -l 'app in (pg-postgres,pg-etcd)'

# exclusion
# get all resources which 
# key env value is not equal to both dev or test 
kubectl get po -l 'env notin (dev, test)'

# multiple conditions
kubectl get po -l 'app in (pg-postgres,pg-etcd),role!=slave,env notin (dev, test)'
```

## Add labels to pod

Put label `role` equals `master` to pod `pg-postgres-2`
```shell
kubectl label po pg-postgres-2 role=master
```
Pod `pg-postgres-2` should not have this label before, or you will get error
```
error: 'role' already has a value (master), and --overwrite is false
```
Which means to overwrite value of label `role` use `--overwrite` key

## Remove labels from pod

Remove label `unhealhy` from `pg-postgres-0` pod (use minus sing)
```shell
kubectl label po pg-postgres-0 unhealhy-
```

## Using labels as a resource selectors

Labels is used as a selectors for resources types:
* Service
* Deployment
* StatefulSet
* ReplicationController
* Job 
* ReplicaSet
* DaemonSet

### Matching condition types

`matchLabels` conditions is used for selectors as key=values pair with AND operation
```yaml
selector:
  matchLabels:
    app: pg-postgres
    healthy: true
    role: master
```

`matchExpressions` is usr for selectors with operations `In`, `NotIn`, `Exists` and `DoesNotExist`
````yaml
selector:
  matchLabels:
    app: pg-postgres
    healthy: true
    role: master
  matchExpressions:
    - {key: env, operator: In, values: [production]}
    - {key: env, operator: NotIn, values: [dev,test]}
````

# Service Accounts

Service Account (sa) - is a special type of account intended to represent a non-human user that needs to authenticate and be authorized to access kubernetes APIs.

## Managing ServiceAccounts

### Get Service Accounts
```shell
kubectl get sa
```

### Create Service Accounts 

Create a service account named `terminal-sa`
```shell
kubectl create sa terminal-sa
```

## Service Account Usage

Create a Service account from previous example

Create pod which uses service account.
`spec.serviceAccountName` and `spec.automountServiceAccountToken`
```shell
apiVersion: v1
kind: Pod
metadata:
  name: terminal-pod
spec:
  serviceAccountName: terminal-sa
  automountServiceAccountToken: true
  containers:
  - image: nginx:alpine
    imagePullPolicy: IfNotPresent
    name: terminal-pod
```

Service accounts secrets will be mounted in `ls -la /var/run/secrets/kubernetes.io/serviceaccount/`

```shell
kubectl exec -ti terminal-pod -- sh

# list contents of 

ls -la /var/run/secrets/kubernetes.io/serviceaccount/

total 4
drwxrwxrwt    3 root     root           140 Jul 30 16:43 .
drwxr-xr-x    3 root     root          4096 Jul 30 16:43 ..
drwxr-xr-x    2 root     root           100 Jul 30 16:43 ..2021_07_30_16_43_14.670754610
lrwxrwxrwx    1 root     root            31 Jul 30 16:43 ..data -> ..2021_07_30_16_43_14.670754610
lrwxrwxrwx    1 root     root            13 Jul 30 16:43 ca.crt -> ..data/ca.crt
lrwxrwxrwx    1 root     root            16 Jul 30 16:43 namespace -> ..data/namespace
lrwxrwxrwx    1 root     root            12 Jul 30 16:43 token -> ..data/token

```

* `ca.crt` - CA certificates bundle
* `namespace` - namespace name
* `token` - api auth token 

#### Role and RoleBinding

An RBAC Role contains rules that represent a set of permissions. Permissions are purely additive (there are no "deny" rules).

A Role always sets permissions within a particular namespace; when you create a Role, you have to specify the namespace it belongs in.

Creating a role which forbids everything except get pod named `terminal-pod`
```shell
kubectl create role terminal-role \
  --verb=get --resource=pods \
  --resource-name=terminal-pod
```
or
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: terminal-role
rules:
- apiGroups: [""]
  resourceNames: ["terminal-pod"]
  resources: ["pods"]
  verbs: ["get"]
```

Where verb is a permissions set:
* get - read a resource
* list - resources
* watch - watch resources
* patch - could be patched runtime
* update - could be updated
* create - could be created new resources of the type
* list - can get a list of resources
* delete - can delete resource of that type

Bind this Role to Service Account creating RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: terminal-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: terminal-role
subjects:
- kind: ServiceAccount
  name: terminal-sa
```

## Kubernetes APIs

API default path:
```
https://kubernetes.default.svc.cluster.local
```
or get API IP from environment variables
```dotenv
export KUBERNETES_PORT='tcp://10.96.0.1:443'
export KUBERNETES_PORT_443_TCP='tcp://10.96.0.1:443'
export KUBERNETES_PORT_443_TCP_ADDR='10.96.0.1'
export KUBERNETES_PORT_443_TCP_PORT='443'
export KUBERNETES_PORT_443_TCP_PROTO='tcp'
export KUBERNETES_SERVICE_HOST='10.96.0.1'
export KUBERNETES_SERVICE_PORT='443'
export KUBERNETES_SERVICE_PORT_HTTPS='443'
```

Export CA certificate bundle, token and namespace
```shell
export NAMESPACE=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)
export CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
export TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```

Get pod information
```shell
curl --cacert ${CA_BUNDLE} \
  -H "Authorization: Bearer ${TOKEN}" \
  https://kubernetes.default.svc.cluster.local/api/v1/namespaces/${NAMESPACE}/pods/terminal-pod
```

# Scaling

## Manual scaling

```shell
# scale deployment nginx-deployment to 3 replicas
kubectl scale --replicas=3 deployment/nginx-deployment

# conditionally scale deployment nginx-deployment
# if currently we have 3 replicas than scale to 5
kubectl scale --current-replicas=3 --replicas=5 \
        deployment/nginx-deployment
```

# Network

## Services

Service (svc) - a way to expose an application running on a set of Pods as a network service.

### Get services

```shell
# get all services in all namespaces
kubectl get svc --all-namespaces
# get all services in current namespace
kubectl get svc
# get all services in namespace test-sandbox
kubectl get svc -n test-sandbox
# get all services by selector app=nginx
kubectl get svc -l app=nginx
# get service by name
kubectl get svc nginx-deployment
```

### Create ClusterIP

Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType

```shell
kubectl expose deploy nginx-deployment -l app=nginx \
  --name=nginx-deployment-svc-clusterip \
  --type=ClusterIP \
  --port=8080 --target-port=80
# or
kubectl create svc clusterip nginx --tcp=8080:80
```

Create `nginx-deployment-svc-clusterip.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    # should match with labels of deployment and pods
    app: nginx
  name: nginx-deployment-svc-clusterip
spec:
  selector:
    # should match with labels of deployment and pods
    app: nginx
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  type: ClusterIP
```
Apply
```shell
kubectl apply -f nginx-deployment-svc-clusterip.yaml
```

Get IP address
```shell
kubectl describe svc nginx-deployment-svc-clusterip | grep IP:
  
  IP:                10.108.153.142
```

Check
```shell
kubectl run -it --rm --image=alpine -- sh 
apk add curl
curl http://10.108.153.142:8080/
```

### Create nodePort

Exposes the Service on each Node's IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You'll be able to contact the NodePort Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`

```shell
kubectl expose deploy nginx-deployment \
        --port=80 --type=NodePort
```

### Create LoadBalancer

Exposes the Service externally using a cloud provider's load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created

```shell
kubectl expose deployment nginx-deployment \
  --type=LoadBalancer --port=8080 --target-port=80
```

* target-port - port of container
* port - external port


# Resources

- [Medium.com](https://medium.com)
    * [Kubernetes in three diagrams](https://tsuyoshiushio.medium.com/kubernetes-in-three-diagrams-6aba8432541c)
    * [Deploying PostgreSQL as a StatefulSet in Kubernetes](https://www.bmc.com/blogs/kubernetes-postgresql/)
- [Kubernetes.io](https://kubernetes.io/)
    * [Kubernetes concepts](https://kubernetes.io/docs/concepts/_print/)
    * [kubectl cli Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
    * [Managing resources](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/)
    * [Workloads](https://kubernetes.io/docs/concepts/workloads/)
    * [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
    * [Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)