
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
