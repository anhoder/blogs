# Nginx配置SSL证书

<!-- more -->

<h3>第一步 获取证书文件</h3>

申请SSL证书后，下载证书文件：

① example.crt （公钥）

② example.key  （私钥）

<h3>第二步 上传至服务器</h3>

将两个证书文件上传到服务器的Nginx配置文件夹下，例如/usr/local/nginx/conf/

<h3>第三步 修改Nginx配置信息</h3>

```conf
# HTTPS   
server {  
  
    listen 443;  
    server_name www.xxx.com; # 项目域名  
  
    ssl on;  
    ssl_certificate server.crt; #(证书公钥)  
    ssl_certificate_key server.key; #(证书私钥)  
  
    ssl_session_timeout 5m;  
    ssl_protocols  SSLv2 SSLv3 TLSv1;  
    ssl_ciphers  HIGH:!aNULL:!MD5;  
    ssl_prefer_server_ciphers on;         
  
    location / {  
        proxy_pass      http://111.111.111.111 #服务器地址
    }  
  
}  
```

<h3>第四步 重启Nginx</h3>

重启Nginx后，即可通过https://xxx.xxx.xxx访问网站

<hr />