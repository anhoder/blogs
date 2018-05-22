# 初探Laravel---round 1 常用artisan命令【转】

<span style="color:red;">**\[文章转载自CSDN---jiandanokok的专栏，[传送门](https://blog.csdn.net/jiandanokok/article/details/72897682)\]**</span>

*Mission Start!*

artisan是Laravel中的命令行工具，本节记录一些常用的artisan命令，以便后面的Laravel学习。

## 全局篇
1、查看artisan命令

```sh
php artisan 
php artisan list
```

2、查看某个帮助命令

```sh
php artisan help make:model
```

3、查看Laravel版本

```sh
php artisan --version
```

4、使用 PHP 内置的开发服务器启动应用

```sh
php artisan serve
```

5、生成一个随机的 key，并自动更新到 app/config/app.php 的 key 键值对（刚安装好需要做这一步）

```sh
php artisan key:generate
```

6、开启Auth用户功能（开启后需要迁移才生效）

```sh
php artisan make:auth
```

7、开启维护模式和关闭维护模式（显示503）

```sh
php artisan down
php artisan up
```

8、进入tinker工具

```sh
php artisan tinker
```

9、列出所有路由器

```sh
php artisan route:list
```

10、生成路由缓存及移除路由缓存文件

```sh
php artisan route:cache
php artisan route:clear
```

## 功能篇
1、创建控制器

```sh
php artisan make:controller StudentController
```

2、创建Rest风格资源控制器（带index、create、store、edit、update、destroy、show方法）

```sh
php artisan make:controller StudentController --resource
```

3、创建模型

```sh
php artisan make:model Student
```

4、创建新建表的迁移和修改表的迁移

```sh
php artisan make:migration create_users_table --create=users
php artisan make:migration add_votes_to_users_table --table=students
```
*可以试试有无create、table参数的效果区别*

5、执行迁移

```sh
php artisan migrate
```

6、创建模型的时候同时生成新建表的迁移

```sh
php artisan make:model Student -m
```

7、回滚上一次迁移

```sh
php artisan migrate:rollback
```

8、回滚所有迁移

```sh
php artisan migrate:reset
```

9、创建填充

```sh
php artisan make:seeder StudentTableSeeder
```

10、执行单个填充

```sh
php artisan db:seed --class=StudentTableSeeder
```

11、执行所有填充

```sh
php artisan db:seed
```

12、创建中间件（app/Http/Middleware 下）

```php
php artisan make:middleware Activity
```

13、创建队列（数据表）的表迁移（需要执行迁移才生效）

```sh
php artisan queue:table
```

14、创建队列类（app/jobs 下）

```sh
php artisan make:job SendEmail
```

15、创建请求类（app/Http/Requests 下）

```sh
php artisan make:request CreateArticleRequest
```

*Misson Complete!*


