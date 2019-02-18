# Lumen框架下安装dingo/api和tymon/jwt-auth

*Mission Start!*

## 一、安装Lumen框架

使用composer命令基于Lumen框架创建项目：

```sh
composer create-project laravel/lumen:"5.6.*" Lumen-Project --prefer-dist
```

在.env文件中配置32位的APP_KEY以及数据库的相关配置。

## 二、安装dingo/api(https://github.com/dingo/api)

① 使用composer命令安装dingo/api：

```sh
composer require dingo/api:"2.0.0-alpha2"
```

② 配置dingo/api：

在.env文件中配置：

```ini
# dingo api
API_NAME=Lumen
API_STANDARDS_TREE=x
API_PREFIX=api
API_STRICT=false
API_DEBUG=true
API_VERSION=v1
API_SUBTYPE=lumen
```

③ 注册dingo/api服务到Lumen：

在App\Providers\AppServiceProvider类的register方法中添加以下代码：

```php
$this->app->register(\Dingo\Api\Provider\LumenServiceProvider);
```

④ 创建dingo/api的配置文件config/api.php

> Lumen框架不自带config目录，需要自行在项目根目录下创建。

api.php可以在vendor/dingo/api/config目录下找到，同理，Lumen框架的配置文件也可在Lumen框架源码中找到。

⑤ 完成以上步骤，可以在routes/web.php中添加以下代码测试是否安装成功：

```php
$api = $app['Dingo\Api\Routing\Router'];
$api->version('v1', function ($api) {
    $api->get('/', function () {
        return [
            'status' => '200',
            'msg' => '成功',
            'data' => [
                'name' => 'Alan',
                'age' => 18
            ],
        ];
    });
})；
```

## 三、安装tymon/jwt-auth

① 使用composer安装：

```sh
composer require tymon/jwt-auth:"1.0.0-rc.2"
```

> 注意：jwt-auth稳定版为0.5，其对lumen高版本不太兼容，所以采用jwt-auth 1.0版

② 修改bootstrap/app.php文件：

去除以下行的注释：

```php
$app->withFacades();
$app->withEloqent();

$app->routeMiddleware([
    'auth' => App\Http\Middleware\Authenticate::class,
]);

$app->register(App\Providers\AppServiceProvider::class);
$app->register(App\Providers\AuthServiceProvider::class);
```

③ 修改app/Providers/AuthServiceProvider.php文件：

在AuthServiceProvider类中的register方法里添加以下代码：

```php
$this->app
->register(\Dingo\Api\Provider\LumenServiceProvider::class);
$this->app['Dingo\Api\Auth\Auth']
->extend('jwt', function () {
    return new \Dingo\Api\Auth\Provider\JWT($this->app['Tymon\JWTAuth\JWTAuth']);
});
```

④ 生成jwt的secret：

```sh
php artisan jwt:secret
```

⑤ 将Lumen框架的配置文件auth.php复制到config/auth.php，并修改内容为：

```php
<?php

return [
    'defaults' => [
        'guard' => env('AUTH_GUARD', 'api'),
        'passwords' => 'users',
    ],
    
    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users'
        ],
    ],

    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model'  => \App\User::class,
        ],
    ],

    'passwords' => [
        //
    ],
];
```

⑥ 修改配置文件，将dingo/api和tymon/jwt-auth关联：

修改dingo/api的配置文件config/api.php:

```php
'auth' => [
    'jwt' => 'Dingo\Api\Auth\Provider\JWT',
],
```

完成以上步骤就完成了在Lumen框架下dingo/api和tymon/jwt-auth的安装。

*Mission Complete!*