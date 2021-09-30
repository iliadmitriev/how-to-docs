# MySQL / MariaDB

#### create MariaDB docker instance M1

```shell
# create env variables
cat > .env_mysql << _EOF
MYSQL_ROOT_PASSWORD=rootsecret
MYSQL_DATABASE=user
MYSQL_USER=user
MYSQL_PASSWORD=secret
_EOF

docker run -d --name user-mariadb --hostname user-mariadb \
           --env-file .env_mysql -p 3306:3306 mariadb
```

#### install macOS MySQL client

```shell
brew install mysql
```

#### connect from macOS to MariaDB 

```shell
mysql -h 192.168.10.1 -P 3306 -u user -D user -p
```
