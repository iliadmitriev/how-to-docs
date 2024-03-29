# Architecture overview

# Deploy etcd

Based on
https://github.com/iliadmitriev/etcd-cluster

1. Service
2. Stateful Set with 3 replicas (soft affinity)
3. Volume Claim

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: pg-etcd
  name: pg-etcd
spec:
  selector:
    app: pg-etcd
  ports:
    - name: client-port
      protocol: TCP
      port: 2379
      targetPort: 2379
    - name: peer-port
      protocol: TCP
      port: 2380
      targetPort: 2380
  type: ClusterIP
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: pg-etcd
spec:
  selector:
    matchLabels:
      app: pg-etcd
  serviceName: pg-etcd
  replicas: 3
  template:
    metadata:
      labels:
        app: pg-etcd
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 5
              podAffinityTerm:
                topologyKey: "kubernetes.io/hostname"
                labelSelector:
                  matchLabels:
                    app: pg-etcd
      containers:
      - name: pg-etcd
        imagePullPolicy: Always
        image: iliadmitriev/etcd-cluster
        readinessProbe:
          httpGet:
            scheme: HTTP
            path: /v2/keys/
            port: 2379
          initialDelaySeconds: 3
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        ports:
        - containerPort: 2379
          name: client-port
        - containerPort: 2380
          name: peer-port
        volumeMounts:
        - name: pg-etcd-pvc
          mountPath: /data
        env:
          - name: ETCD_CLUSTER_IP
            value: pg-etcd
          - name: HOSTNAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: HOST_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
          - name: ETCD_INITIAL_CLUSTER_TOKEN
            value: secret_token_for_etcd
  volumeClaimTemplates:
  - metadata:
      name: pg-etcd-pvc
      labels:
        app: pg-etcd
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

# Deploy postgres

Based on
https://github.com/iliadmitriev/postgres-cluster

1. Service
2. Stateful set with 3 replicas
3. Service master
4. Service Account, Role, Role Binding

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    # should match with labels of deployment and pods
    app: pg-postgres
  name: pg-postgres # .namespace.svc.cluster.local
spec:
  selector:
    # should match with labels of deployment and pods
    app: pg-postgres
  ports:
    - name: pg
      port: 5432
      targetPort: 5432
    - name: pgbouncer
      port: 6432
      targetPort: 6432
    - name: patroni
      port: 8008
      targetPort: 8008
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: pg-postgres # postgres-0.postgres-db.namespace.svc.cluster.local
spec:
  selector:
    matchLabels:
      app: pg-postgres
  serviceName: pg-postgres # should match .metadata.name of headless service
  replicas: 3
  template:
    metadata:
      labels:
        app: pg-postgres
    spec:
      serviceAccountName: pg-service-account
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 5
              podAffinityTerm:
                topologyKey: "kubernetes.io/hostname"
                labelSelector:
                  matchLabels:
                    app: pg-postgres
      containers:
      - name: pg-postgres-pod
        imagePullPolicy: Always
        image: iliadmitriev/postgres-cluster
        readinessProbe:
          httpGet:
            scheme: HTTP
            path: /readiness
            port: 8008
          initialDelaySeconds: 3
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        livenessProbe:
          httpGet:
            scheme: HTTP
            path: /liveness
            port: 8008
          initialDelaySeconds: 3
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        ports:
          - name: pg
            containerPort: 5432
          - name: pgbouncer
            containerPort: 6432
          - name: patroni
            containerPort: 8008
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
          - name: PATRONI_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: PATRONI_ETCD_HOST
            value: pg-etcd:2379
          - name: PATRONI_SCOPE
            value: main
          - name: PATRONI_POSTGRESQL_CONNECT_ADDRESS
            value: "$(POD_IP):5432"
          - name: PATRONI_RESTAPI_CONNECT_ADDRESS
            value: "$(POD_IP):8008"
          - name: PATRONI_POSTGRESQL_DATA_DIR
            value: /data/postgres/data
        volumeMounts:
        - name: pg-postgres-pvc
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: pg-postgres-pvc
      labels:
        app: pg-postgres
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: pg-postgres
    role: master
  name: pg-postgres-master
spec:
  selector:
    app: pg-postgres
    role: master
  ports:
    - name: pg
      port: 5432
      targetPort: 5432
    - name: pgbouncer
      port: 6432
      targetPort: 6432
    - name: patroni
      port: 8008
      targetPort: 8008
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pg-service-account
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pg-role
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - patch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pr-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pg-role
subjects:
- kind: ServiceAccount
  name: pg-service-account
```