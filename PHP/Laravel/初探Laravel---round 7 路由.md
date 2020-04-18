# 初探Laravel---round 7 路由

<!-- more -->

*Mission Start!*

路由是框架根据URL的不同，将请求提交给指定的控制器或者功能函数来处理。Laravel中，应用程序中的路由基本在laravel/routes目录下。路由文件在应用程序服务提供者启动过程中，通过app\Providers\RouteServiceProvider.php文件中的map方法进行加载。

## 基础路由

在Laravel框架中，基础路由是一个URL对应一个响应程序，这个程序可以是一个闭包函数，也可以是一个控制器响应函数标识。    

以下为路由定义格式：

```php
Route::方法名('URL', 闭包函数或控制器响应函数标识);
```

路由定义中的方法名有：get、post、put、delete等。使用实例：

```php
// 该实例将根URL的GET请求路由到处理函数
Route::get('/', function () {
    return 'Hello World';
});
```

如果需要路由多种请求，可以使用match和any方法：

```php
// 匹配'/'的get、post请求
Route::match(['get', 'post'], '/', function () {
    return 'Hello World';
});

// 匹配'/'的任意请求
Route::any('/', function () {
    return 'Hello World';
});
```

## 路由参数

使用GET请求可以通过URL传递参数。在Laravel中，使用URL传递参数的格式如下：

```php
Route::get('资源标识/{参数[?][/{参数}...]}', 闭包函数或控制响应函数标识)[->where('参数名', '正则表达式')];
```

通过这种形式传递参数，闭包函数或控制器函数可以直接在路由处理函数中添加```$参数名```的形参来接受参数。例如：

```php
// 当http://localhost/user/3的GET请求到来时，输出$id=3
// 该实例的id参数必须添加
Route::get('user/{id}', function ($id) {
    return '$id=' . $id;
});

// 参数名后接?表示该参数可选
Route::get('user/{name?}', function ($name = NULL) {
    return '$name=' . $name;
});

// where用来对输入参数进行限定，只有符合正则表达式要求的参数才能传递。
Route::get('user/{name}', function ($name) {
    return '$name=' . $name;
})->where('name', '[A-Za-z]+');

// 需要匹配多个参数时，使用数组传入
Route::get('user/{id}/{name}', function ($name) {
    return '$name=' . $name . PHP_EOL . 
        '$id=' . $id . PHP_EOL;
})->where(['name' => '[A-Za-z]+', 'id' => '[0-9]+']);
```

> <span style="color:red;">注意：**形参的传入与参数名无关，只与形参的先后顺序有关。**</span>

## 路由命名

路由命名相当于在路由定义的时候，为路由起一个别名，在以后的程序中通过这个别名来获取路由信息。如：

```php
// 为user/name起一个别名name，可以在其他地方使用，如：$url = route('name');获取该URL
Route::get('user/name', ['as' => 'name', 'uses' => function () {
    return 'Hello World';
}]);
```

## 路由群组

当存在许多路由时，可以对路由进行分组，以增加程序的可读性。路由群组便是用来给路由分组的，同时给这个路由组添加前缀、中间件、子域名等。如：

```php
// 存在以下两个路由，它们的前缀都是user，中间件都是auth。
Route::get('user/id', ['middleware' => 'auth', function () {
    return 'Hello ID';
}]);
Route::get('user/name', ['middleware' => 'auth', function () {
    return 'Hello NAME';
}]);

// 此时，可将其定义为一个路由群组
Route::group(['prefix' => 'user', 'middleware' => 'auth'], function () {
    Route::get('id', function () {
        return 'Hello ID';
    });
    Route::get('name', function () {
        return 'Hello NAME';
    });
});
```


*Mission Complete!*

