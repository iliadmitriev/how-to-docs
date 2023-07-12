# Setting up nginx webdav with http basic auth


Install apache2 tools for adding users
```shell
sudo yum install httpd-tools nginx
sudo touch /etc/nginx/htpasswd
sudo htpasswd -c /etc/nginx/htpasswd username
```

create `nginx.conf`
```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;


    server {
        listen       80 default_server;
        server_name  _;
        root         /nginx/root;

        location / {

            client_body_temp_path /nginx/client_temp;
            client_max_body_size 100m;

            dav_methods PUT DELETE MKCOL COPY MOVE;
            dav_access            group:rw  all:r;

            autoindex on;
            create_full_put_path  on;

            limit_except GET PROPFIND OPTIONS HEAD {
                auth_basic           "Restricted Area Foodtech";
                auth_basic_user_file /etc/nginx/htpasswd;
            }
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

start service
```shell
service nginx start
```

enable service
```shell
sudo systemctl enable nginx
```
