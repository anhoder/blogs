# 初探Laravel---round 6 补充：Laravel目录结构

<!-- more -->

*Mission Start!*

打开Laravel框架，其目录文件如下图：

![目录结构](https://media.alan123.xyz/imgs/blogs/laravel/1.jpg)

目录结构介绍如下表：


| 文件夹名称 | 简介 |
| --- | --- |
| app | 应用程序的业务逻辑代码存放文件夹 |
| app/Console | 存放自定义artisan命令文件 |
| app/Events | 用于放置与事件相关的类 |
| app/Exceptions | 包含应用程序的异常处理类，用于处理应用程序抛出的任何异常 |
| app/Http | 主要包含路由文件、控制器文件、请求文件、中间件文件等，是应用程序与Laravel框架源代码等外部库交互的主要地方 |
| app/Http/Controllers | 存放控制器文件 |
| app/Http/Middleware | 存放中间件文件 |
| app/Jobs | 主要包含消息队列的各种消息类文件 |
| app/Listeners | 主要包含监听事件类文件 |
| app/Providers | 主要包含服务提供者的相关文件 |
| bootstrap | 框架启动与自动加载设置相关文件 |
| config | 应用程序的配置文件 |
| database | 数据库操作相关文件（数据库迁移和数据库填充） |
| node_modules | 存放NPM依赖模块 |
| public | 前端控制器和资源相关文件 |
| resources | 应用资源 |
| resources/assets | 未编译前的应用资源文件 |
| resources/lang | 多语言文件 |
| resources/views | 视图文件 |
| routes | 路由目录 |
| routes/api.php | 用于定义API类型的路由 |
| routes/channels.php | 时间转播注册信息 |
| routes/console.php | 用于定义Artisan命令 |
| routes/web.php | 用于定义Web类型的路由 |
| storage | 编译后的视图、基于会话、文件缓存和其他框架生成的文件 |
| storage/app | 目录可用于存储应用程序使用的任何文件 |
| storage/framework | 目录被用于保存框架生成的文件及缓存 |
| storage/logs | 应用程序的日志文件 |
| tests | 应用测试相关文件 |
| vendor | Composer依赖模块 |
| composer.json | 应用依赖的扩展包  |
| composer.lock | 扩展包列表，确保这个应用的副本使用相同版本的扩展包 |
| package.json | 应用所需的NPM包配置文件 |
| phpunit.xml | 测试工具PHPUnit的配置文件 |
| server.php | 使用PHP内置服务器时的URL重写 |
| webpack.mix.js | Laravel的前端工作流配置文件 |
| .env | 环境变量配置文件 |
| .gitignore | 配置被Git所忽略的文件 |

*Mission Complete!*


