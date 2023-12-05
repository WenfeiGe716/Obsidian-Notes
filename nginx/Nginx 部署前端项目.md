
## 1、创建目录结构：
```shell
 .  
 ├── docker-compose.yaml # 配置docker容器  
 └── 项目名  
     ├── conf.d   
     │   └── lnzh.conf # 前端项目配置文件，包括路由、端口等  
     ├── nginx.conf # nginx 的配置文件  
     └── www  
         └── dist # 前端打包之后生成的文件
```

## 2、docker-compose.yaml

```docker-co
 version: '3.8'  
 ​  
 networks:  
   lnzh:  
     name: lnah_network  
 ​  
 services:  
   lnzh-bigData-web:  
     image: nginx:latest # 镜像名称  
     container_name: lnzh-bigData-web # 容器名字  
     restart: always # 开机自动重启  
     networks:  
       - lnzh  
     ports:  
       # 端口号绑定（宿主机:容器内）  
       - '8000:80'  
     volumes:  
       # 目录映射（宿主机:容器内）  
       - ./lnzhBigData/conf.d:/etc/nginx/conf.d  
       - ./lnzhBigData/www:/usr/share/nginx/html  
       - ./lnzhBigData/nginx.conf:/etc/nginx/nginx.conf  
```
 ​

## 3、lnzh.conf

 server {  
         listen 80 default_server;  
         #listen 443 ssl;  
         # server_name ccf.lab.cetcxl.com;  
 ​  
         #ssl_certificate     /etc/nginx/ssl/ccf.lab.cetcxl.com.pem;  
         #ssl_certificate_key /etc/nginx/ssl/ccf.lab.cetcxl.com.key;  
 ​  
         location ~ /\. {  
                 deny all;  
         }  
 ​  
         location / {  
                 #rewrite ^/(.*)$ /$1 break;  
                 root /usr/share/nginx/html/dist;  
                 try_files $uri $uri/ /index.html;  
         }  
 ​  
     #     location /api {  
     #             rewrite ^/api/(.*) /$1 break;  
     #             proxy_pass http://172.22.70.146:8091;  
     #             proxy_buffer_size 64k;  
     #             proxy_buffers   32 32k;  
     #             proxy_busy_buffers_size 128k;  
     #             proxy_http_version 1.1;  
     #             proxy_set_header Connection "Keep-Alive";  
     #             client_max_body_size 100M;  
     #     }  
     #    location /baas {  
     #             rewrite ^/api/(.*) /$1 break;  
     #             proxy_pass http://172.22.70.146:12001;  
     #             proxy_buffer_size 64k;  
     #             proxy_buffers   32 32k;  
     #             proxy_busy_buffers_size 128k;  
     #             proxy_http_version 1.1;  
     #             proxy_set_header Connection "Keep-Alive";  
     #             client_max_body_size 100M;  
     #     }  
 ​  
         # location ^~ /download {  
         #         alias /usr/share/nginx/html/public/$1;  
         #         default_type application/pdf;  
         #         add_header Content-Disposition 'inline';  
         # }  
         # 访问静态资源  
         location ~* \.(pdf|jpg|jpeg)$ {  
                 alias /usr/share/nginx/html/public/$uri;  
                 default_type application/pdf;  
                 add_header Content-Disposition 'inline';  
         }  
         # 定义 404 页面  
         error_page  404              /404.html;  
         location = /404.html {  
             root   /usr/share/nginx/html/dist;  
         }  
           
         # 定义 50x 页面  
         error_page   500 502 503 504  /50x.html;  
         location = /50x.html {  
             root   /usr/share/nginx/html/dist;  
         }  
 }

## 4、nginx.conf

 user  root; # 主机用户  
 worker_processes  auto;  
 ​  
 error_log  /var/log/nginx/error.log notice;  
 pid        /var/run/nginx.pid;  
 ​  
 ​  
 events {  
     worker_connections  1024;  
 }  
 ​  
 ​  
 http {  
     include       /etc/nginx/mime.types;  
     default_type  application/octet-stream;  
 ​  
     proxy_set_header        Host            $host;  
     proxy_set_header        X-Real-IP       $remote_addr;  
     client_max_body_size    1024m;  
     #proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;  
     proxy_set_header        X-Forwarded-For $remote_addr;  
     proxy_set_header        x-forwarded-for $remote_addr;  
     client_body_buffer_size 128k;  
     proxy_connect_timeout   600s;  
     proxy_send_timeout      600s;  
     proxy_read_timeout      600s;  
     #proxy_buffers           32 4k;  
 ​  
     fastcgi_buffers 8 16k;  
     fastcgi_buffer_size 32k;  
     fastcgi_connect_timeout 300;  
     fastcgi_send_timeout 300;  
     fastcgi_read_timeout 300;  
 ​  
     log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '  
                       '$status $body_bytes_sent "$http_referer" '  
                       '"$http_user_agent" "$http_x_forwarded_for"';  
 ​  
     access_log  /var/log/nginx/access.log  main;  
 ​  
     sendfile        on;  
     #tcp_nopush     on;  
 ​  
     keepalive_timeout  65;  
 ​  
     #gzip  on;  
 ​  
     include /etc/nginx/conf.d/*.conf;  
 }

5、运行

 docker-compose up -d