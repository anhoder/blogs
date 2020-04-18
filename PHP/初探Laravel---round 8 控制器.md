# 初探Laravel---round 8 控制器

<!-- more -->

*Mission Start!*

Laravel的控制器类通常存放在laravel/app/Http/Controllers目录下。应用程序的控制器类需要继承Laravel的控制器基类（Illuminate/Routing/Controller）。控制器类作为HTTP请求的二次分发控制部分，与路由有着紧密的关系，Laravel通依赖注入的方式降低了这种耦合。

## 控制器路由

前面讲了路由的处理由闭包函数完成，其实也可以将请求分发给控制器的具体函数进行处理。

### 基础控制器路由

基本格式与前面的路由一致，只是将第二个参数从闭包函数换为"控制器类@函数名"形式的字符串。如：

```php
Route::get('user/{name}', 'UserController@index');
```
上述用例中，将user/{name}形式的请求分发给UserController下的index函数进行处理，其中index函数需要一个形参用于接收URL中的name参数。控制器类如：

```php
namespace App\Http\Controllers;

use Illuminate\Routing\Controller as BaseController;

class UserController extends BaseController
{
    public function index($name)
    {
        return 'Hello ' . $name;
    }
}

```

### <span style="color:darkred;">隐式控制器路由（在Laravel 5.3中被弃用）</span>

基础控制器路由需要对路由逐个定义，效率较低，而隐式控制器路由是一条语句定义多条路由信息。    

隐式控制器路由的具体格式：

```php
Route::controller('路由前缀', '控制器类名' [, 命名路由]);
// 最后一个参数是当需要添加命名路由时，才以关联数组的形式给出。
```

实例：

```php
// 路由
<?php
Route::controller('user', 'UserController');

// 控制器类
<?php
namespace App\Http\Controllers;

use Illuminate\Routing\Controller as BaseController;

class UserController extends BaseController
{
    public function getIndex($name)
    {
        return 'Hello ' . $name;
    }
    
    public function anyTest()
    {
        return 'This is any request Test';
    }
    
    public function postIndex()
    {
    
    }
}
```

以上实例中，user/index/Alan的GET请求会被分发给getIndex方法；user/test的任意请求都会被分发给anyTest方法；user/index的POST请求会分发给postIndex方法。


> <span style="color: red;">注意：</span>
> <span style="color: red;">**① 隐式控制器路由在5.3版本及之后被弃用；**</span>
> <span style="color: red;">**② 路由前缀可以为'/'；**</span>
> <span style="color: red;">**③ 当控制器的方法名为多个单词时（如：getUserIndex），请求地址应该是home-index的形式；**</span>

### RESTful资源控制器路由

定义格式：

```php
Route::resource('根资源标识', '控制器类名');
```

resource方法可以创建多条路由来处理不同的HTTP请求，控制器类的方法与HTTP请求的对应关系如下：


| 请求方法 | 路径 | 行为 | 控制器类处理函数名 |
| --- | --- | --- | --- |
| GET | /home | 索引（首页） | index |
| GET | /home/create | 创建 | create |
| POST | /home | 保存 | store |
| GET | /home/{id} | 显示 | show |
| GET | /home/{id}/edit | 编辑 | edit |
| PUT/PATCH | /home/{id} | 更新 | update |
| DELETE | /home/{id} | 删除 | destroy |

用例：

```php
// 路由文件
<?php
Route::resource('user', 'UserController');

// 控制器类
<?php
namespace App\Http\Controllers;

use Illuminate\Routing\Controller;

class UserController extends Controller
{
    public function index()
    {
        return 'This is index';
    }
    
    public function show($id)
    {
        return 'ID is' . $id;
    }
}
```

上述用例中，当路由为'/'时，GET请求被分发到index方法；'/home/12'时，GET请求被分发到show方法。

*Mission Complete!*