# Nginx服务器配置详解

*Mission Start!*

Nginx的配置结构大致如下：

```conf
events {
  use epoll;
}

http {
  #http
  server {
    listen 80;
    ...
    
    location / {
        if() {
          ...
        }
    }
    ...
  }
  
  #https
  server {
    listen 443;
    ...
  }
  
  ...
}
```
## 一、location规则
格式如下：    

```conf
location [ = | ^~ | ~ | ~* | / ] [字符串 | 正则表达式] {
    [相关处理]
}
```
- ① = 表示精确匹配，例如：location = / {...}表示只匹配根目录的请求，后加字符串的不能匹配
- ② ^~ 表示匹配URI以常规字符串开头的请求，不是正则匹配，例如：location ^~ /home {...}只能匹配/home/index、/home/other/other等以其开头的请求
- ③ ~ 表示区分大小写的正则匹配
- ④ ~* 表示不区分大小写的正则匹配
- ⑤ / 表示通用匹配，任何请求都会匹配
    
优先级：(location =) > (location 完整路径) > (location ^~) > (location ~,~*) > (location 部分起始路径) > (location /)

## 二、rewrite规则
作用：使用nginx的全局变量及自定义变量实现URL的重写及重定向（放在server、location、if中）。   
   
rewrite和location具有相似之处，其区别在于：
> location主要用于访问控制、反向代理   
> rewrite主要用于实现URL的重写、跳转

rewrite规则格式如下：

```conf
rewrite 正则表达式 目标URL 标志位;
```

例如：

```conf
server {
  rewrite /(.*)$ /index.php/$1 last; # 隐藏URL中的index.php

  location ~ (.*)/index\.php$ {
    ...
  }
}
# http://www.xxxx.com/home/controller/action/param请求到来
# ① server中的rewrite匹配到该请求，并将URL重写为http://www.xxxx.com/index.php/home/controller/action/param
# ② location匹配到重写后的URL（http://www.xxxx.com/index.php/home/controller/action/param），执行location中的操作
```

### 1、标志位
- ① last 表示完成rewrite
- ② break 表示停止当前虚拟主机的后续rewrite指令
- ③ redirect 表示返回302临时重定向，地址栏会显示跳转后的地址
- ④ permanent 表示返回301永久重定向，地址栏显示跳转后的地址

### 2、if指令
格式如下：

```conf
if(condition) {
  ...
}
```

需要注意的是：

- ① 判断是否相等使用 = 或 !=
- ② ~ 正则表达式匹配，~* 不区分大小写的匹配，!~ 区分大小写的不匹配
- ③ -f 和 !-f 用于判断是否存在文件
- ④ -d 和 !-d 用来判断是否存在目录
- ⑤ -e 和 !-e 用来判断是否存在文件或目录
- ⑥ -x 和 !-x 用来判断文件是否可执行

if使用用例：

```conf
if ($http_user_agent ~ MSIE) {
    ...
}
```

### 3、Nginx全局变量
- $args：这个变量等于请求行中的参数，同$query_string
- $content_length：请求头中的Content-length字段。
- $content_type：请求头中的Content-Type字段。
- $document_root：当前请求在root指令中指定的值。
- $host：请求主机头字段，否则为服务器名称。
- $http_user_agent：客户端agent信息
- $http_cookie：客户端cookie信息
- $limit_rate：这个变量可以限制连接速率。
- $request_method：客户端请求的动作，通常为GET或POST。
- $remote_addr：客户端的IP地址。
- $remote_port：客户端的端口。
- $remote_user：已经经过Auth Basic Module验证的用户名。
- $request_filename：当前请求的文件路径，由root或alias指令与URI请求生成。
- $scheme：HTTP方法（如http，https）。
- $server_protocol：请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
- $server_addr：服务器地址，在完成一次系统调用后可以确定这个值。
- $server_name：服务器名称。
- $server_port：请求到达服务器的端口号。
- $request_uri：包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
- $uri：不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
- $document_uri：与$uri相同。

## 三、upstream模块与负载均衡
Nginx的反向代理是其出色的特点之一，通过反向代理可以实现负载均衡，将请求分发到其他服务器。而upstream节点就是配置反向代理的关键所在。   
   
### 配置步骤
① 在http节点下加入upstream节点：

```conf
upstream upstreamName {
    server ip1:port1;
    server ip2:port2;
    ...
}
```
② 在server节点下的location节点中，将相应的请求通过proxy_pass转发到http://upstreamName下：

```conf
location / {
    ...
    proxy_pass http://upstreamName;
    ...
}
```
   
至此，负载均衡已经配置完毕，Nginx会根据轮询的策略将请求按时间顺序逐一分配到不同的后端服务器上。   
   
当然，还存在其他的分配策略。

### 请求分配策略
#### ① 轮询   
默认情况下，Nginx使用轮询的方式进行请求的分配，这种方式虽然配置简单，但缺点在于可靠性不高、请求分发不均衡。   
   
#### ② weight权重   
在upstream节点中指定server的weight权重，Nginx会根据每个服务器的权重调整分发请求的几率，weight与分发请求的几率成正比，例如：

```conf
upstream upstreamName{
    server ip1:port1 weight=1;
    server ip2:port2 weight=2;
    # 转发请求给ip2服务器的几率是转发给ip1服务器的两倍
    ...
}
```

#### ③ ip_hash   
每个请求都按照访问ip的hash结果进行分配，即访问ip与服务器绑定，同一个用户的请求都会固定转发到一台服务器上进行处理（该方法可以用于解决分布式session不一致的问题）。使用方法：

```conf
upstream upstreamName{
    ip_hash;
    server ip1:port1;
    server ip2:port2;
    ...
}
```

#### ④ fair   
使用后端服务器的响应时间作为依据，响应时间短的服务器分配到更多的请求。

```conf
upstream upstreamName{
    fair;
    server ip1:port1;
    server ip2:port2;
}
```

### ⑤ url_hash
通过访问url的hash结果来分配请求，让url定向到同一个后端服务器：

```conf
upstream upstreamName{
    server ip1:port1;
    server ip2:port2;
    hash $request_uri;  
    hash_method crc32;  # 计算hash时使用的hash算法
    ### 使用hash后，不能使用weight等其他参数
}
```

### 状态值
upstream节点中，可以为每一个设备设置状态值，如下：
> down: 表示当前的server不参与负载均衡   
> weight: weight越大，收到的请求就越多   
> max_fails: 允许请求失败的最大次数，当超过该值时，返回proxy_next_upstream模块定义的错误   
> fail_timeout: 达到max_fails次失败时，暂停的时间   
> backup: 后备服务器，当其他全部的非backup机器down或者忙的时候，请求backup机器   
   
使用方法：

```conf
upstream upstreamName{
    server ip1:port1 down;
    server ip2:port2 weight=2;
    server ip3:port3 backup;
}
```

### 分布式系统session的处理方法
① session复制：当某一台服务器的session发生改变时，将session的改变复制到其他所有服务器上（毫无疑问，该方法的性能较低）   
② 请求ip与服务器绑定：客户端的ip与服务器绑定在一起，同一用户的请求都转发到同一台机器上进行处理   
③ 将session信息存放在MySQL等数据库中   
④ 将session存放在Redis、memcached内存数据库中

## 四、Events事件模型、阻塞/非阻塞和同步/异步
Nginx的I/O模型包括标准事件模型、高效事件模型。

### 阻塞/非阻塞与同步/异步
我相信有很大一部分人会把非阻塞I/O和异步I/O混为一谈，因为我在写这篇博客之前也是这么认为的。

#### 阻塞I/O、非阻塞I/O

> 阻塞I/O：应用程序需要等待I/O完成后才返回结果，阻塞I/O造成CPU等待I/O，CPU的能力不能得到充分利用。
> 非阻塞I/O：在调用后直接返回，不等待I/O完成，但之后需要使用轮询或者其他事件模型（select、poll、epoll、kqueue）来判断是否完成I/O，若I/O完成，则执行后续操作。
> 在非阻塞I/O返回后，CPU的时间片可以用来处理其他事务。

#### 同步I/O、异步I/O

> 同步I/O：导致请求进程阻塞，直到I/O操作完成；
> 异步I/O：不导致请求进程阻塞，I/O完成发出通知

#### 关系
阻塞I/O、非阻塞I/O都属于同步I/O，用知乎的一个回答来说明吧：

> *作者：愚抄*
> *链接：https://www.zhihu.com/question/19732473/answer/23434554*
> *来源：知乎*
> 
> 老张爱喝茶，废话不说，煮开水。
> 出场人物：老张，水壶两把（普通水壶，简称水壶；会响的水壶，简称响水壶）。
> 
> 1 老张把水壶放到火上，立等水开。（同步阻塞）
> 老张觉得自己有点傻
> 2 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。（同步非阻塞）
> 老张还是觉得自己有点傻，于是变高端了，买了把会响笛的那种水壶。水开之后，能大声发出嘀~~~~的噪音。
> 3 老张把响水壶放到火上，立等水开。（异步阻塞）
> 老张觉得这样傻等意义不大
> 4 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。（异步非阻塞）
> 老张觉得自己聪明了。
> 
> 所谓同步异步，只是对于水壶而言。
> 普通水壶，同步；响水壶，异步。
> 虽然都能干活，但响水壶可以在自己完工之后，提示老张水开了。这是普通水壶所不能及的。同步只能让调用者去轮询自己（情况2中），造成老张效率的低下。
> 
> 所谓阻塞非阻塞，仅仅对于老张而言。
> 立等的老张，阻塞；看电视的老张，非阻塞。
> 情况1和情况3中老张就是阻塞的，媳妇喊他都不知道。虽然3中响水壶是异步的，可对于立等的老张没有太大的意义。所以一般异步是配合非阻塞使用的，这样才能发挥异步的效用。
> ——来源网络，作者不明。


### 标准事件模型

如果当前系统不存在更高效的方法，则使用标准事件模型select或poll

#### read

read是最原始、最低效的事件模型。其通过重复调用来检查I/O状态，即轮询。

#### select

select是read的改进版，它通过轮询监视文件描述符集合，来获取发生变化的文件描述符（集合中的文件描述符会被内核进行标志位修改）。   
缺点：① 文件描述符的数量具有限制；② 当文件描述符数量增多时，采取轮询的性能会降低；

#### poll

poll的处理机制与select类似，区别在于poll使用链表，而不是数组，避免了长度限制。但文件描述符较多时，性能还是十分低下。

### 高效事件模型
#### Epoll

*使用于Linux内核2.6版本及以后的系统*    
相对于poll，epoll只会处理发生改变的文件描述符，这样避免了不必要的事件处理，提升了性能及效率。

#### Kqueue

*使用于FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 和 MacOS X*   
实现方式与epoll类似，不过它仅在FreeBSD系统下存在。

## 五、Nginx配置文件 + 注释

```conf
# Nginx的用户和组
user www www;

# 工作进程的数目
worker_processes auto;

# 错误日志存放位置，格式为：error_log 文件路径 错误级别（debug | info | notice | warn | error | crit | alert | emerg），级别越高记录的信息越少
error_log /wwwlogs/error_nginx.log crit;

# 进程标识符
pid /var/run/nginx.pid;

# 进程可以打开最大描述符的数目
worker_rlimit_nofile 51200;

events {
  # 使用的I/O模型（epoll、kqueue、select、poll etc.）
  use epoll;

  # 若该项设置为off，当一个请求到来时，所有休眠的worker进程都会被唤醒，来争夺这个请求，但只有一个进程可以处理该请求，其他未获得请求的进程继续进入休眠，这就是惊群现象，会造成极大地资源浪费
  # 该项设置为on后，Nginx规定同一时刻只能有唯一一个持有mutex锁的worker进程会接受并处理请求
  accept_mutex on;

  # worker进程的等待时间，过了等待时间后，下一个worker进程取得mutex锁
  accept_mutex_delay 500ms;

  # 每个工作进程的最大连接数，与前面的
  worker_connections 51200;

  # 让worker进程尽可能多的接受请求
  # 设置为on时，让worker进程一次性地接受监听队列里的所有请求；设置为off则必须一个一个地接受监听队列里的请求
  multi_accept on;
}

# http服务器
http {
  # 包含mime.types文件，该文件为MIME类型映射表（例如图片、文本、视频等）
  # types {
  #  text/html                                        html htm shtml;
  #  text/css                                         css;
  #  text/xml                                         xml;
  #  image/gif                                        gif;
  #  image/jpeg                                       jpeg jpg;
  #  application/javascript                           js;
  #  application/atom+xml                             atom;
  #  application/rss+xml                              rss;
  #  ......
  # }
  include mime.types;

  # 指定默认MIME类型（application/octet-stream为普通的文件流，浏览器会提示用户下载）
  default_type application/octet-stream;

  # 保存服务器名字的Hash表大小（若增加多个域名后无法启动，可能需要将该项适当调大，并保持为处理器缓存大小的倍数）
  server_names_hash_bucket_size 128;
  # 服务器最大个数
  server_names_hash_max_size 512;

  # 客户端请求头部的缓存区大小
  client_header_buffer_size 32k;
  # nginx默认使用client_header_buffer_size来读取header的值，若header过大，则使用large_client_header_buffers来读取
  large_client_header_buffers 4 32k;

  # 客户端请求body的最大值（nginx上传文件的大小）
  client_max_body_size 1024m;
  # 客户端请求body缓冲区大小
  client_body_buffer_size 10m;

  # sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
  sendfile on;
  # 允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
  tcp_nopush on;

  # keep-alive超时时间
  keepalive_timeout 120;

  # 开启或关闭nginx的版本信息 
  server_tokens off;

  # 提高数据的实时响应性，仅在将连接转变为长连接的时候才被启用（在upstream发送响应到客户端时也会启用）
  tcp_nodelay on;

  # fastcgi相关配置
  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 4 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  fastcgi_intercept_errors on;

  # Gzip压缩
  gzip on;
  # 16 8k代表以8k为单位，按照原始数据大小以8k为单位的16倍申请内存
  gzip_buffers 16 8k;
  # 压缩级别，等级越高，消耗CPU越多
  gzip_comp_level 6;
  # 设置支持http协议的最小压缩版本
  gzip_http_version 1.1;
  # 允许压缩页面的最小字节数
  gzip_min_length 256;
  # Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含"Via"的 header头。
  # off - 关闭所有的代理结果数据的压缩
  # expired - 启用压缩，如果header头中包含 "Expires" 头信息
  # no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
  # no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
  # private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
  # no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
  # no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
  # auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
  # any - 无条件启用压缩
  gzip_proxied any;
  # 在http头文件中加个“Vary: Accept-Encoding”，给代理服务器用的，有的浏览器支持压缩，有的不支持所以避免浪费不支持的也压缩，所以根据客户端的HTTP头来判断，是否需要压缩
  gzip_vary on;
  # 压缩的MIME类型
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";

  #If you have a lot of static files to serve through Nginx then caching of the files' metadata (not the actual files' contents) can save some latency.
  open_file_cache max=1000 inactive=20s;
  open_file_cache_valid 30s;
  open_file_cache_min_uses 2;
  open_file_cache_errors on;

  # 隐藏头部信息，格式为：proxy_hide_header 响应头部字段
  proxy_hide_header Vary;
  proxy_hide_header X-Powered-By;

######################## default ############################
   #http
   server {
    listen 80;
    # 服务器名
    server_name www.xxxx.com;
    
    # 重定向
    rewrite ^(.*) https://www.xxxx.com;
    
    # 访问日志
    access_log /www/LOG/access_nginx.log combined;

    # 默认主页
    index index.html index.htm index.php;

    # 错误页面，格式：error_page 错误码 错误页面路径
    #error_page 404 /404.html;
    #error_page 502 /502.html
    
    # http://www.xxxx.com/nginx_status查看nginx状态
    location /nginx_status {
      stub_status on;
      access_log off;
      allow 127.0.0.1; # 允许访问的ip
      deny all;   # 拒绝除allow之外的所有访问
    }

    # 转发PHP请求到php-fpm
    location ~ [^/]\.php(/|$) {
      fastcgi_pass 127.0.0.1:9000; # 转发到本地9000端口，php-fpm默认端口
      #fastcgi_pass unix:/dev/shm/php-cgi.sock;
      fastcgi_index index.php;
      include fastcgi.conf;
    }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico)$ {
      expires 30d; # 规定时间后过期
      access_log off;
    }
    location ~ .*\.(js|css)?$ {
      expires 7d;
      access_log off;
    }
    location ~ /\.ht {
      deny all;
    }

  }

  #https 
  server{
    listen 443;
    server_name www.xxxx.com; 

    root  /www/website;
    index index.html index.htm index.php;
    access_log /www/LOG/access_nginx.log combined;

    # ssl配置
    ssl on;
    ssl_certificate "xxxx.crt";
    ssl_certificate_key "xxxx.key";
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   

    location /nginx_status {
      stub_status on;
      access_log off;
      allow 127.0.0.1;
      deny all;
    }
    location ~ [^/]\.php(/|$) {
      #fastcgi_pass 127.0.0.1:9000;
      fastcgi_pass unix:/dev/shm/php-cgi.sock;
      fastcgi_index index.php;
      include fastcgi.conf;
    }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico)$ {
      expires 30d;
      access_log off;
    } 
    location ~ .*\.(js|css)?$ {
      expires 7d;
      access_log off;
    } 
    location ~ /\.ht {
      deny all;
    } 
  }
########################## vhost #############################
  # 包含其他配置信息
  include vhost/*.conf;
}
```

*Mission Complete!*