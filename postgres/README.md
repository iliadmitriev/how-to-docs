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

## Loking

1. https://www.postgresql.org/docs/current/monitoring-stats.html#WAIT-EVENT-LOCK-TABLE
2. https://www.postgresql.org/docs/current/explicit-locking.html

## Check master

```sql
SELECT pg_is_in_recovery()
select * from pg_stat_replication;
select * from pg_replication_slots;
SHOW transaction_read_only;
```

## Check slave replica

```sql
SHOW transaction_read_only;

select * from pg_stat_wal_receiver;
SELECT pg_wal_replay_pause();
SELECT pg_wal_replay_resume();
select now() - pg_last_xact_replay_timestamp();

SELECT
  pg_is_in_recovery() AS is_slave,
  pg_last_wal_receive_lsn() AS receive,
  pg_last_wal_replay_lsn() AS replay,
  pg_last_wal_receive_lsn() = pg_last_wal_replay_lsn() AS synced,
  (
   EXTRACT(EPOCH FROM now()) -
   EXTRACT(EPOCH FROM pg_last_xact_replay_timestamp())
  )::int AS lag;
```

## Pause/resume slave

```sql
SELECT pg_wal_replay_pause();
SELECT pg_get_wal_replay_pause_state();
SELECT pg_wal_replay_resume();
```
