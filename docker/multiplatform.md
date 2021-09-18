# Docker multiplatform build

1. create buildx context and set is as default

```shell
docker buildx create --use --name mybuild
```

2. build and push to repository

```shell
docker buildx build --platform linux/amd64,linux/arm64 \
 -t iliadmitriev/mailcatcher:latest --push ./
```

3. stop context and switch to default

```shell
docker buildx stop
docker buildx use default
```
