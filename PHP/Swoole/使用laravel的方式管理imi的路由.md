# 使用laravel的方式管理imi的路由

## 1、imi是什么

imi与Swoft、Hyperf一样，是一个基于Swoole的PHP框架。

**不同之处在于，Swoft与Hyperf是主要针对微服务推出的框架，而imi致力于单体应用。**

此外，imi还支持代码热更新，只要监听的文件发生了更改，imi就会在很短的时间内重启服务。大体实现是：
> 
> ① 在worker进程的启动事件onWorkerStart中，载入相应的PHP文件。
> 
> ② 当监听到代码更新后，向swoole发送一个SIGUSR1信号，swoole会重启所有的worker进程，并触发onWorkerStart事件，重新载入相应的PHP文件。此时，worker进程就是最新的代码了。

虽然这个功能对于线上服务没有很大的帮助，但是却可以大大提高开发时效率。

## 2、imi的路由管理

### 注解

imi路由的管理方式与Swoft框架类似。使用**注解**来管理路由，例如：

```php
<?php
namespace Test;
 
use Imi\Controller\HttpController;
use Imi\Server\Route\Annotation\Route;
use Imi\Server\Route\Annotation\Action;
use Imi\Server\Route\Annotation\Controller;
 
/**
 * @Controller(prefix="home")
 */
class Index extends HttpController
{
    /**
     * @Action
     * @Route(url="/index")
     */
    public function index()
    {
        // 使用/index可以访问到
        return $this->response->write('hello imi!');
    }
    
    /**
     * @Action
     * @Route(url="test")
     */
    public function test()
    {
        // 使用/home/test可以访问到
        return $this->response->write('hello test!');
    }
}
```

上面的代码中，`Index`类上添加了`@Controller`注解，标识该类为一个控制器，并设置路由前缀为`home`；`index`及`test`方法上添加了`Action`、`Route`注解，并分别为它们设置路由为`/home`、`test`。

使用注解管理路由可以很直观地将控制器的方法和路由对应起来，但是它也会带来一些缺陷：
* 当路由越来越多时，路由的管理就变得很复杂。例如：我们需要为一批路由添加一个中间件，此时，就需要修改很多文件。
* 路由结构不够清晰。

### 配置

当然，imi也支持使用配置的方式管理路由：

```php
return [
    'route' => [
        [
            'controller' => \ImiDemo\HttpDemo\MainServer\Controller\Test::class,
            'method' => 'index',
            'route' => [
                'url' => '/test',
                // 'method'    =>    'PUT',
                // 'method'    =>    ['GET', 'POST'],
                // 'domain'    =>    '{name}.xxx.com',
            ],
        ],
     ]
];
```
但是，这种方式存在多重嵌套，可读性并不高。

既然如此，还有什么其他路由管理方式呢？🤔

## 3、imi-route

我认为laravel的路由管理方式是很方便且易读的，例如：

```php
Route::group(['middleware' => ['auth']], function () {
    Route::get('/auth/userinfo', 'AuthController@getUser');
});
```

所以，我编写了一个组件——[`imi-route`](https://github.com/anhoder/imi-route)，希望可以将laravel路由管理的基本功能应用到在imi框架中去。

### 基本实现思路

首先，我们需要知道imi框架在启动时是如何加载相关路由的：

1. 解析应用目录下的所有文件，通过反射获取所有类的注释
2. 注释通过字符串解析后得到相应注解，例如：路由注解为`RouteAnnotation`
3. 将所有的路由注解添加到HttpRoute中

以上的流程都是在服务启动时进行处理，得到路由结果会常驻内存。

而**`imi-route`就是在服务启动时，将存储的路由转化为路由注解，添加到HttpRoute中。**

### 使用

1、使用composer安装依赖：

```sh
composer require alanalbert/imi-route
```
2、将下列代码添加到项目根目录的`Main.php`文件中：
  
```php
<?php
namespace ImiApp;

use Imi\Main\AppBaseMain;
  
class Main extends AppBaseMain
{
  public function __init()
  {
      \Alan\ImiRoute\Route::init(); // Add this line
  }

}
```
3、在项目根目录下，创建`route/route.php`目录及文件，在`route.php`文件中，你就可以管理你的路由了：

```php
/**
 * @var $router Route
 */

use Alan\ImiRoute\Route;
use ImiApp\ApiServer\Controller\IndexController;
use ImiApp\ApiServer\Middleware\Test2Middleware;
use ImiApp\ApiServer\Middleware\TestMiddleware;

$router->group(['middleware' => TestMiddleware::class], function (Route $router) {

    $router->group(
        [
            'middleware' => Test2Middleware::class, 
            'ignoreCase' => true, 
            'prefix' => 'prefix'
        ], function (Route $router) {
        $router->get('hi', 'ImiApp\ApiServer\Controller\IndexController@index');
    });

    $router->any('/hi/api/abc', [IndexController::class, 'index']);

    $router->any('/hi/api/{time}', [IndexController::class, 'api']);

});

$router->group(['prefix' => 'prefix'], function (Route $router) {
    $router->get('/TEST/{time}', [IndexController::class, 'api']);
});
```
