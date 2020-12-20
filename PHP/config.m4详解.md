# config.m4详解

config.m4是扩展的编译配置文件，其会被包含到configure.in中，最终被autoconf编译为configure。

<!-- more -->

## PHP提供的宏

### PHP_ARG_WITH(arg_name, check_message, help_info)

* 定义一个`--with-param[=arg]`的编译参数。
* 这个宏有五个参数，但常用的是前三个。第一个参数为`参数名`，第二个参数为`执行./configure时的展示信息`，第三个参数为`执行 --help 时的展示信息`。
* 该宏定义的参数可以在config.m4中通过`$PHP_大写参数名`的形式进行访问。

例如：

```php
PHP_ARG_WITH(extname, for extname support, [--with-extname=yes])
# 通过$PHP_EXTNAME即可获得--with-extname=yes中的yes
```

### PHP_ARG_ENABLE(arg_name, check_message, help_info)

* 定义一个`--enable-param[=arg]`或`--disable-param`（等价于`--enable-param=no`）
* 该宏与`PHP_ARG_WITH`类似，如果参数需要设置具体的值，则使用`PHP_ARG_WITH`，否则使用`PHP_ARG_ENABLE`

### AC_MSG_CHECKING、AC_MSG_RESULT、AC_MSG_ERROR

* AC_MSG_CHECKING([提示信息]): 在执行./configure时，check的提示信息。
* AC_MSH_RESULT([检测的路径]): 结束check，继续执行
* AC_MSG_ERROR([错误信息]): 输出错误信息并退出./configure的执行

### AC_DEFINE(variable, value, [description])

* 定义一个宏，例如：`AC_DEFINE(DEBUG_MODEL, 1, [open debug model])`，最终会执行`#define DEBUG_MODEL 1`

### PHP_ADD_INCLUDE(path)

* 添加include路径，相当于`#include "file"`
* 引用外部库或分为多个目录时会使用到

### PHP_CHECK_LIBRARY(library, function [, action-found [, action-not-found [, extra-libs]]])

* 检查依赖库library中是否存在需要的function，action-found为存在时执行的动作，action-not-found是不存在时执行的动作

例如：

```php
PHP_CHECK_LIBRARY($EXAMPLE_LIB_NAME, example_critical_function, [
  dnl 添加所需的 include 目录
  PHP_EVAL_INCLINE($EXAMPLE_INCDIRS)
  dnl 添加所需的扩展库及扩展库所在目录
  PHP_EVAL_LIBLINE($EXAMPLE_LIBS, EXAMPLE_SHARED_LIBADD)
],[
  dnl 跳出
  AC_MSG_ERROR([example library not found. Check config.log for more information.])
],[$EXAMPLE_LIBS])
```

### AC_CHECK_FUNC(function, [action-if-found], [action-if-not-found])

* 检查函数function是否存在，action-if-found为存在时执行的动作，action-if-not-found为不存在时执行的动作

### PHP_ADD_LIBRARY_WITH_PATH

* 格式为：`PHP_ADD_LIBRARY_WITH_PATH($LIBNAME, $XXX_DIR/$PHP_LIBDIR, XXX_SHARED_LIBADD)`，添加链接库

例如：

```php
PHP_ADD_LIBRARY_WITH_PATH(example-extra, $PHP_EXAMPLE_EXTRA/lib, EXAMPLE_SHARED_LIBADD)
```

### PHP_ADD_BUILD_DIR

* 加载所需所有C文件

### PHP_REQUIRE_CXX()

* 使用C++进行开发


### PHP_SUBST(XXX_SHARED_LIBADD)

* 扩展编译成动态链接库的形式

### PHP_NEW_EXTENSION(extname, sources , $ext_shared [, sapi_class [, extra-cflags [, cxx [, zend_ext]]]])

* 注册一个扩展
* 第一个参数是扩展的名称，和包含它的目录同名
* 第二个参数是做为扩展的一部分的所有源文件的列表
* 第三个参数总是 $ext_shared
* 第四个参数指定一个“SAPI 类”，仅用于专门需要 CGI 或 CLI SAPI 的扩展
* 第五个参数指定了构建时要加入 CFLAGS 的标志列表
* 第六个参数是一个布尔值，为 "yes" 时会强迫整个扩展使用 `$CXX` 代替 `$CC` 来构建
* 第三个以后的所有参数都是可选的

例如：

```php
PHP_NEW_EXTENSION(example, example.c $EXAMPLE_SOURCES, $ext_shared)
```


