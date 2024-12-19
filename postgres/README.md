# Postgres

Create SSL CA and certificates

```bash
git clone https://github.com/iliadmitriev/openssl-scripts ssl
cd ssl
./create_ca.sh
./create_cert.sh localhost
cd ../
```

```bash

export PGBASE=main
export PGPASS=secret

docker volume create postgres-${PGBASE}

docker run -d -p 5432:5432 \
      -v ${PGBASE}-postgres:/var/lib/postgresql/data \
      -v ${PWD}/ssl/localhost.pem:/ssl/localhost.pem:ro \
      -v ${PWD}/ssl/localhost.key:/ssl/localhost.key:ro \
      -e POSTGRES_USER=${PGBASE} \
      -e POSTGRES_DB=${PGBASE} \
      -e POSTGRES_PASSWORD=${PGPASS} \
      --name ${PGBASE}-postgres \
 postgres:alpine \
    -c ssl=on \
    -c ssl_cert_file=/ssl/localhost.pem \
    -c ssl_key_file=/ssl/localhost.key
```

Cleanup

```bash
docker rm -f ${PGBASE}-postgres
docker volume rm ${PGBASE}-postgres
```
