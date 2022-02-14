
# Server

create sertivicates using [docs](https://docs.docker.com/engine/security/protect-access/)

create file `/etc/docker/daemon.json`
```json
{
    "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"],
    "tls": true,
    "tlscacert": "/etc/docker/ca.pem",
    "tlscert": "/etc/docker/server-cert.pem",
    "tlskey": "/etc/docker/server-key.pem",
    "tlsverify": true
}
```

## Ubuntu 20

```
systemctl status docker
vim /lib/systemd/system/docker.service
```

```
systemctl daemon-reload
```

```
systemctl stop docker
systemctl start docker
```

```
tail -f /var/log/syslog
```

# Client

```
export HOST=xx.xx.xx.xx
export PATH_CERT=/path/to/client/certs

docker context create remote_ctx --docker "host=tcp://$HOST:2376,ca=$PATH_CERT/ca.pem,cert=$PATH_CERT/cert.pem,key=$PATH_CERT/key.pem"

docker context use remote_ctx
```

```
docker version
```
