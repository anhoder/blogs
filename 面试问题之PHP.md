# 面试问题之PHP

> 博客内容为[PHP-Interview-QA](https://github.com/colinlet/PHP-Interview-QA)读后笔记

## 变量

* PHP中的八大数据类型：整型、字符串、浮点型、布尔、数组、对象、NULL、资源。
* 字符串的定义：单引号、双引号、NowDoc及HereDoc。双引号包裹的字符串会被解析，单引号的不会。
* 判断语句中被判定为false的情况：0、0.0、'0'、''、array()、null、false

## 引用

* 引用计数、写时复制（对象除外）
* unset不销毁内存空间、只是取消引用

## echo、print、print_r、var_dump、printf、sprintf

> echo、print属于语言结构，print_r、var_dump、printf、sprintf是普通函数

* echo：输出一个或多个字符串（echo '123', '345', '567';）
* print：输出字符串
* print_r：可输出复杂数据类型Array、Object，布尔类型true输出为1，false输出为0
* var_dump：可输出复杂数据类型Array、Object，布尔输出正常
* printf：格式化输出
* sprintf：格式化返回

## isset、is_null、empty

* isset：当变量为NULL、声明后未赋值或者变量未声明时，返回false
* is_null：当变量为NULL、声明后未赋值或者变量未声明时，返回true（当变量未声明或未赋值时，PHP输出警告）
* empty：当变量为空字符串、false、array()、null、"0"、0、0.0或变量未声明时，返回true

## 超全局数组

* $GLOBALS: 包含以下8个
* $_GET: 请求的GET参数
* $_POST: 请求的POST参数
* $_FILES: 提交的文件信息
* $_REQUEST: 请求参数，包含GET及POST的参数
* $_COOKIE: 请求中携带的cookie数据
* $_SESSION: 服务器中的session数据
* $_ENV: 服务器系统的环境变量
* $_SERVER: 服务器脚本信息、客户端信息、请求信息

## 常量

* const：语言结构，可以在类中定义常量，数据更快
* define：函数

> PHP中预定义的常量（魔术常量）：__FILE__、__LINE__、__DIR__、__FUNCTION__、__CLASS__、__TRAIT__、__METHOD__、__NAMESPACE__

## static、self、$this

* static表示当前实例化的类，不能用于访问非静态属性
* self表示当前语句所在的类，也不能用于访问非静态属性，
* $this指的是实际实例化的对象，其不能访问静态属性和常量，且不能出现于静态方法中

## require、require_once、include、include_once

## 常见的数组函数

## 常见的字符串函数



  

