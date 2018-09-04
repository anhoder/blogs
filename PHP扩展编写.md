# PHP扩展编写

*Mission Start!*

PHP的扩展分为两类：PHP扩展、Zend扩展，在内核中分别称为module、extension，以下的主要是关于module的内容。    
    
    
扩展可以在编译PHP时一起编译（静态编译），也可以将扩展编译为共享库，然后在配置文件php.ini中加入进去。

## 加载PHP扩展的过程

[PHP生命周期](https://blog.alan123.xyz/php/592.html)有5个阶段：模块初始化阶段（php_module_startup）、请求初始化阶段（php_request_startup）、脚本执行阶段（script execute）、请求关闭阶段（php_request_shutdown()）、模块关闭阶段（php_module_shutdown()）。    
   
   
> ① PHP扩展会在模块初始化阶段解析php.ini配置文件，将相对应的扩展共享库加载到PHP中，保存在全局变量extension_lists中；    
> ② 在php_ini_register_extensions()中依次遍历php_extension_lists.engine、php_extension_lists.functions，然后分别调用php_load_zend_extension_cb()、php_load_php_extension_cb()完成Zend扩展、PHP扩展的注册加载。

## 扩展的组成

① 通过zend_module_entry结构体来定义扩展的相关信息，包括扩展名称、版本号、hook函数等。且zend_module_entry结构体声明的变量名称需要遵循modulename_module_entry的格式。    
    
    
> * **name**: 扩展名称
> * **functions**: 扩展定义的内部函数entry
> * **module_startup_func**: 在模块初始化阶段回调的hook函数
> * **module_shutdown_func**: 在模块关闭阶段回调的hook函数
> * **request_startup_func**: 在请求初始化阶段回调的hook函数
> * **request_shutdown_func**: 在请求关闭阶段回调的hook函数
> * **info_func**: php_info()函数时调用
> * **version**: 扩展的版本信息
    
    
② 定义宏ZEND_GET_MODULE(extension_name)以获取扩展的zend_module_entry结构地址：    

```c
#define ZEND_GET_MODULE(name) \
    BEGIN_EXTERN_C() \
    ZEND_DLEXPORT zend_module_entry *get_module(void) { return &name##_module_entry; } \
    END_EXTERN_C()
```

## 开发PHP扩展的步骤

### 1、ext_skel生成扩展基本框架
    
通过ext_skel脚本生成扩展的基本框架，ext_skel在PHP源码的ext目录下，使用方法：    
    
```sh
./ext_skel --extname=extension_name
```
    
生成的目录框架如下：    
    
```sh
$ ./ext_skel --extname=TestExtension

TestExtension
├── CREDITS
├── EXPERIMENTAL
├── TestExtension.c     // 扩展源码
├── TestExtension.php   // 用于在PHP中测试扩展是否可用，可去除
├── config.m4           // 扩展编译时的配置文件，用于phpize脚本生成configure文件
├── config.w32          // windows环境的配置
├── php_TestExtension.h // 头文件
└── tests               // 测试用例，用于make test的测试
    └── 001.phpt
```

### 2、修改config.m4配置
    
Todo...
    
### 3、编写扩展的功能
    
在TestExtension.c文件中编写扩展实现的具体功能。
    
## 编译安装扩展的步骤

### 1、使用phpize解析config.m4，生成configure及其他配置文件
    
当代码量较少，结构简单的项目中，我们往往会手动编写Makefile文件，但是在大型项目中编写Makefile是很难的，于是出现了autoconf/automake/autoheader/autolocal等一系列用于生成configure的自动化工具。    
    
    
而phpize就是一款操作这一系列复杂工具的脚本，它会读取并解析config.m4文件，并以此生成configure及其他文件。    
    
    
使用方式：    
    
> 在待编译的扩展目录使用 phpize 即可，如：
> 
> ```
> $ phpize
> Configuring for:
> PHP Api Version:        
> Zend Module Api No:     
> Zend Extension Api No:
> ```

### 2、编译安装
    
使用上一步生成的configure文件检测安装平台的特征：    

```sh
./configure
```
你可以加上一些参数，例如：    


```sh
./configure --with-php-config=php-config_path   
// php-config是在PHP安装时保存PHP安装信息的脚本，指定php-config更好
```
    
最后，使用make、make install命令完成扩展的安装。


*Mission Complete!*




