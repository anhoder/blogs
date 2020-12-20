# 初探Docker---编写Dockerfile搭建PHP开发环境

使用Dockerfile搭建PHP开发环境（之后添加docker-compose），项目地址见**原文链接**。

<!-- more -->

## 一、编写Dockerfile

### 1. PHP-FPM

```dockerfile
FROM php:7.2-fpm

LABEL maintainer="AlanAlbert" version="1.0" description="php-fpm(php 7.2)"

# 设置时区
ENV TZ=Asia/Shanghai

# 复制php配置文件、php-fpm配置文件
COPY php.ini /usr/local/etc/php/
COPY php-fpm.conf /usr/local/etc/
COPY php-fpm.d/* /usr/local/etc/php-fpm.d/

# 安装PHP扩展——swoole、redis、pdo_mysql、mysqli
RUN docker-php-ext-install pdo_mysql mysqli \
    && pecl install redis swoole \
    && docker-php-ext-enable redis swoole

# 挂载站点目录、php-fpm日志目录为匿名卷
VOLUME /home/www /var/log/php-fpm

# 添加www用户及切换站点目录所有者
RUN useradd www \
    && chown -R www /home/www

# 切换工作目录为根目录
WORKDIR /

# 容器启动时执行命令
CMD ["/usr/local/sbin/php-fpm"]

# 声明使用9000端口
EXPOSE 9000
```

### 2. Nginx

```dockerfile
FROM nginx

LABEL maintainer="AlanAlbert" version="1.0" description="nginx"

# 设置时区
ENV TZ=Asia/Shanghai

# 复制Nginx配置
COPY nginx.conf /etc/nginx/
COPY sites/*.conf /etc/nginx/conf.d/

# 挂载站点、Nginx日志为匿名卷
VOLUME /home/www /var/log/nginx

# 添加www用户、更换站点目录所有者
RUN useradd www \
    && chown -R www /home/www

# 设置工作目录
WORKDIR /home/www

# 设置容器启动时执行的命令
CMD ["/usr/sbin/nginx"]

# 声明使用的端口
EXPOSE 80 443
```

### 3. MySQL

```dockerfile
FROM mysql:5.7

LABEL maintainer="AlanAlbert" version="1.0" description="mysql"

# 设置环境变量
ENV MYSQL_ALLOW_EMPTY_PASSWORD=true \
    MYSQL_ROOT_PASSWORD="" \
    TZ=Asia/Shanghai

# 复制MySQL配置
COPY my.cnf /etc/mysql/

# 挂载MySQL数据目录为匿名卷
VOLUME /var/lib/mysql

# 声明使用的端口
EXPOSE 3306
```

### 4. Redis

```dockerfile
FROM redis

LABEL maintainer="AlanAlbert" version="1.0" description="redis"

# 设置时区
ENV TZ=Asia/Shanghai

# 声明需要使用的端口
EXPOSE 6379
```

## 二、编写shell脚本管理容器

因为暂时没有使用docker-compose，而docker命令较长，很复杂，所以编写脚本来管理容器

### 1. build

```sh
#!/bin/bash

# 发生错误，中断执行
set -e

echo "开始构建..."
echo "正在构建PHP-FPM..."

# 构建PHP-FPM镜像，并设置名称为alan/php-fpm
docker build -t alan/php-fpm php-fpm

# 创建PHP-FPM容器。
# 1. -p: 将容器9000端口映射到物理机9000端口；
# 2. --name: 指定名称为alan_php-fpm；
# 3. -i: 开启容器标准输入
# 4. -t: 分配一个伪输入终端
# 5. -v: 挂载www目录到/home/www、logs/php-fpm目录到/car/log/php-fpm
# 6. --network: 连接到alan_php_env网络
docker create -p 9000:9000 --name alan_php-fpm -it -v `pwd`/www:/home/www -v `pwd`/logs/php-fpm:/var/log/php-fpm --network alan_php_env alan/php-fpm

echo "PHP-FPM构建完成."
echo "正在构建Nginx..."

# 构建Nginx镜像，设置名称为alan/nginx
docker build -t alan/nginx nginx

# 创建Nginx容器。
# 1. -p: 映射端口
# 2. --name: 设置镜像名称
# 3. -i: 开启标准输入
# 4. -t: 分配伪输入终端
# 5. -v: 挂载目录
# 6. --network: 连接网络
docker create -p 80:80 --name alan_nginx -it -v `pwd`/www:/home/www -v `pwd`/logs/nginx:/var/log/nginx --network alan_php_env alan/nginx

echo "Nginx构建完成."
echo "正在构建MySQL..."

# 构建MySQL镜像
docker build -t alan/mysql mysql

# 创建MySQL容器
docker create -p 3306:3306 --name alan_mysql -it -v `pwd`/data:/var/lib/mysql --network alan_php_env alan/mysql

echo "MySQL构建完成."
echo "正在构建Redis..."

# 构建Redis镜像
docker build -t alan/redis redis

# 创建Redis容器
# --requirepass: 设置Redis密码
docker create -p 6379:6379 --name alan_redis -it --network alan_php_env alan/redis --requirepass ""

echo "Redis构建完成."
echo "正在创建网络..."

# 创建bridge网络
docker network create -d bridge alan_php_env

echo "网络创建完成."
echo "构建完成."
```

### 2. start

```sh
#!/bin/bash
set -e

echo "正在启动..."

# 启动所有容器
docker start alan_php-fpm alan_nginx alan_mysql alan_redis

echo "启动完成."

```

### 3. stop

```sh
#!/bin/bash

echo "正在停止..."

# 停止所有容器
docker stop alan_redis alan_mysql alan_nginx alan_php-fpm

echo "已停止."
```

### 4. restart


```sh
#!/bin/bash
set -e

echo "正在重启..."

# 重启所有镜像
docker restart alan_redis alan_mysql alan_nginx alan_php-fpm

echo "重启完成."
```

### 5. uninstall

```sh
#!/bin/bash

# 停止所有容器 && 删除所有容器 && 删除所有镜像 && 删除网络

echo "正在卸载..."
echo "正在停止运行容器..."

docker stop alan_redis alan_mysql alan_nginx alan_php-fpm

echo "已停止."
echo "正在删除容器..."

docker rm alan_redis alan_mysql alan_nginx alan_php-fpm

echo "已删除容器."
echo "正在删除镜像..."

docker rmi alan/redis alan/mysql alan/nginx alan/php-fpm

echo "已删除镜像."
echo "正在删除网络..."

docker network rm alan_php_env

echo "已删除网络."
echo "卸载完成."
```