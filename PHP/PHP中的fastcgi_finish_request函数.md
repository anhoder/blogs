# PHP中的fastcgi_finish_request函数

fastcgi_finish_request是**工作在FastCGI模式下**才可用的函数。使用这个脚本可以提高请求响应速度。

<!-- more -->

## CGI

CGI，公共网关接口，是一种Web服务器与后端语言的通信规范。CGI会在请求到来时，fork出一个子进程用于处理请求，当处理完成后，会杀死该进程，下次请求到来时继续创建，如此往复...

因此，CGI在创建进程上会存在极大地资源消耗。现在使用较少。

## FastCGI

FastCGI可以说是CGI的升级版本，其解决了CGI在创建进程上的资源消耗。

具体方法为，当FastCGI进程管理器启动时，会创建若干个CGI进程（数量可配置）等待请求到来，不需要每次都花时间去创建，同时可以根据请求数对CGI进程数进行伸缩。

## fastcgi_finish_request

该函数就是**工作在FastCGI模式下的**。具体作用为，当执行到该函数时，直接向客户端返回响应，但后续脚本仍然执行，只是对用户不可见。

例如脚本文件a.php：

```php
echo 'Hello, Alan';

fastcgi_finish_request();

sleep(5);
file_put_contents('a.txt', '测试内容');
```

同时，使用tail命令监控a.txt文件的内容：

```sh
$ tail -f -n 5 a.txt
```

启动Nginx、PHP-FPM，并访问a.php，可以发现很快就能得到响应，并不会受后面`sleep(5)`的影响，5秒之后a.txt文件也会出现`测试内容`。

## 其他

因为该函数**只能在FastCGI模式下可用**，因此，如果需要考虑可移植性，可以自己根据业务逻辑实现fastcgi_finish_request函数：


```php
if (!function_exists('fastcgi_finish_request')) {
  function fastcgi_finish_request() {
  
  }
}
```