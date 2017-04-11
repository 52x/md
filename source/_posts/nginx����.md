title: nginx配置以及基本命令使用
date: 2016-01-01 13:02:47
tags: [nginx,服务器端]
---


## nginx配置


```js

user  root; #定义Nginx运行的用户和用户组
worker_processes 1; #nginx进程数，建议设置为等于CPU总核心数。
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid;
events {
    use epoll;
    worker_connections  1024;#单个进程最大连接数（最大连接数=连接数*进程数）
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    server_tokens off;
    charset utf-8;
    server_names_hash_bucket_size 128; #服务器名字的hash表大小
    client_header_buffer_size 5m; #上传文件大小限制
    large_client_header_buffers 4 64k; #设定请求缓
    client_max_body_size 8m; #设定请求缓

    tcp_nopush on; #防止网络阻塞
    tcp_nodelay on; #防止网络阻塞
    keepalive_timeout 120; #长连接超时时间，单位是秒

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
   
    gzip  on;
    gzip_min_length 1k;
    gzip_buffers 4 8k;
    gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2; #压缩等级
    gzip_types  text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    upstream node_app {
      server 127.0.0.1:3001;
    }

    upstream node_zy_app {
       server 127.0.0.1:3012;
     }

  server {
      listen  80;
      server_name www.domain.com;
      index index.html index.htm;
      root /data/host/manage/server/;#路径
      
      access_log /data/host/manage/server/logs/host.access.log;#日志路径
      error_log /data/host/manage/server/logs/host.error.log;#日志路径
      location / {
            proxy_pass http://node_app;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;#设定ip
      }

      location ~.*\.(gif|jpg|jpeg|png|bmp|swf|js|css|ejs|htm|html|txt) {#静态文件处理
            root   /data/host/manage/server/public/;
            index  index.html index.htm;
            expires max;
        }
    }

   server {
    	listen 80;
    	server_name domain1.com;
    	deny 217.112.97.187;# ip禁止
    	deny 162.144.87.81;
    	deny 171.8.158.168;
    	deny 178.162.212.20;
        deny 171.88.49.3;
	    rewrite  ^/(.*) http://www.domain1.net/$1 permanent;
   }


    # HTTPS server # https 配置
    #

    server {
        listen     443 ssl;
        server_name  www.domain2.com:443;
        ssl  on;
        


        deny  218.28.166.162;
        ssl_certificate      ../ssl/domain2.com/1_domain2.com_bundle.crt;
        ssl_certificate_key  ../ssl/domain2.com/2_domain2.com.key;

        ssl_session_cache    shared:SSL:10m;
        ssl_session_timeout  10m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            rewrite ^(.*)  http://www.domain2.com$1 permanent;
         }

        location ~* /reg {
            proxy_pass http://node_app;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location ~* /login {
            proxy_pass http://node_app;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location ~* /space {
            proxy_pass http://node_app;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location ~ \.(css|js|gif|html|ttf|woff|swf)$ {
           proxy_pass http://node_app;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }


}


```


## nginx基本命令
#### 配置文件正确性检测
```js
nginx -t 
or
nginx -t -c /opt/nginx.conf
```

#### 显示版本号

```js
nginx -v
```

#### 启动
```js
nginx 
or
nginx -c /opt/nginx.conf
```

#### 关闭

```js
nginx -s stop 
```
or 

```js
ps -ef | grep nginx
kill -9 pid
```

