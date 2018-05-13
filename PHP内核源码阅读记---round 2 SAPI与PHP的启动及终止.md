# PHP内核源码阅读记---round 2 SAPI与PHP的启动及终止

*Mission Start!*

## 一、SAPI
SAPI, Server Application Programming Interface（服务器应用程序接口），SAPI就是PHP与外部环境的代理器，它将外部环境抽象后，为内部的PHP提供一套固定的、统一的接口，使得PHP自身实现能够不受错综复杂的外部环境影响，保持一定独立性。

## 二、PHP的启动及终止
PHP的启动、终止可以分别分为两个概念的启动、终止，一个是作为Apache的一个模块启动与终止（或者PHP-FPM的启动与终止），在这次的启动中PHP会初始化一些必要的数据，并将这些数据常驻内存；终止则相反。另外一个则是当接收到一个请求时，PHP会有一次启动和终止。（按我的理解应该是master进程和work进程的区别）    
    
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


