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
ZEND_GET_MODULE(extension_name)
```
<br />
在PHP中，ZEND_GET_MODULE宏的定义如下：

```c
#define ZEND_GET_MODULE(name) \
    BEGIN_EXTERN_C() \
    ZEND_DLEXPORT zend_module_entry *get_module(void) { return &name##_module_entry; } \    // ##起到拼接字符串的作用
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
    
[PHP autoconf m4](https://www.php.net/manual/zh/internals2.buildsys.configunix.php)

* PHP_ARG_WITH() 用于取得参数的选项，例如扩展所需库或程序的位置
* PHP_ARG_ENABLE() 用于代表简单标志的选项

以下为PHP官方文档给出的config.m4样例

```m4
dnl $Id$
dnl config.m4 for extension example
PHP_ARG_WITH(example, for example support,
[  --with-example[=FILE]       Include example support. File is the optional path to example-config])
PHP_ARG_ENABLE(example-debug, whether to enable debugging support in example,
[  --enable-example-debug        example: Enable debugging support in example], no, no)
PHP_ARG_WITH(example-extra, for extra libraries for example,
[  --with-example-extra=DIR      example: Location of extra libraries for example], no, no)

dnl 检测扩展是否已启用
if test "$PHP_EXAMPLE" != "no"; then
  
  dnl 检测 example-config。首先尝试所给出的路径，然后在 $PATH 中寻找
  AC_MSG_CHECKING([for example-config])
  EXAMPLE_CONFIG="example-config"
  if test "$PHP_EXAMPLE" != "yes"; then
    EXAMPLE_PATH=$PHP_EXAMPLE
  else
    EXAMPLE_PATH=`$php_shtool path $EXAMPLE_CONFIG`
  fi
  
  dnl 如果找到可用的 example-config，就使用它（检测文件是否存在，是否可执行以及执行的结果）
  if test -f "$EXAMPLE_PATH" && test -x "$EXAMPLE_PATH" && $EXAMPLE_PATH --version > /dev/null 2>&1; then
    AC_MSG_RESULT([$EXAMPLE_PATH]) dnl 结束check，继续执行
    EXAMPLE_LIB_NAME=`$EXAMPLE_PATH --libname`
    EXAMPLE_INCDIRS=`$EXAMPLE_PATH --incdirs`
    EXAMPLE_LIBS=`$EXAMPLE_PATH --libs`
    
    dnl 检测扩展库是否工作正常
    dnl PHP_CHECK_LIBRARY() 尝试编译、链接和执行程序，
    dnl 在第一个参数指定的库中调用由第二个参数指定的符号，使用第五个参数给出的字符串作为额外的链接选项。
    dnl 如果尝试成功了，则运行第三个参数所给出的脚本。此脚本从 example-config 所提供的原始的选项字符串中取出头文件路径、库文件路径和库名称，告诉 PHP 构建系统。
    dnl 如果尝试失败，脚本则运行第四个参数中的脚本。此时调用 AC_MSG_ERROR() 来中断程序执行。
    PHP_CHECK_LIBRARY($EXAMPLE_LIB_NAME, example_critical_function,
    [
      dnl 添加所需的 include 目录
      PHP_EVAL_INCLINE($EXAMPLE_INCDIRS)
      dnl 添加所需的扩展库及扩展库所在目录
      PHP_EVAL_LIBLINE($EXAMPLE_LIBS, EXAMPLE_SHARED_LIBADD)
    ],[
      dnl 跳出
      AC_MSG_ERROR([example library not found. Check config.log for more information.])
    ],[$EXAMPLE_LIBS]
    )
  else
    dnl 没有可用的 example-config，跳出
    AC_MSG_RESULT([not found])   dnl 结束check
    AC_MSG_ERROR([Please check your example installation.]) dnl 报错
  fi
  
  dnl 检测是否启用调试
  if test "$PHP_EXAMPLE_DEBUG" != "no"; then
    dnl 是，则设置 C 语言宏指令
    AC_DEFINE(USE_EXAMPLE_DEBUG,1,[Include debugging support in example])
  fi
  
  dnl 检测额外的支持
  if test "$PHP_EXAMPLE_EXTRA" != "no"; then
    if test "$PHP_EXAMPLE_EXTRA" == "yes"; then
      AC_MSG_ERROR([You must specify a path when using --with-example-extra])
    fi
    
    PHP_CHECK_LIBRARY(example-extra, example_critical_extra_function,
    [
      dnl 添加所需路径
      PHP_ADD_INCLUDE($PHP_EXAMPLE_EXTRA/include)
      PHP_ADD_LIBRARY_WITH_PATH(example-extra, $PHP_EXAMPLE_EXTRA/lib, EXAMPLE_SHARED_LIBADD)
      AC_DEFINE(HAVE_EXAMPLEEXTRALIB,1,[Whether example-extra support is present and requested])
      EXAMPLE_SOURCES="$EXAMPLE_SOURCES example_extra.c"
    ],[
      AC_MSG_ERROR([example-extra lib not found. See config.log for more information.])
    ],[-L$PHP_EXAMPLE_EXTRA/lib]
    )
  fi
  
  dnl 最后，将扩展及其所需文件等信息传给构建系统
  dnl PHP_NEW_EXTENSION
  dnl 第一个参数是扩展的名称，和包含它的目录同名。
  dnl 第二个参数是做为扩展的一部分的所有源文件的列表。参见 PHP_ADD_BUILD_DIR() 以获取将在子目录中源文件添加到构建过程的相关信息。
  dnl 第三个参数总是 $ext_shared， 当为了 --with-example[=FILE] 而调用 PHP_ARG_WITH()时，由 configure 决定参数的值。
  dnl 第四个参数指定一个“SAPI 类”，仅用于专门需要 CGI 或 CLI SAPI 的扩展。其他情况下应留空。
  dnl 第五个参数指定了构建时要加入 CFLAGS 的标志列表。
  dnl 第六个参数是一个布尔值，为 "yes" 时会强迫整个扩展使用 $CXX 代替 $CC 来构建。
  dnl 第三个以后的所有参数都是可选的。
  PHP_NEW_EXTENSION(example, example.c $EXAMPLE_SOURCES, $ext_shared)
  dnl PHP_SUBST() 来启用扩展的共享构建
  PHP_SUBST(EXAMPLE_SHARED_LIBADD)
fi

```
    
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




