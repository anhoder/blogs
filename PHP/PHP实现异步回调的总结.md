# PHP实现异步回调的总结

很久没更新博客了，最近在下班时间写了一个简单的PHP库——[php-async](https://github.com/anhoder/php-async)及[php-async-manager](https://github.com/anhoder/php-async-manager)，以实现PHP中的简单异步回调。

<!-- more -->

*Mission Start!*

## 0. 前言

> 最近遇到个问题，PHP的异步回调怎么实现？
 
Workman、Swoole、fastcgi_finish_request？

* Workman、Swoole这些框架虽然对异步回调实现的很好，但是如果只需要简单的异步回调就上这些未免大材小用。
* 而fastcgi_finish_request虽然使用方面，但毕竟只能在php-fpm模式下使用，而且一次请求只能使用一次。

之后就一直在思考：能不能自己实现一个PHP可用的简单异步回调呢？最初出现在我脑海的有几个方案：

1. 编写PHP扩展，使用C/C++创建守护进程进程，用于监听是否有异步任务到来，如果有任务到来，则执行任务及回调函数。而PHP进程则将异步任务传递给该监听进程，从而实现异步。
2. 使用pcntl扩展创建子进程，并让子进程脱离父进程的控制，成为守护进程。子进程用于处理耗时的异步任务，父进程处理HTTP请求及返回响应。

由于第二种方案更简单，因此选择使用第二种方案进行实现。

## 1. 实现原理及思路

实现原理比较简单：

> 1. 在当前进程中创建子进程，父进程处理耗时较短的HTTP业务逻辑；子进程处理耗时较长的任务，例如邮件发送等。
> 2. 创建子进程后，让子进程脱离父进程的控制，成为守护进程。
> 3. 父进程通过设置`Callable`或`Closure`函数，让子进程执行异步任务及回调函数。

每个异步任务都需要两个关键元素组成：异步任务本身、回调函数。所以设计了JobInterface接口，该接口要求实现两个方法：job、callback，分别对应异步任务及回调函数。具体代码为：

```php
interface JobInterface
{
    /**
     * 异步任务
     *
     * @return void
     */
    public function job();

    /**
     * 回调函数
     *
     * @return void
     */
    public function callback();
}
```

## 2. 遇到的问题及解决方法

在CLI模式下开发运行十分顺利，没遇到什么较大的问题。但是，一切开发完成之后，在php-fpm模式下测试，遇到以下几个问题：

### 问题

1. **创建的子进程由php-fpm管理进程控制**，当子进程执行完成后，因为php-fpm没有给该异步进程“收尸”，**导致其成为僵尸进程，占用系统资源**。
2. 因为在php-fpm模式下，进程受php-fpm管理进程的控制，因此猜测**异步进程的数量可能受php-fpm配置的进程数限制**。

### 解决方案

1. 对于“僵尸进程”的问题，让父进程忽略子进程的结束，从而将子进程的回收权移交给系统的init进程，由init进程回收创建的子进程：`pcntl_signal(SIGCHLD, SIG_IGN);`。
2. php-fpm配置的进程数限制，经过测试（将php-fpm配置文件中的最大进程数改为5，并使用静态模式）发现，并未出现猜想中的创建子进程被限制的问题。

## 3. 最后结果

最后实现的结果为: [php-async](https://github.com/anhoder/php-async)。在CLI和php-fpm模式下都没有发现问题。


之后，我还基于symfony/console开发了一个用于查看异步任务的命令行工具——[php-async-manager](https://github.com/anhoder/php-async-manager)，可以用于查看异步任务运行状态、停止正在运行的任务等。

## 4. 后续拓展

该异步实现方案还存在一个很大的问题，即，每执行一个异步任务就要创建一个进程，当处理的请求数量比较多时，就需要创建太多的进程。


之后了解了NodeJS的异步实现原理，其底层使用libuv库，以事件为驱动，libuv负责将来自操作系统的事件收集起来，或者监视其他来源的事件。这样，用户就可以注册回调函数，回调函数会在事件发生的时候被调用。event-loop会一直保持运行状态。


因此，最好的异步实现方案还是使用C语言开发一个PHP的扩展😂。

*Mission Complete!*

