# 初探Docker---Docker及Dockerfile

学习Docker过程中自己的一些理解。

* 什么是Docker、Dockerfile？
* 怎么使用Docker？
* 如何编写Dockerfile？

<!-- more -->

## 1. 镜像

镜像是一种特殊的文件系统，包含所需要的程序、资源、库、配置等文件，还有一些为运行时准备的配置参数。Docker镜像并不是一个文件，而是由多层文件系统联合组成的。镜像是一层一层构建的，而且删除前一层文件的操作并不是真的删除文件，而是将其标记为该文件已删除。

## 2. 容器

容器是使用镜像来创建的，其类似于`类`与`对象`的关系：

* 镜像是`类`，是静态的定义；
* 容器是实例化的`对象`，是动态运行时的实体。

## 3. 仓库

**仓库（Repository）**是一个集中存储、分发镜像的场所，例如：存放Ubuntu镜像的仓库是Ubuntu仓库。另一个概念是**注册服务器（Registry）**，注册服务器是存放仓库的地方。

## 4. docker命令列表

* `docker help`: 获取帮助信息
* `docker exec`: 在正在运行的容器中运行命令
* `docker pull`: 从服务器提取镜像或存储库
* `docker ps`: 列出容器信息
* `docker images`: 列出所有镜像
* `docker build`: 通过Dockerfile构建镜像
* `docker create IMAGE`: 创建一个新的容器但不启动
* `docker run IMAGE [COMMAND]`: 创建一个新容器并运行一个命令
* `docker restart CONTAINER`: 重启容器
* `docker start CONTAINER`: 启动一个或多个容器
* `docker stop CONTAINER`: 停止容器
* `docker top CONTAINER`: 查看容器中运行的进程信息
* `docker rm`: 删除容器
* `docker rmi`: 删除镜像
* `docker volume`: 数据卷管理
* `docker network`: 网络管理
* `docker commit`: 从当前更改的容器状态创建新镜像
* `docker search`: 在服务器中搜索镜像
* `docker version`: 查看docker版本信息
* `docker attach`: 将本地输入、输出、错误流附加到正在运行的容器
* `docker history`: 查看镜像历史记录
* `docker info`: 显示设备信息
* `docker inspect`: 获取容器/镜像的元数据
* `docker login`: 登录到服务器
* `docker pause CONTAINER`: 暂停容器中所有的进程
* `docker unpause CONTAINER`: 恢复容器中的所有进程
* `docker kill CONTAINER`: 杀掉一个运行中的容器

## 5. Dockerfile

Dockerfile中提供了很多指令，以下将会提到这些指令的作用和用法。

### `FROM`

* 指定**基础镜像**。如: nginx, ubunu等。

* 还可以使用空白镜像(`scratch`)作为基础镜像。

> 使用：`FROM nginx`，或指定版本`FROM ubuntu:stretch`

### `RUN`

* 运行命令。

* 可以运行可执行文件：`RUN ["可执行文件", "参数1", "参数2"]`

> 使用：`RUN echo '测试' > a.txt`

### `COPY`

* 复制文件。

* 可以使用通配符：`COPY *.txt /var/www/`

* 可以指定复制后文件的所属用户及组：`COPY --chown=name a.txt /var/www/`

> 使用：`COPY a.txt /var/www/`

### `ADD`

* 高级的`COPY`指令，可以指定URL。从URL下载的文件默认权限为600。

* 该命令可以用`RUN`指令执行`wget`或`curl`替换

**不推荐使用**

### `CMD`

* Docker容器启动时执行。

* 与RUN指令一样，该指令也支持exec、shell两种写法：`CMD <命令>`、 `CMD ["可执行文件", "参数1", "参数2"...]`

* 一个Dockerfile只能有一个CMD命令。

> 使用: `CMD ["curl", "-s", "https://dogedoge.com"]`

### `ENTRYPOINT`

* 与`CMD`指令一样，用于指定容器启动程序及参数。

* 同样支持exec及shell写法

* 与CMD指令的区别在于：command会作为参数传递给ENTRYPOINT指令

> 使用：`ENTRYPOINT ["curl", "-s", "https://dogedoge.com"]`

### `ENV`

* 设置环境变量
 
> 使用：`ENV <key> <value>` 或 `ENV <key1>=<value1> <key2>=<value2> ...`

### `ARG`

* 与ENV指令效果一样，都是设置环境变量。但ARG设置的环境变量在容器运行时是不会存在的

> 使用：`ARG name=Alan`

### `VOLUME`

* 挂载匿名卷(volume)。为了防止用户在运行时忘记将动态文件所在的文件挂载为卷(docker -v)，我们可以事先在Dockerfile中指定挂载某些目录为匿名卷。
 
* Docker运行时应保持容器存储层不发生写操作，容器存储层的数据在容器停止后不会持续存在。所以将数据写入到卷中，可以持久化存储数据，提高性能。

> 使用：`VOLUME /var/data`

### `EXPOSE`

* 声明打算使用的端口。

* 与`docker -p <宿主端口>:<容器端口>`不同，`EXPOSE`只是声明打算使用的端口，并不会自动做端口映射
> 使用：`EXPOSE 8080`

### `WORKDIR`

* 指定当前工作目录。

> 使用：`WORKDIR /var/data`

### `USER`

* 指定当前用户，`USER <用户名>:<用户组>`

> 使用：`USER php`

### `HEALTHCHECK`

* 检查容器的健康状况

> 使用：`HEALTHCHECK [选项] CMD [命令]` 或 `HEALTHCHECK NONE`

### `ONBUILD`

* 当该镜像作为基础镜像被其他镜像使用时才会被执行。

> 使用：`ONBUILD RUN ["npm", "install"]`


---


*下一篇：[初探Docker---编写Dockerfile搭建PHP的开发环境](#)*