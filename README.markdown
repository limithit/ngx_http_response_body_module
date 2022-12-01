Name
====

ngx_http_response_body_module - extract body response into variable.

Table of Contents
=================

* [Name](#name)
* [Status](#status)
* [Synopsis](#synopsis)
* [Description](#description)
* [Configuration directives](#configuration-directives)

Status
======

This library is production ready.

Description
===========

Capture response body into nginx $response_body variable.

[Back to TOC](#table-of-contents)

Synopsis
========

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  escape=json '$remote_addr - $remote_user [$time_local] "$request" "$request_body" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" "$http_cookie"';

    log_format  safe escape=json '$remote_addr - $remote_user [$time_local] "$request" "$request_body"" || "[$response_body]"'
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                      '"$upstream_addr" "$upstream_response_time"';


    access_log  logs/access.log  main;  
     map $login_status $cond_login {
                301       0;
                default   1;
        }
  
    map $status $cond {
      500       1;
      401       1;
      403       1;
      404       1;
      default   0;
    }

    capture_response_body                   off;
    capture_response_body_buffer_size       1m;
    capture_response_body_buffer_size_min   4k;
    capture_response_body_buffer_size_multiplier   2;
    capture_response_body_if                $cond_login 1;
    capture_response_body_if_latency_more   1s;

    map $response_body $err {
      ~\"error\":\"(?<e>.+)\" $e;
      default "";
    }  

   include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    server_tokens off;
    proxy_hide_header X-Powered-By;
    proxy_hide_header Server;


    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header   REMOTE-HOST $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;  
    proxy_buffer_size 256k;
    proxy_buffers 8 256k;
    proxy_busy_buffers_size 512k;
    proxy_temp_file_write_size 512k;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade"; 
  

    client_max_body_size 1000m;
    client_header_buffer_size 256k;
    server_tokens off;
    fastcgi_buffer_size 256k;
    fastcgi_buffers 8 256k;
    fastcgi_busy_buffers_size 512k;
    fastcgi_temp_file_write_size 512k;  

    server_names_hash_bucket_size 128;
    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.

    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
       return 444;
    }

     upstream portal_server{
            server 10.1.0.131:80;
            server 10.1.0.132:80;
        }
   server {
          listen       80;
          server_name  workstation.xx.com workstation-static.xx.com;
          rewrite ^(.*)$  https://$host$1 permanent;
      }

        server {
        listen 443 ssl;
        server_name workstation.xx.com workstation-static.xx.com;
              ssl_certificate       cert/xx.com.pem;
              ssl_certificate_key   cert/xx.com.key;
              ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
              ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
              ssl_prefer_server_ciphers on;
              ssl_session_timeout 5m;

          access_log /var/log/nginx/workstation_access.log;
          error_log /var/log/nginx/workstation_error.log;
          location = /favicon.ico {
                log_not_found   off;
                access_log off;
        }
          
        location /{
                 set $flag 0;
                if ($document_uri ~* "login"){
                        set $flag "${flag}1";
                }
                if ($request_method = POST ) {
                        set $flag "${flag}2";
                 }
                  if ($flag = "012"){
                 access_log /var/log/nginx/login.log safe;
                 }
                proxy_pass http://portal_server/;
                proxy_http_version 1.1;
        }
 }

}
  
```

[Back to TOC](#table-of-contents)

Configuration directives
========================

capture_response_body
--------------
* **syntax**: `capture_response_body on|off`
* **default**: `off`
* **context**: `http,server,location`

Turn on response body capture.

capture_response_body_var
--------------
* **syntax**: `capture_response_body_var <name>`
* **default**: `response_body`
* **context**: `http,server,location`

Variable name.

capture_response_body_buffer_size
--------------
* **syntax**: `capture_response_body_buffer_size <size>`
* **default**: `pagesize`
* **context**: `http,server,location`

Maximum buffer size.

capture_response_body_buffer_size_min
--------------
* **syntax**: `capture_response_body_buffer_size_min <size>`
* **default**: `pagesize`
* **context**: `http,server,location`

Minimum amount of memory allocated for chunked response.

capture_response_body_buffer_size_multiplier
--------------
* **syntax**: `capture_response_body_buffer_size_multiplier <n>`
* **default**: `2`
* **context**: `http,server,location`

Reallocation multiplier.

capture_response_body_if
--------------
* **syntax**: `capture_response_body_if <complex variable> <value>`
* **default**: `none`
* **context**: `http,server,location`

Capture response body if result of calculation <complex variable> is equal <value>.  
<value> may be empty string or special '*'.

capture_response_body_if_1xx
--------------
* **syntax**: `capture_response_body_if_1xx on`
* **default**: `off`
* **context**: `http,server,location`

Capture response body for http statuses 1xx.

capture_response_body_if_2xx
--------------
* **syntax**: `capture_response_body_if_2xx on`
* **default**: `off`
* **context**: `http,server,location`

Capture response body for http statuses 2xx.

capture_response_body_if_3xx
--------------
* **syntax**: `capture_response_body_if_3xx on`
* **default**: `off`
* **context**: `http,server,location`

Capture response body for http statuses 3xx.

capture_response_body_if_4xx
--------------
* **syntax**: `capture_response_body_if_4xx on`
* **default**: `off`
* **context**: `http,server,location`

Capture response body for http statuses 4xx.

capture_response_body_if_5xx
--------------
* **syntax**: `capture_response_body_if_5xx on`
* **default**: `off`
* **context**: `http,server,location`

Capture response body for http statuses 5xx.

capture_response_body_if_latency_more
--------------
* **syntax**: `capture_response_body_if_latency_more <sec>`
* **default**: `none`
* **context**: `http,server,location`

Capture response body only if request time is greather than specified in the parameter.

[Back to TOC](#table-of-contents)
