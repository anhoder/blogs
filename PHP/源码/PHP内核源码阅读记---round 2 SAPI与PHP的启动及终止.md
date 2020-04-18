# PHP内核源码阅读记---round 2 SAPI与PHP的启动及终止

<!-- more -->

*Mission Start!*
## 一、PHP的生命周期
PHP的整个生命周期可以划分为五个阶段，分别为：**模块初始化阶段、请求初始化阶段、执行脚本阶段、请求关闭阶段、模块关闭阶段**。在不同的SAPI下，各阶段的执行情况可能存在差异，比如：在命令行模式下，每次执行一个脚本文件都会完整的经历这五个阶段，而在fastcgi模式下，启动时经历一次模块初始化，处理每个请求时会经历请求初始化、执行脚本、请求关闭三个阶段，在关闭时会经历一次模块关闭。
## 二、SAPI
SAPI, Server Application Programming Interface（服务器应用程序接口），SAPI就是PHP与外部环境的代理器，针对不同的外部环境使用不同的接口。它将外部环境抽象后，为内部的PHP提供一套固定的、统一的接口，使得PHP自身实现能够不受错综复杂的外部环境影响，保持一定独立性。

简单介绍一下PHP几种经典的SAPI：CLI、FPM、Embed。
### 1、CLI
**CLI，Command Line Interface，命令行接口**，可以在命令行模式下执行PHP脚本，如：

```sh
$ php test.php
```

CLI每次执行完请求后就关闭，所以每次执行脚本都会经历了模块初始化、请求初始化、执行脚本、请求关闭、模块关闭。

### 2、FPM
**FPM，FastCGI Process Manager，FastCGI进程管理器**，是PHP FastCGI运行模式下的一个进程管理器。
先了解一下FastCGI，它是CGI（Common Gateway Interface，公共网关接口）的升级版（[CGI与FastCGI的区别](https://www.baidu.com/s?wd=CGI与FastCGI的区别)），与CGI、HTTP一样属于应用层的通信协议。因为PHP本身并没有HTTP网络库，而是通过实现FastCGI协议来处理HTTP请求，其工作方式为：

> HTTP协议 --> Web服务器 --> FastCGI协议 --> PHP
> PHP执行完毕依次返回数据

FPM是一个多进程模型，由一个master进程和多个worker进程组成。master进程负责管理worker进程，worker进程才是真正的请求处理者。
#### master进程
master拥有三种不同的进程管理方式：静态模式、动态模式、按需模式。

> **静态模式**：在启动时master进程fork出一定量的worker进程，固定不变。<span style="color:red;">*（worker进程数量由配置项pm.max_children决定）*</span>
> **动态模式**：常用的模式，在启动时根据pm.start_servers数量fork出相同数量的worker进程。在运行期间，若发现空闲的worker进程数小于配置pm.min_spare_servers的数量（表示请求过多，worker进程处理不过来），master则会继续fork worker进程；相反，若发现空闲的worker进程数大于pm.max_spare_servers，master则会杀掉一些worker进程，以防止占用过多资源。<span style="color:red;">*（注意：在整个过程中，worker进程数不得超过pm.max_children）*</span>
> **按需模式**：在启动时，不分配worker进程，当请求到来时，同时master进程fork worker进程，处理完请求后，worker进程不会立即退出，当空闲时间超过pm.process_idle_timeout后再退出。<span style="color:red;">*（同样的，worker进程数不超过pm.max_children）*</span>

#### worker进程
worker进程周期为：
> 等待请求 --> 解析请求 --> 请求初始化 --> 执行PHP脚本 --> 关闭请求 --> 等待请求

### 3、Embed
编译后是普通的库文件（静态库或共享库），可以在C/C++应用中调用PHP提供的API，甚至提供给其他语言使用。

## 三、PHP的启动及终止
PHP的启动、终止可以分别分为两个概念的启动、终止，一个是作为Apache的一个模块启动与终止（或者PHP-FPM的启动与终止），在这次的启动中PHP会初始化一些必要的数据，并将这些数据常驻内存；终止则相反。另外一个则是当接收到一个请求时，PHP会有一次启动和终止。  
    
PHP内核中预置了四个与启动、终止有关的宏函数：PHP_MINIT_FUNCTION、PHP_MSHUTDOWN_FUNCTION、PHP_RINIT_FUNCTION、PHP_RSHUTDOWN_FUNCTION。   
   
### 1、PHP_MINIT_FUNCTION
当PHP最初的初始化时，会把自己所有已加载扩展的PHP_MINIT_FUNCTION方法都执行一遍。在PHP_MINIT_FUNCTION中定义的内容都会常驻内存，可以被所有请求使用。《PHP扩展开发与内核应用》中的栗子：   
   
```c
//walu是我扩展的名称
int time_of_minit;//在MINIT()中初始化，在每次页面请求中输出，看看是否变化
PHP_MINIT_FUNCTION(walu)
{
    time_of_minit=time(NULL);//我们在MINIT启动中对他初始化
    return SUCCESS;//返回SUCCESS代表正常，返回FALIURE就不会加载这个扩展了。
}
```    
    
### 2、PHP_RINIT_FUNCTION
当PHP接收到一个请求时，PHP会迅速的开辟一个新的环境，并重新扫描自己的各个扩展，遍历执行它们各自的PHP_RINIT_FUNCTION方法。栗子：    
    
```c
int time_of_rinit;//在RINIT里初始化，看看每次页面请求的时候是否变化。
PHP_RINIT_FUNCTION(walu)
{
	time_of_rinit=time(NULL);
	return SUCCESS;
}
```    
    
### 3、PHP_RSHUTDOWN_FUNCTION
当接收的请求结束时，PHP会启动回收程序，执行所有已加载扩展的PHP_RSHUTDOWN_FUNCTION方法，释放这次请求使用过的所有东西。栗子：
    
```c
PHP_RSHUTDOWN_FUNCTION(walu)
{
	FILE *fp=fopen("time_rshutdown.txt","a+");
	fprintf(fp,"%ld\n",time(NULL));//让我们看看是不是每次请求结束都会在这个文件里追加数据
	fclose(fp);
	return SUCCESS;
}
```
    
### 4、PHP_MSHUTDOWN_FUNCTION
当Apache（或者PHP-FPM）需要停止时，PHP便会执行所有扩展的PHP_MSHUTDOWN_FUNCTION方法，一旦PHP将所有这些方法执行完，便会进入自毁程序（一定要将擅自申请的内存释放掉）。栗子：   
   
```c
PHP_MSHUTDOWN_FUNCTION(walu)
{
	FILE *fp=fopen("time_mshutdown.txt","a+");
	fprintf(fp,"%ld\n",time(NULL));
	return SUCCESS;
}
```
    
    
    
前面提到的四个栗子中的宏都是在wuli.c中实现的。   
将以上栗子组合在一起进行测试，其结果为：   

* time_of_minit的值每次请求都不变；
* time_of_rinit的值每次请求都改变；
* 每次页面请求结束都会往time_rshutdown.txt中写入数据；
* 只有在apache结束后time_mshutdown.txt才写入有数据。   
    
可见，四个宏函数执行的时机和所描述一致。    
    
*Mission Complete!*


