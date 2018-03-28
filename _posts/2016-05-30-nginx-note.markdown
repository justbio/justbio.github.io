---
layout: post
category: "read"
title:  "NGINX.CONF各参数的意义"
---

搬运+翻译至 http://qiita.com/syou007/items/3e2d410bbe65a364b603

/etc/nginx/nginx.conf  
记录各个参数的意义

#### user

>user nginx;  
nginx开启后会启动3个进程master process，worker process，cache manager process。
本参数指定了master process以外的进程的用户。
master process是用root启动的。

<!-- more -->

#### worker_processes

>worker_processes auto  
worker_processes 2  
指定Nginx运行时使用的CPU核数。  
设成auto会自动判断CPU的核数。  

用以下命令可以看CPU核数。
>$ grep processor /proc/cpuinfo |wc

以下参数指定了哪个cpu分配给哪个进程，一般来说不用特殊指定。如果一定要设的话，用0和1指定分配方式，比如：
>worker_processes 4     #4核CPU  
worker_cpu_affinity 0001 0010 0100 1000    #这样设就是给1-4个进程分配单独的核来运行，出现第5个进程是就是随机分配了

#### worker_rlimit_nofile

>worker_rlimit_nofile 4096;
设置毎个进程的最大文件打开数。如果不设的话上限就是系统的ulimit –n的数字。
一般来说设成下面提到的worker_connections的3-4倍就够用了。

#### error_log

>error_log /var/log/nginx/error.log;
nginx的日志，没特殊要求的话默认值就可以了。

#### pid

>pid /var/run/nginx.pid;
指定pid文件的位置，默认值就可以。

### Events模块

events {...}

用来定义Event模块。  
以下3个项目需要记载在event模块中

#### worker_connections

>worker_connections 1024;  
一个worker进程的最大连接数。默认为512，按自己系统的硬件配置调整，不能超过worker_rlimit_nofile。

#### multi_accept

>multi_accept on;  
默认是on。设置为on后，多个worker按串行方式来处理连接，也就是一个连接只有一个worker被唤醒，其他的处于休眠状态。  
设置为off后，多个worker按并行方式来处理连接，也就是一个连接会唤醒所有的worker，知道连接分配完毕，没有取得连接的继续休眠。  
当你的服务器连接数不多时，开启这个参数会让负载有一定程度的降低。但是当服务器的吞吐量很大时，为了效率，请关闭这个参数。

#### use

use epoll  
Linux内核2.6以上为epoll，BSD为kqueue。

### http模块

http {...}

用作Web服务器的配置。

#### server_tokens

>server_tokens off;  
错误页面的标签上是否表示 Nginx的版本。
安全上的考虑还是off掉吧。

#### include

>include /etc/nginx/mime.types;  
定义MIME类型和后缀名关联的文件的位置。

```
types {
text/html html htm shtml;
text/css css;
text/xml xml;
image/gif gif;
image/jpeg jpeg jpg;
application/javascript js;
...
      }
```
mime.types文件中大概是这个样子的。

#### default_type

>default_type application/octet-stream;  
指定mime.types文件中没有记述到的后缀名的处理方法。  
默认值是text/plain。

#### log_format

>log_format main 'time:$time_iso8601\t'...  
log_format ltsv 'time:$time_iso8601\t'...

定义日志的格式。可以选择main或者ltsv，后面定义要输出的内容。
 1. $remote_addr 与$http_x_forwarded_for 用以记录客户端的ip地址；
 2. $remote_user ：用来记录客户端用户名称；
 3. $time_local ：用来记录访问时间与时区；
 4. $request ：用来记录请求的url与http协议；
 5. $status ：用来记录请求状态； 
 6. $body_bytes_s ent ：记录发送给客户端文件主体内容大小；
 7. $http_referer ：用来记录从那个页面链接访问过来的；
 8. $http_user_agent ：记录客户端浏览器的相关信息；。

#### access_log

>access_log /var/log/nginx/access.log main;  
连接日志的路径，上面指定的日志格式放在最后。  
access_log off;  
也可以关掉。

 

#### charset

>charset UTF-8;  
设置应答的文字格式。

#### sendfile

>sendfile on;  
指定是否使用OS的sendfile函数来传输文件。  
普通应用应该设为on，下载等IO重负荷的应用应该设为off。默认值是off。 

#### tcp_nopush

>tcp_nopush on;  
sendfile为on时这里也应该设为on，数据包会累积一下再一起传输，可以提高一些传输效率。

#### tcp_nodelay

>tcp_nodelay on;  
小的数据包不等待直接传输。默认为on。  
看上去是和tcp_nopush相反的功能，但是两边都为on时nginx也可以平衡这两个功能的使用。

#### keepalive_timeout

>keepalive_timeout 75;  
HTTP连接的持续时间。设的太长会使无用的线程变的太多。  
设成0关闭此功能。  
默认为75。  

#### keepalive_requests

>keepalive_requests 100;  
keepalive_timeout时效内同样的客户端超过指定数量的连接时会被强制切断。  
一般的话keepalive_timeout 5和keepalive_requests 20差不多就够了。  
默认为100。

#### set_real_ip_from和real_ip_header

>set_real_ip_from 10.0.0.0/8;  
real_ip_header X-Forwarded-For;  
可以防止经过代理或者负载均衡服务器时丢失源IP。  
set_real_ip_from指定代理或者负载均衡服务器的IP，可以指定复数个IP。  
real_ip_header指定从哪个header头检索出要的IP地址。  

#### client_header_timeout和client_body_timeout

>client_header_timeout 10;  
client_body_timeout 10;  
读取客户端的请求head部分和客户端的请求body部分的超时时间。

#### client_body_buffer_size和client_body_temp_path

>client_body_buffer_size 32k;  
client_body_temp_path /dev/shm/client_body_temp 1 2;  
接受的请求body部分到client_body_buffer_size为止放在内存中，超出的部分输出至client_body_temp_path文件里。

 

#### client_max_body_size

>client_max_body_size 1m;  
客户端上传的body的最大值。超过最大值就会发生413(Request Entity Too Large)错误。  
默认为1m，最好改大一点。

#### client_header_buffer_size和large_client_header_buffers

>client_header_buffer_size 1k;  
large_client_header_buffers 4 8k;  
一般来说默认就够了。  
发生414 (Request-URI Too Large) 错误时请增大这两个参数。

#### limit_conn和limit_conn_zone

>limit_conn_zone $binary_remote_addr zone=addr:10m;  
limit_conn addr 100;  
限制某条件下的同时连接数。

#### Proxy相关

>proxy_buffering on;  
proxy_buffer_size 8k;  
proxy_buffers 100 8k;  
proxy_cache_path /var/lib/nginx/cache levels=1:2 keys_zone=CACHE:512m inactive=1d max_size=60g;  
作为反向代理时需要用到的参数。

#### gzip相关

>gzip on;  
gzip_http_version 1.0;  
gzip_comp_level 2;  
gzip_proxied expired no-cache no-store private auth;  
gzip_vary off;  
gzip_types text/plain  
text/css  
text/xml  
...  
application/json;  
gzip_min_length 1000;  
gzip_disable "MSIE [1-6]\.";  
应答时使用gzip时设为on。可以减少服务器间的数据传输量。  
gzip_types：只对指定的文件类型起效。  
gzip_proxied：只对指定的请求类型起效。  
gzip_min_length：length比此值小的不压缩。  
gzip_disable：其他不压缩的情况，一般设为IE6以下。  

#### open_file_cache相关

>open_file_cache max=100 inactive=10s;  
open_file_cache_valid 30s;  
open_file_cache_min_uses 2;  
open_file_cache_errors on;  
文件描述信息，大小，更新时间等信息可以保存在cache中。

#### server_names_hash_bucket_size

>server_names_hash_bucket_size 64  
nginx启动时出现could not build the server_names_hash, you should increase错误时请提高这个参数的值  
一般设成64就够了。

#### types_hash_max_size

>types_hash_max_size 1024;  
types_hash_max_size影响散列表的冲突率。  
types_hash_max_size越大，就会消耗更多的内存，但散列key的冲突率会降低，检索速度就更快。  
types_hash_max_size越小，消耗的内存就越小，但散列key的冲突率可能上升。  
默认为1024

#### types_hash_bucket_size

>types_hash_bucket_size 64;  
types_hash_bucket_size 设置了每个散列桶占用的内存大小。  
默认为64

#### listen

>listen 80 default_server;  
nginx当网页服务器使用的时候写在http模块中，一般来说用作虚拟主机的情况下不会写在这里。

#### server_name_in_redirect

>server_name_in_redirect off;  
重定向的时候需不需要把服务器名写入head，基本上不会设成on。

#### port_in_redirect

>port_in_redirect on;  
设为on后，重定向的时候URL末尾会带上端口号。

#### upstream

>upstream resinserver{  
ip_hash;  
server 127.0.0.1:8000 down;  
server 127.0.0.1:8080 weight=2;  
server 127.0.0.1:6801;  
server 127.0.0.1:6802 backup;  
                    }  
Ip_hash:每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题  
down：不参与负责均衡  
weight：比重，越大的分配的越多  
backup：其他的非backup机器都down或者忙的时候才会请求到这台机器  

#### proxy_set_header等
 
>proxy_pass http://127.0.0.1:3000;  
proxy_set_header Host $host;  
proxy_set_header X-Real-IP $remote_addr;  
proxy_set_header X-Forwarded-Server $host;  
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
proxy_hide_header X-Powered-By;  
proxy_ignore_headers Expires;  
重定义发送至后端服务器的head。  
一般这样设置就没问题了。  

#### proxy_redirect

>proxy_redirect off;  
重定向到后端服务器时的location head。on 时按照proxy_pass重定向。  
Off时按照服务器的指示重定向。

 

#### error_page，proxy_intercept_errors

>proxy_intercept_errors on;  
error_page 404 /404.html;  
error_page 403 =404 /notfound.html; # 403变404  
error_page 500 502 503 504 /50x.html;  
错误页面的显示。

#### include

>include /etc/nginx/vhost.d/*.conf;  
放在这个文件夹的设定文件也可以被读取。

#### Server模块

```
http {  
server {  
...  
       }  
     }  
```
Nginx用作虚拟主机时使用。  
每一个server模块生成一个虚拟主机。  
写在http模块内部。

#### Listen和server_name

>listen 80;  
server_name localhost;  
连接虚拟主机的信息。  
listen指定端口号。  
server_name指定服务器的域名。  

#### root

>root /path/public  
定义服务器的默认网站根目录位置。

#### rewrite

>rewrite /(.*)/index.html $1.html permanent;  
需要重定向的时候使用。

 

#### satisfy,auth_basic

>satisfy any;  
auth_basic "basic authentication";  
auth_basic_user_file /etc/nginx/.htpasswd;  
satisfy any|all 部分地址Basic认证的方式  
![](../assets/table1.PNG)

>auth_basic：认证的名称  
auth_basic_user_file：密码文件  

 

 

#### try_files

>try_files $uri $uri.html $uri/index.html @unicorn;  
从左边开始找指定文件是否存在。  
比如连接http://***/hoge时按hoge.html、hoge/index.html、location @unicorn {}的顺序查找。

#### Location模块

```
http {  
server {  
location / {  
...  
           }   
       }  
     }  
```
指定位置（文件或路径）时使用。  
也可以用正则表达式。  

>location ~ /\.(ht|svn|git) {  
deny all;  
                           }  
不想让用户连接.htaccess，.svn，.git文件时用上面的设置。

 

#### stub_status，allow，deny

>stub_status on;  
access_log off;  
allow 127.0.0.1;  
deny all;  
stub_status连接指定的位置时可以显示现在的连接数。一般来说不会公开。  
Allow允许指定IP的连接。  
Deny拒绝指定IP的连接。  

Nginx规则是从上到下判断的，上面的例子先判断的是allow 127.0.0.1，所以127.0.0.1允许连接。  
如果反过来设成下面这样，所有的IP都被拒绝了。（注意和Apache不一样。）

>deny all;  
allow 127.0.0.1;  

#### expires

>expires 10d;  
使用浏览器缓存时设置。上面的例子用了10天内的浏览器缓存。

#### add_header

>add_header Cache-Control public;  
设置插入response header的值。  
不是很懂…

#### break，last

>break|last;
rewrite后接break指令，完成rewrite之后会执行完当前的location（或者是if）指令里的其他内容  
（停止执行当前这一轮的ngx_http_rewrite_module指令集），然后不进行新URL的重新匹配。
rewrite后接last指令，在完成rewrite之后停止执行当前这一轮的ngx_http_rewrite_module指令集  
已经后续的指令，进而为新的URL寻找location匹配。

#### internal

```
error_page 404 /404.html;  
location /404.html {  
internal;  
                   }  
```
只在内部重定向的时候使用。  
上面的例子就无法直接访问/404.html页面。

### 附：配置文件例
```
user  nginx;
worker_processes  4;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
    server {
    listen       80;
    server_name  www.hoge.co.jp;
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    proxy_set_header Host            $host;
    proxy_set_header X-Real-IP       $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    location / {
        proxy_pass http://backend;
    }
}
upstream backend {
      ip_hash;
      server 172.20.1.201:80 max_fails=3 fail_timeout=30s ;
      server 172.20.1.202:80 max_fails=3 fail_timeout=30s ;
      server 127.0.0.1:8080  down;
}
}
```