# kubectl

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
    + [Examples of Stateful Set](#examples-of-stateful-set)
- [Control commands](#control-commands)
  * [Apply](#apply)
  * [Delete](#delete)
  * [Edit](#edit)
  * [Patch](#patch)
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
- [Service Accounts](#service-accounts)
  * [Managing ServiceAccounts](#managing-serviceaccounts)
    + [Get Service Accounts](#get-service-accounts)
    + [Create Service Accounts](#create-service-accounts)
  * [Service Account Usage](#service-account-usage)
      - [Role and RoleBinding](#role-and-rolebinding)
  * [Kubernetes APIs](#kubernetes-apis)
    + [Patch resource using API](#patch-resource-using-api)
- [Nodes](#nodes)
  * [Cordon and uncordon node](#cordon-and-uncordon-node)
  * [Safely Drain a Node](#safely-drain-a-node)
  * [Pod Affinity and anti-affinity](#pod-affinity-and-anti-affinity)
- [Deploying](#deploying)
  * [Rollout](#rollout)
    + [Rollout restart](#rollout-restart)
    + [Partial deployment](#partial-deployment)
    + [Rollback to previous revision](#rollback-to-previous-revision)
  * [Scaling](#scaling)
    + [Manual scaling](#manual-scaling)
    + [Autoscale](#autoscale)
      - [Units](#units)
      - [Horizontal pod autoscaler](#horizontal-pod-autoscaler)
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

### Examples of Stateful Set
 
1. [etcd cluster](https://github.com/iliadmitriev/etcd-cluster)
2. [postgresql](https://github.com/iliadmitriev/postgres-cluster)

# Control commands

## Apply

Applies configuration to resources or create new resources.

from file
```shell
kubectl apply -f nginx-deployment.yaml
# or 
cat nginx-deployment.yaml | kubectl apply -f-
```
or from stdin

```shell
kubectl apply -f - << EOF
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
EOF
```

## Delete

Delete resources

```shell
# delete resources specified in yaml file
kubectl delete -f nginx-deployment.yaml
# delete pod 
kubectl delete po nginx-web-1
# delete deployment with all it's pods
kubectl delete deployment nginx-web
# delete pods and services by label selector
kubectl delete pods,services -l name=nginx
```

## Edit

Edit a resource by the default editor.
Editor can be changed via `KUBE_EDITOR` environment variable.

```shell
# edit service web-nginx
kubectl edit svc web-nginx
```

## Patch

Update field(s) of a resource using strategic merge patch, a JSON merge patch, or a JSON patch.

```shell
# patch pod named terminal, set new image alpine to containers named "terminal"
kubectl patch pod terminal \
  -p '{"spec":{"containers":[{"name":"terminal","image":"alpine"}]}}'
# patch pod named terminal, using json patch
kubectl patch pod terminal --type='json' \
 -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"alpine"}]'

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

# Secrets

A Secret is a namespaced resource that contains a small amount of secretive data such as a password, a token, or a key, which you don't want to include in your application code.

A secret can be used with pods in three ways:
* As file(s) with volume mounted to pod
* As container environment variable(s)

## Types of Secret

* Opaque (generic, default) - arbitrary user-defined data
* docker-registry (kubernetes.io/dockerconfigjson) serialized ~/.docker/config.json file
* tls (kubernetes.io/tls) - data for a TLS client or server, x509 certificate and a private key
* service account token (kubernetes.io/service-account-token) service account token data to authorize at kubernetes api
* kubernetes.io/basic-auth - credentials for basic authentication (http)
* kubernetes.io/ssh-auth - credentials for ssh authentication (ssh, git)

## Opaque secrets

Opaque is the default Secret type if omitted. Creation of opaque secret:
```shell
kubectl create secret generic mypasswords --from-literal=key1=supersecret --from-literal=key2=topsecret
```
or with yaml file
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mypasswords
data:
  key1: c3VwZXJzZWNyZXQ=
  key2: dG9wc2VjcmV0
```
key1 and key2 values go as base64 encoded strings

## Docker registry auth data

```shell
kubectl create secret docker-registry dockerhub \
  --docker-server=hub.docker.com \
  --docker-username=dockerusername \
  --docker-password=secret \
  --docker-email=myemail@gmail.com
```
or 
```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/dockerconfigjson
metadata:
  name: dockerhub
data:
  .dockerconfigjson: eyJhdXRocyI6eyJodWIuZG9ja2VyLmNvbSI6eyJ1c2VybmFtZSI6ImRvY2tlcnVzZXJuYW1lIiwicGFzc3dvcmQiOiJzZWNyZXQiLCJlbWFpbCI6Im15ZW1haWxAZ21haWwuY29tIiwiYXV0aCI6IlpHOWphMlZ5ZFhObGNtNWhiV1U2YzJWamNtVjAifX19
```
where `.data.dockerconfigjson` is base64 encoded json with docker login data
```json
{
  "auths": {
    "hub.docker.com": {
      "username": "dockerusername",
      "password": "secret",
      "email": "myemail@gmail.com",
      "auth": "ZG9ja2VydXNlcm5hbWU6c2VjcmV0"
    }
  }
}
```

## TLS secret

TLS Secret contains `tls.key` and `tls.crt` keys, a pair of x509 certificate and a private key stored in pem format.

When creating tls secret with kubectl, it checks provided data on:
* private key structure
* x509 certificate structure
* key and certificate match each other

```shell
kubectl create secret tls my-cert \
  --cert git/openssl-scripts/localhost.pem \
  --key git/openssl-scripts/localhost.key
```

## Using secrets

### As a mount from Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: nginx:alpine
    volumeMounts:
    - name: foo
      mountPath: "/mypasswords"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mypasswords
```

Check 
```shell
kubectl exec -ti mypod -- sh

ls -l /mypasswords/
# total 0
# lrwxrwxrwx    1 root     root            11 Aug  4 11:52 key1 -> ..data/key1
# lrwxrwxrwx    1 root     root            11 Aug  4 11:52 key2 -> ..data/key2

cat /mypasswords/key1
# supersecret

cat /mypasswords/key2
# topsecret
```

### As a mount from Pod to a specific key path

Set mode 0600 default
and 0610 to a file mykey1
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: nginx:alpine
    volumeMounts:
    - name: foo
      mountPath: "/mypasswords"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mypasswords
      defaultMode: 0600
      items:
        - key: key1
          path: mykey1
          mode: 0610
```
Check
```shell
kubectl exec -ti mypod -- sh

ls -l /mypasswords/
# total 0
# lrwxrwxrwx    1 root     root            13 Aug  4 11:57 mykey1 -> ..data/mykey1

cat /mypasswords/mykey1 
# supersecret

ls -l /mypasswords/..data/mykey1 
# -rw---x---    1 root     root            11 Aug  4 12:04 /mypasswords/..data/mykey1
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

Service accounts secrets will be mounted to pod container in directory

`/var/run/secrets/kubernetes.io/serviceaccount/`

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

Execute shell in [terminal-pod created earlier](#service-account-usage)

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

### Patch resource using API

Let's update our Role created in [previous example](#role-and-rolebinding)
And add permission to `patch` resource named `terminal-pod`

```shell
kubectl edit roles terminal-role
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: terminal-role
  namespace: test-sandbox
rules:
- apiGroups:
  - ""
  resourceNames:
  - terminal-pod
  resources:
  - pods
  verbs:
  - get
  - patch
```
Add patch verb to the list

Get back to our pod `terminal-pod` terminal sh
And add labels `main` with value `yes` to our pod using API from inside the container
```shell
curl --cacert ${CA_BUNDLE} \
  -H "Authorization: Bearer ${TOKEN}" \
  https://kubernetes.default.svc.cluster.local/api/v1/namespaces/${NAMESPACE}/pods/terminal-pod \
   -XPATCH -H 'Content-type: application/merge-patch+json' \
   -d '{"metadata":{"labels":{"main":"yes"}}}'
```

Lets check labels 
```shell
kubectl get po terminal-pod --show-labels

NAME           READY   STATUS    RESTARTS   AGE   LABELS
terminal-pod   1/1     Running   0          51m   main=yes
```

# Nodes

Get node list
```shell
kubectl get nodes
kubectl get nodes -o wide
```

Get node info and statistics
```shell
kubectl describe nodes minikube
```

## Cordon and uncordon node

Cordon - mark node as unschedulable. New pods will not be assigned to this node.
```shell
kubectl cordon minikube
```
Check if node is unschedulable
```shell
kubectl describe nodes minikube
```
Output
```
....
Unschedulable:      true
...
```

Make node schedulable again
```shell
kubectl uncordon minikube
```

## Safely Drain a Node

```shell
kubectl drain <node name>
```

## Pod Affinity and anti-affinity

Pod Affinity and Anti-affinity is a language on how pod attracted to node or distracted from node.
There is two types of affinity and anti affinity:
* `preferredDuringSchedulingIgnoredDuringExecution` is a set of soft rules
* `requiredDuringSchedulingIgnoredDuringExecution` is hard requirements

Let's assume we have 3 worker nodes in our kubernetes cluster (for example we will use [minikube setup](minikube.md))

```shell
kubectl get nodes

# NAME           STATUS   ROLES                  AGE     VERSION
# minikube       Ready    control-plane,master   9m56s   v1.21.2
# minikube-m02   Ready    <none>                 9m16s   v1.21.2
# minikube-m03   Ready    <none>                 8m41s   v1.21.2
```

Create deployment
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
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - topologyKey: "kubernetes.io/hostname"
              labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - nginx
      containers:
      - name: nginx
        image: nginx:1.20-alpine
        ports:
        - containerPort: 80
```
This deployment means that during scheduling pods can't be on the same node (the node that has pods matching with selector `app In [nginx]`)

Make sure the nodes all the pods running and they all running on different nodes
```shell
kubectl get po -o wide
# NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
# nginx-deployment-5dfdbd8546-5n2dg   1/1     Running   0          14s   10.244.2.2   minikube-m03   <none>           <none>
# nginx-deployment-5dfdbd8546-rv8mg   1/1     Running   0          14s   10.244.0.3   minikube       <none>           <none>
# nginx-deployment-5dfdbd8546-whks7   1/1     Running   0          14s   10.244.1.2   minikube-m02   <none>           <none>
```

Let's scale pods to 4 replicas
```shell
kubectl scale deployment nginx-deployment --replicas 4
# deployment.apps/nginx-deployment scaled
```

And check pods 
```shell
kubectl get po -o wide                                
# NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
# nginx-deployment-5dfdbd8546-5n2dg   1/1     Running   0          36s   10.244.2.2   minikube-m03   <none>           <none>
# nginx-deployment-5dfdbd8546-lmht4   0/1     Pending   0          2s    <none>       <none>         <none>           <none>
# nginx-deployment-5dfdbd8546-rv8mg   1/1     Running   0          36s   10.244.0.3   minikube       <none>           <none>
# nginx-deployment-5dfdbd8546-whks7   1/1     Running   0          36s   10.244.1.2   minikube-m02   <none>           <none>
```
4th pod has been created but is not running and have status `Pending`
because of "hard" rules of anti-affinity.
There is no node which hasn't pods with label app=nginx

Let's create another worker node and add it to kubernetes cluster
```shell
minikube node add 
# ...
# 🔎  Verifying Kubernetes components...
# 🏄  Successfully added m04 to minikube!

kubectl get node 
# NAME           STATUS   ROLES                  AGE   VERSION
# minikube       Ready    control-plane,master   24m   v1.21.2
# minikube-m02   Ready    <none>                 23m   v1.21.2
# minikube-m03   Ready    <none>                 22m   v1.21.2
# minikube-m04   Ready    <none>                 18s   v1.21.2
```

And check 4th pod status
```shell
kubectl get po -o wide
# NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
# nginx-deployment-5dfdbd8546-5n2dg   1/1     Running   0          28m   10.244.2.2   minikube-m03   <none>           <none>
# nginx-deployment-5dfdbd8546-lmht4   1/1     Running   0          28m   10.244.3.2   minikube-m04   <none>           <none>
# nginx-deployment-5dfdbd8546-rv8mg   1/1     Running   0          28m   10.244.0.3   minikube       <none>           <none>
# nginx-deployment-5dfdbd8546-whks7   1/1     Running   0          28m   10.244.1.2   minikube-m02   <none>           <none>

kubectl describe po nginx-deployment-5dfdbd8546-lmht4

# ...
# Events:
#   Type     Reason            Age                 From               Message
#   ----     ------            ----                ----               -------
#   Warning  FailedScheduling  11m (x20 over 30m)  default-scheduler  0/3 nodes are available: 3 node(s) didn't match pod affinity/anti-affinity rules, 3 node(s) didn't match pod anti-affinity rules.
#   Warning  FailedScheduling  11m                 default-scheduler  0/4 nodes are available: 1 node(s) had taint {node.kubernetes.io/not-ready: }, that the pod didn't tolerate, 3 node(s) didn't match pod affinity/anti-affinity rules, 3 node(s) didn't match pod anti-affinity rules.
#   Normal   Scheduled         11m                 default-scheduler  Successfully assigned default/nginx-deployment-5dfdbd8546-lmht4 to minikube-m04
#   Normal   Pulling           11m                 kubelet            Pulling image "nginx:1.20-alpine"
#   Normal   Pulled            10m                 kubelet            Successfully pulled image "nginx:1.20-alpine" in 3.850949002s
#   Normal   Created           10m                 kubelet            Created container nginx
#   Normal   Started           10m                 kubelet            Started container nginx

```

So now as we added a new node to cluster, which wasn't have pods with label app=nginx, kubernetes scheduler successfully deployed pod and changed status to `Running`


# Deploying

## Rollout

Rollout can be applied to:
* [deployments](#deployments)
* [stateful sets](#statefulset)
* daemon stets

Get status of current deployment
```shell
kubectl rollout status statefulset web-nginx-stateful
```
```
partitioned roll out complete: 1 new pods have been updated...
```

Get history of previous 3 deployments
```shell
kubectl rollout history statefulset web-nginx-stateful
```
```
statefulset.apps/web-nginx-stateful 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

### Rollout restart

```shell
kubectl rollout restart statefulset web-nginx-stateful 
```

### Partial deployment

Partition size designates an ordinal greater than or equal to that will be updated.

In other words if you have 3 pods numbered from 0 to 2, and partition is set to 3, this means no pods will be updated.
To have all pods updated you have to set partition to 0.
```shell
kubectl get statefulsets.apps web-nginx-stateful \
  -o json | jq '.spec.updateStrategy'
```
```json
{
  "spec": {
    "updateStrategy": {
      "type": "RollingUpdate",
      "rollingUpdate": {
        "partition": 0
      }
    }
  }
}
```

```shell
kubectl describe statefulsets.apps web-nginx-stateful
```
```
Update Strategy:    RollingUpdate
  Partition:        0
```

1) Set partition size to 3
```shell
kubectl patch statefulset web-nginx-stateful \
  -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":3}}}}'
```

2) Patch pod template with a new image of nginx
```shell
kubectl patch statefulset web-nginx-stateful \
  --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"nginx:1.21-alpine"}]'
```

Let's check that rolling update does not start
```shell
kubectl rollout status statefulset web-nginx-stateful
# partitioned roll out complete: 0 new pods have been updated...
```

```shell
kubectl get pods -l app=web-nginx \
  -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}'
  
# web-nginx-stateful-0:	nginx:1.20-alpine, 
# web-nginx-stateful-1:	nginx:1.20-alpine, 
# web-nginx-stateful-2:	nginx:1.20-alpine,
```

3) Set partition size to 2
```shell
kubectl patch statefulset web-nginx-stateful \
 -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":2}}}}'
```
Let's check that rolling update goes only on one last pod
```shell
kubectl rollout status statefulset web-nginx-stateful
# partitioned roll out complete: 1 new pods have been updated...
```
Check updated pod
```shell
kubectl get pods -l app=web-nginx \
  -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}'

# web-nginx-stateful-0:	nginx:1.20-alpine, 
# web-nginx-stateful-1:	nginx:1.20-alpine, 
# web-nginx-stateful-2:	nginx:1.21-alpine,                                                                                                                                                       
```

4) Set partition size to 0 (finish deploy operation)
```shell
kubectl patch statefulset web-nginx-stateful \
  -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":0}}}}'
```
Check rollout status
```shell
kubectl rollout status statefulset web-nginx-stateful
# Waiting for 1 pods to be ready...
# Waiting for 1 pods to be ready...
# partitioned roll out complete: 3 new pods have been updated...
```
Check container images:
```shell
kubectl get pods -l app=web-nginx \
  -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}'

# web-nginx-stateful-0:	nginx:1.21-alpine, 
# web-nginx-stateful-1:	nginx:1.21-alpine, 
# web-nginx-stateful-2:	nginx:1.21-alpine,
```

### Rollback to previous revision

Get history of deployments revision
```shell
kubectl rollout history statefulset web-nginx-stateful
```
```
statefulset.apps/web-nginx-stateful 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

Rollback to revision 2
```shell
kubectl rollout undo statefulset web-nginx-stateful --to-revision 2
```

Rollback operation creates another rollout in revision history

## Scaling

### Manual scaling

```shell
# scale deployment nginx-deployment to 3 replicas
kubectl scale --replicas=3 deployment/nginx-deployment

# conditionally scale deployment nginx-deployment
# if currently we have 3 replicas than scale to 5
kubectl scale --current-replicas=3 --replicas=5 \
        deployment/nginx-deployment
```

### Autoscale

Before you begin

[Metrics server](https://github.com/kubernetes-sigs/metrics-server) monitoring needs to be deployed in the cluster to provide metrics through the Metrics API. Addon [metrics-server](minikube.md#add-nodes-to-minikube-cluster) should be enabled for [minikube](minikube.md) setup

Check if metrics server properly works
```shell
# check metrics from nodes
kubectl top nodes

# check metrics from pods
kubectl top pod <podName>
```

Let's create a pod requiring CPU resources, performing vast amount of calculations.

As an example we can use [python script](https://github.com/iliadmitriev/hw/blob/master/workload.py) which calculates a sum of square root between 1 and 1.000.000

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-sqrt-svc
  labels:
    app: sqrt
spec:
  selector:
    app: sqrt
  ports:
  - port: 8080
    name: http
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-sqrt
  labels:
    app: sqrt
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sqrt
  template:
    metadata:
      labels:
        app: sqrt
    spec:
      containers:
      - name: sqrt-pod
        image: iliadmitriev/workload:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        # it's mandatory to specify
        # resources limits
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 250m
```

#### Units

CPU
* 1 - means vCPU unit
* 500m (milli cpu) - 0,5 of vCPU unit

Memory
* suffixes: E, P, T, G, M, k
* suffixes: Ei, Pi, Ti, Gi, Mi, Ki
* 128Mi - 128 megabytes

#### Horizontal pod autoscaler

Create autoscaler for deployment `web-sqrt`
```shell
kubectl autoscale deployment web-sqrt \
  --max 10 --min 2 --cpu-percent=50
```

Get autoscaler state
```shell
kubectl describe hpa web-sqrt
```
Output
```
Name:                                                  web-sqrt
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Mon, 02 Aug 2021 22:03:25 +0300
Reference:                                             Deployment/web-sqrt
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  0% (1m) / 50%
Min replicas:                                          2
Max replicas:                                          10
Deployment pods:                                       2 current / 2 desired
Conditions:
  Type            Status  Reason               Message
  ----            ------  ------               -------
  AbleToScale     True    ScaleDownStabilized  recent recommendations were higher than current one, applying the highest recent recommendation
  ScalingActive   True    ValidMetricFound     the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  False   DesiredWithinRange   the desired count is within the acceptable range
Events:           <none>
```
Get current workloads 
```shell
kubectl get pod -l app=sqrt -o wide -w
```
Output
```
NAME                       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
web-sqrt-9949c44f7-nhdfv   1/1     Running   0          2m36s   10.244.2.2   minikube-m03   <none>           <none>
web-sqrt-9949c44f7-p259n   1/1     Running   0          2m36s   10.244.1.2   minikube-m02   <none>           <none>
```

Simulate huge load
```shell
# create a new empty container
kubectl run terminal -ti --image=alpine --command -- sh

# install apache benchmark
apk add apache2-utils

# make 10.000 requests in 40 threads to service
ab -c 40 -n 10000 http://web-sqrt-svc:8080/
```
New pods are created
```
NAME                       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
web-sqrt-9949c44f7-26ngb   1/1     Running   0          2m15s   10.244.1.6   minikube-m02   <none>           <none>
web-sqrt-9949c44f7-96lbc   1/1     Running   0          2m15s   10.244.2.6   minikube-m03   <none>           <none>
web-sqrt-9949c44f7-cklmf   1/1     Running   0          2m30s   10.244.2.4   minikube-m03   <none>           <none>
web-sqrt-9949c44f7-g7m7t   1/1     Running   0          2m30s   10.244.1.4   minikube-m02   <none>           <none>
web-sqrt-9949c44f7-gvchr   1/1     Running   0          2m      10.244.2.7   minikube-m03   <none>           <none>
web-sqrt-9949c44f7-j4jbp   1/1     Running   0          2m15s   10.244.1.5   minikube-m02   <none>           <none>
web-sqrt-9949c44f7-nhdfv   1/1     Running   0          8m20s   10.244.2.2   minikube-m03   <none>           <none>
web-sqrt-9949c44f7-nx7nf   1/1     Running   0          2m      10.244.0.3   minikube       <none>           <none>
web-sqrt-9949c44f7-p259n   1/1     Running   0          8m20s   10.244.1.2   minikube-m02   <none>           <none>
web-sqrt-9949c44f7-zss2v   1/1     Running   0          2m15s   10.244.2.5   minikube-m03   <none>           <none>
```

Cancel `ab` requests. After a while horizontal pod autoscaler will scale down and all extra pods will be stopped and deleted

```
Name:                                                  web-sqrt
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Mon, 02 Aug 2021 21:13:57 +0300
Reference:                                             Deployment/web-sqrt
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  0% (1m) / 50%
Min replicas:                                          2
Max replicas:                                          10
Deployment pods:                                       2 current / 2 desired
Conditions:
  Type            Status  Reason            Message
  ----            ------  ------            -------
  AbleToScale     True    ReadyForNewScale  recommended size matches current size
  ScalingActive   True    ValidMetricFound  the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  True    TooFewReplicas    the desired replica count is less than the minimum replica count
Events:
  Type     Reason                        Age                  From                       Message
  ----     ------                        ----                 ----                       -------
  Normal   SuccessfulRescale             18m                  horizontal-pod-autoscaler  New size: 8; reason: cpu resource utilization (percentage of request) above target
  Warning  FailedGetResourceMetric       10m (x3 over 10m)    horizontal-pod-autoscaler  failed to get cpu utilization: did not receive metrics for any ready pods
  Warning  FailedComputeMetricsReplicas  10m (x3 over 10m)    horizontal-pod-autoscaler  invalid metrics (1 invalid out of 1), first error is: failed to get cpu utilization: did not receive metrics for any ready pods
  Normal   SuccessfulRescale             9m54s (x2 over 19m)  horizontal-pod-autoscaler  New size: 4; reason: cpu resource utilization (percentage of request) above target
  Normal   SuccessfulRescale             9m39s                horizontal-pod-autoscaler  New size: 5; reason:
  Normal   SuccessfulRescale             8m53s                horizontal-pod-autoscaler  New size: 7; reason: cpu resource utilization (percentage of request) above target
  Normal   SuccessfulRescale             2m53s                horizontal-pod-autoscaler  New size: 4; reason: All metrics below target
  Normal   SuccessfulRescale             2m38s (x2 over 12m)  horizontal-pod-autoscaler  New size: 2; reason: All metrics below target
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