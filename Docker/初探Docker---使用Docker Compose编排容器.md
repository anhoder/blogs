# 初探Docker---使用Docker Compose编排容器

使用Docker Compose将之前使用Docker搭建的PHP开发环境进行编排。

<!-- more -->

## 什么是Docker Compose？

Docker Compose是Docker官方编排项目之一，可以快速部署应用。

其使用一个单独的YAML文件（docker-compose.yml）来定义一组相关联的应用容器为一个项目。

Docker Compose不是必须使用的，你也可以只使用Docker来构建项目。Docker Compose只是提供一种更简单方便的方式来管理应用容器。

## 编排PHP开发环境

```yaml
version: "3"

## 网络
networks:
    frontend:
        driver: bridge
    backend:
        driver: bridge

services:
    ## PHP-FPM
    php-fpm:
        container_name: alan_php-fpm # 容器名
        build: 
            context: ./php-fpm
        volumes: 
            - ${WEBSITES_DIR}:/home/www # 站点目录
            - ${PHP_FPM_LOG_DIR}:/var/log/php-fpm # php-fpm日志
            - ${PHP_FPM_INI}:/usr/local/etc/php # php-fpm配置文件
        ports: 
            - "${PHP_FPM_PORT}:9000" # php-fpm端口
            - "${SWOFT_PORT}:18306"  # 预留swoft端口
        networks:
            - backend 

        
    ## Nginx
    nginx:
        container_name: alan_nginx # 容器名
        build:
            context: ./nginx 
        volumes: 
            - ${WEBSITES_DIR}:/home/www # 站点目录
            - ${NGINX_LOG_DIR}:/var/log/nginx # Nginx日志
            - ${NGINX_CONF}:/etc/nginx/conf.d # Nginx配置文件
        ports: 
            - "${NGINX_PORT}:80" # Nginx端口
        depends_on:
            - php-fpm # 依赖与php-fpm
        networks:
            - frontend
            - backend

    
    ## MySQL
    mysql:
        container_name: alan_mysql # 容器名
        build: 
            context: ./mysql
        volumes: 
            - ${MYSQL_DATA}:/var/lib/mysql # MySQL数据存储位置
            - ${MYSQL_CONF}:/etc/mysql/conf.d # MySQL配置文件
        ports: 
            - "${MYSQL_PORT}:3306" # MySQL端口
        environment:
            MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD} # MySQL的root密码
            MYSQL_USER: ${MYSQL_USER} # MySQL用户
            MYSQL_PASSWORD: ${MYSQL_PASSWORD} # MySQL密码
            MYSQL_ALLOW_EMPTY_PASSWORD: ${MYSQL_ALLOW_EMPTY_PASSWORD} # 是否允许空密码
        networks:
            - backend

    
    ## Redis
    redis:
        container_name: alan_redis #容器名
        build: 
            context: ./redis
        ports: 
            - "${REDIS_PORT}:6379" # Redis端口
        networks:
            - backend

```

其中的变量（例如：${REDIS_PORT}）可以在.env文件中进行自定义。

## ENV环境变量文件示例

.env配置文件的样例：

```env
# web站点目录
WEBSITES_DIR=./www


## PHP-FPM
# PHP-FPM日志
PHP_FPM_LOG_DIR=./logs/php-fpm

# PHP-FPM配置
PHP_FPM_INI=./php-fpm/php

# PHP-FPM端口
PHP_FPM_PORT=9000

# SWOFT端口
SWOFT_PORT=18306


## Nginx
# Nginx日志
NGINX_LOG_DIR=./logs/nginx

# Nginx配置
NGINX_CONF=./nginx/sites

# Nginx端口
NGINX_PORT=8081


## MySQL
# MySQL数据
MYSQL_DATA=./data

# MySQL配置
MYSQL_CONF=./mysql/conf.d

# MySQL端口
MYSQL_PORT=3306


## Redis
# Redis端口
REDIS_PORT=6379
```

