# Config Map

config map for redis cluster
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-cluster-config
data:
  update-ip.sh: |
    #!/bin/sh
    sed -i -e "/myself/ s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/${IP}/" /data/nodes.conf
    exec "\$@"
  redis.conf: |+
    cluster-enabled yes
    cluster-config-file /data/nodes.conf
    appendonly yes
```

# Stateful set

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: redis-cluster
  replicas: 6
  selector:
    matchLabels:
      app: redis-cluster
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis
        image: redis:6.2.5-alpine3.14
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        command: ["/conf/update-ip.sh", "redis-server", "/conf/redis.conf"]
        env:
        - name: IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        volumeMounts:
        - name: conf
          mountPath: /conf
          readOnly: false
        - name: data
          mountPath: /data
          readOnly: false
      volumes:
      - name: conf
        configMap:
          name: redis-cluster-config
          defaultMode: 0755
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

# Build cluster

```shell
kubectl get pods -l app=redis-cluster \
 -o jsonpath='{range.items[*]}{.status.podIP}:6379 ' 

POD_IPS=$(...)

kubectl exec -it redis-cluster-0 -- \
   redis-cli --cluster create --cluster-replicas 2 $POD_IPS
```

# References

1. [Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)
2. [Deploy Redis Guestbook Microservice App on Kubernetes](https://platform9.com/docs/kubernetes/tutorials-deploy-redis-microservices-app)
