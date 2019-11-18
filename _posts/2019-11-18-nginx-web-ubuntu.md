---
layout: post
title: Zabbix Web + Nginx
author: foger
tags: zabbix monitoring nginx php
---

[Original source](https://github.com/zabbix/zabbix-docker/tree/4.4/web-nginx-mysql/ubuntu)

**/etc/nginx/nginx.conf**

```
user www-data;
worker_processes 5;
#worker_rlimit_nofile 256000;

error_log /dev/fd/2 warn;

pid        /var/run/nginx.pid;

events {
    worker_connections 5120;
    use epoll;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /dev/fd/1 main;

    client_body_timeout             5m;
    send_timeout                    5m;

    connection_pool_size            4096;
    client_header_buffer_size       4k;
    large_client_header_buffers     4 4k;
    request_pool_size               4k;
    reset_timedout_connection       on;


    gzip                            on;
    gzip_min_length                 100;
    gzip_buffers                    4 8k;
    gzip_comp_level                 5;
    gzip_types                      text/plain;
    gzip_types                      application/x-javascript;
    gzip_types                      text/css;

    output_buffers                  128 512k;
    postpone_output                 1460;
    aio                             on;
    directio                        512;

    sendfile                        on;
    client_max_body_size            8m;
    client_body_buffer_size	    256k;
    fastcgi_intercept_errors        on;

    tcp_nopush                      on;
    tcp_nodelay                     on;

    keepalive_timeout               75 20;

    ignore_invalid_headers          on;

    index                           index.php;
    server_tokens                   off;

    include /etc/nginx/conf.d/*.conf;
}
```

<!-- more -->

**/etc/php/7.2/fpm/conf.d/99-zabbix.ini**

```
max_execution_time=300
memory_limit=128M
post_max_size=16M
upload_max_filesize=2M
max_input_time=300
always_populate_raw_post_date=-1
max_input_vars=10000
date.timezone=Europe/Moscow
session.save_path=/var/lib/php7
```

**/etc/zabbix/nginx.conf**

```
server {
    listen          80;
    server_name     zabbix;
    index           index.php;

    access_log      /dev/fd/1 main;
    error_log       /dev/fd/2 notice;

    set $webroot '/usr/share/zabbix';

    root $webroot;

    large_client_header_buffers 8 8k;
    client_max_body_size 10M;


    location = /favicon.ico {
        log_not_found off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # deny running scripts inside writable directories
    location ~* /(images|cache|media|logs|tmp)/.*\.(php|pl|py|jsp|asp|sh|cgi)$ {
        return 403;
        error_page 403 /403_error.html;
    }

    # Deny all attempts to access hidden files such as .htaccess, .htpasswd, .DS_Store (Mac).
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # caching of files
    location ~* \.(ico|pdf|flv)$ {
        expires 1y;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|swf|xml|txt)$ {
        expires 14d;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ .php$ {
        fastcgi_pass   unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_index  index.php;

        fastcgi_param  SCRIPT_FILENAME  $webroot$fastcgi_script_name;

        include fastcgi_params;
        fastcgi_param  QUERY_STRING     $query_string;
        fastcgi_param  REQUEST_METHOD   $request_method;
        fastcgi_param  CONTENT_TYPE     $content_type;
        fastcgi_param  CONTENT_LENGTH   $content_length;
        fastcgi_intercept_errors        on;
        fastcgi_ignore_client_abort     off;
        fastcgi_connect_timeout 60;
        fastcgi_send_timeout 180;
        fastcgi_read_timeout 180;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_temp_file_write_size 256k;
    }
}
```

**/etc/zabbix/nginx_ssl.conf**

```
server {
    listen          443 ssl http2;
    server_name     zabbix;
    server_name_in_redirect off;

    index  index.php;
    access_log      /dev/fd/1 main;
    error_log       /dev/fd/2 error;

    set $webroot '/usr/share/zabbix';

    root $webroot;

    large_client_header_buffers 8 8k;

    client_max_body_size 10M;


    ssl on;
#    ssl_stapling on;
    ssl_certificate     /etc/ssl/nginx/ssl.crt;
    ssl_certificate_key /etc/ssl/nginx/ssl.key;
    ssl_dhparam /etc/ssl/nginx/dhparam.pem;

    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_verify_depth 3;
    ssl_session_cache    shared:SSL:10m;
    ssl_session_timeout  10m;
    ssl_prefer_server_ciphers on;

    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
    add_header Content-Security-Policy-Report-Only "default-src https:; script-src https: 'unsafe-eval' 'unsafe-inline'; style-src https: 'unsafe-inline'; img-src https: data:; font-src https: data:; report-uri /csp-report";

    location =/nginx_status {
        stub_status on;
        access_log   off;
        allow 127.0.0.1;
        deny all;
    }

    location = /favicon.ico {
        log_not_found off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }   

    # deny running scripts inside writable directories
    location ~* /(images|cache|media|logs|tmp)/.*\.(php|pl|py|jsp|asp|sh|cgi)$ {
        return 403;
        error_page 403 /403_error.html;
    }

    # Deny all attempts to access hidden files such as .htaccess, .htpasswd, .DS_Store (Mac).
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # caching of files
    location ~* \.(ico|pdf|flv)$ {
        expires 1y;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|swf|xml|txt)$ {
        expires 14d;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ .php$ {
        fastcgi_pass   unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_index  index.php;

        fastcgi_param  SCRIPT_FILENAME  $webroot$fastcgi_script_name;

        include fastcgi_params;
        fastcgi_param  QUERY_STRING     $query_string;
        fastcgi_param  REQUEST_METHOD   $request_method;
        fastcgi_param  CONTENT_TYPE     $content_type;
        fastcgi_param  CONTENT_LENGTH   $content_length;
        fastcgi_intercept_errors        on;
        fastcgi_ignore_client_abort     off;
        fastcgi_connect_timeout 60;
        fastcgi_send_timeout 180;
        fastcgi_read_timeout 180;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_temp_file_write_size 256k;
    }
}
```
