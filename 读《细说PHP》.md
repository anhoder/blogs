# 读《细说PHP》

[TOC]

*Mission Start!*

## 一、PHP的错误ERROR与异常EXCEPTION

*注意：在不同的语言中，错误与异常的定义不同。因此以下内容只在PHP中成立。*

### 错误ERROR

错误：属于PHP脚本自身的问题，可能是由错误的语法、服务器环境等导致，使得编译器无法通过检查，甚至无法运行的情况。    

**错误不能被try/catch捕获**    

错误的级别：

 * Fatal Error  致命错误，脚本终止运行
    * E_ERRO: 致命的运行时错误，阻止脚本的执行
    * E_CORE_ERROR: PHP初始化启动过程发生的错误，由PHP引擎核心产生
    * E_COMPILE_ERROR: 致命编译时错误，由Zend脚本引擎产生
    * E_USER_ERROR: 用户产生的错误信息，在代码中使用函数trigger_error()产生（错误类型设置为E_USER_ERROR）
 * Parse Error  编译时解析错误，语法错误，脚本终止运行
    * E_PARSE: 编译时语法解析错误，仅有分析器产生
 * Warning Error  警告错误，脚本不终止运行
    * E_WARNING: 运行时警告，非致命错误
    * E_CORE_WARNING: PHP初始化启动过程中发生的警告，非致命错误，由PHP引擎核心产生
    * E_COMPILE_WARNING: 编译时警告，非致命错误，由Zend脚本产生
    * E_USER_WARNING: 用户产生的警告信息，在代码中使用函数trigger_error()产生（错误类型设置为E_USER_WARNING）
 * Notice Error  通知错误，脚本不终止运行
    * E_NOTICE: 运行时通知，表示脚本遇到可能会表现为错误的情况，但是在可以正常运行的脚本里面也可能会有类似的通知
    * E_USER_NOTICE: 用户产生的通知信息，在代码中使用函数trigger_error()产生（错误类型设置为E_USER_NOTICE）

    另外：
    
    * E_ALL: 除E_ALL外的所有的错误、警告和注意
    * E_STRICT: 关于PHP版本移植的兼容性和互操作性建议

### 异常EXCEPTION

异常：在程序运行期间出现的不符合预期的情况，允许发生但不正常的情况。属于业务逻辑的错误，而不是语法或编译的错误。

### 捕获错误和异常

在PHP中，用于捕获异常和错误的三个重要函数：**error_reporting**、**set_error_handler**、**register_shutdown_function**、**error_get_last**、**set_exception_handler**。

#### error_reporting函数

设置报告哪些错误

```php
int error_reporting([int $err_type])

/**
 * int $err_type 可选参数，需要设置错误等级
 * 返回值：int 返回旧的错误级别
 */
```

用例：

```php
error_report(E_ALL | E_STRICT);
```

#### set_error_handler函数

捕获错误，并执行用户自定义的错误处理函数。

```php
mixed set_error_handler(callable $error_handler 
    [, int $error_types = E_ALL | E_STRICT])
/**
 * callable $error_handler
 *     回调函数，需要声明四个参数：错误类型、错误信息、错误所在文件、错误所在行。
 *     第五个参数可选，指向错误发生时活动符号表的array，但在PHP 7.2.0被废）
 *     
 * int $error_types
 *     可选，规定在哪个错误报告级别显示用户定义的错误
 */
```

注意事项：

> 1. 如果存在该方法，相应的error_reporting()就不能再使用，所有错误都会交给自定义函数处理；
> 2. 此函数只能捕获系统产生的一些Warning、Notice级别的错误，不能捕获：E_ERROR、E_PARSE、E_CORE_ERROR、E_CORE_WARNING、E_COMPILE_WARNING以及set_error_handler()函数所在文件中产生的E_STRICT。

用例：

```php
<?php
    set_error_handler('error_handler');
    function error_handler($err_type, $err_msg, $err_file, $err_line)
    {
        var_dump($err_type);
        var_dump($err_msg);
        var_dump($err_file);
        var_dump($err_line);
    }
    
    // 或者
    
    set_error_handler(array('class_name', 'function_name'));
```

#### register_shutdown_function函数

该函数时PHP脚本执行结束前最后一个调用的函数。通过这个函数就可以在脚本结束前判断是否产生错误。这是需要使用到**error_get_last**函数，获取本次执行产生的所有错误。**register_shutdown_function可以配合set_error_handler捕获所有错误**

```php
void register_shutdown_function(callable $callback
    [, mixed $parameter
    [, mixed $...]])
/**
 * callable $callback 回调函数
 * mixed $parameter 传递给回调的参数
 * ... 更多回调
 */
```

注意事项：

> 1. 该函数可以捕获的错误：Fatal Error、Parse Error等
> 2. register_shutdown_function所在的文件发生语法错误，不能触发回调函数（用例之后会说到）

用例：

```php
<?php
    register_shutdown_function('shutdown_handler');
    function shutdown_handler()
    {
        if ($err = error_get_last()) {
            var_dump($err);
        }
    }
```

但是，如果执行下列测试文件，是不能触发调用回调函数：

```php
<?php
    register_shutdown_function('shutdown_handler');
    function shutdown_handler()
    {
        if ($err = error_get_last()) {
            var_dump($err);
        }
    }
    echo 1++;
```

为什么不能触发呢？这是因为：

> 如果register_shutdown_function所在文件发生语法错误，该脚本不能通过PHP的语法分析，register_shutdown_function函数就不会被执行，也就没有注册回调函数。
> 
> 因此，我们需要保证register_shutdown_function所在文件没有语法错误。

将上述代码修改为：

```php
<?php
    register_shutdown_function('shutdown_handler');
    function shutdown_handler()
    {
        if ($err = error_get_last()) {
            var_dump($err);
        }
    }
    require 'a.php';
    
// a.php文件
<?php
    echo 1++;
```

修改之后，因为register_shutdown_function所在文件没有语法错误，通过了PHP的语法分析，所以register_shutdown_function函数得以正常运行，便能注册回调函数了。    

**因此，最好将register_shutdown_function单独放在一个的文件中，并保证该文件没有语法错误。**

#### error_get_last函数

该函数用于获取本次执行中产生的错误信息。

```php
array error_get_last()
/**
 * 返回值：array 
 * type =>　错误类型
 * message => 错误信息
 * file => 发生错误的文件
 * line => 发生错误的行号
 */
```

#### set_exception_handler函数

设置异常处理函数，如果异常没被捕获就会进到该方法中，并且在回调函数调用后异常会中止。

```php
callable set_exception_handler(callable $exception_handler)

/**
 * callable $exception_handler 回调函数，需要接受一个参数：捕获的异常对象
 * 返回值： callable 返回定义的回调函数的名称
 */
```

**需要注意的是：set_error_handler、register_shutdown_function、set_exception_handler三个函数捕获错误的范围并不是相互独立，可能存在一定的交集，但错误只会被捕获一次，不会重复被捕获**

## 二、日期与时间

* mktime: 获取时间戳，参数依次为时、分、秒、月、日、年，皆可选
* strtotime: 将自然语言英语转换为时间戳
* gettimeofday: 获取某一天中的具体时间
* getdate: 获取一个由时间戳组成的关联数组，可选参数：一个时间戳
* date: 以指定格式返回时间字符串，参数：指定格式，可选参数为时间戳
* date_default_timezone_set: 设置默认时区，RPC、Asia/Shanghai、Asia/Chongqing、Etc/GMT-8
* microtime: 获取当前微秒级时间戳，可选参数：是否以浮点数形式返回

## 三、文件与目录

### 类型

```php
// 获取文件类型
filetype(string $file) : string | false
```

Windows中PHP只能获取到3种文件类型：file、dir、unknown，而在UNIX中，可以获取7种block、char、dir、fifo、file、link、unknown。

* block：块设备文件，如：硬盘分区、软驱、光驱等
* char：字符设备，指I/O传输中以字符为单位进行传输的设备，如键盘、打印机等
* dir：目录类型，目录也是文件的一种
* fifo：命名管道，常用于将信息从一个进程传递到另一个进程
* file：普通文件类型
* link：链接，软链接、硬链接
* unknown：未知类型

### 文件

* is_file、is_dir、is_link...：判断是否给定的文件为某一类型
* file_exists：**文件或目录**是否存在
* filesize：文件大小
* is_readable：是否可读
* is_writeable：是否可写
* is_executable：是否可执行
* filectime：文件创建时间（inode修改时间）
* filemtime：文件修改时间
* fileatime：文件访问时间
* stat：获取文件的大部分属性值
* unlink：删除文件
* copy：复制文件
* rename：重命名
* fopen：打开文件，返回文件句柄，r、r+、w、w+、x、x+、a、a+、b、t
* fclose：关闭打开的文件
* fwrite：往打开的文件中写
* fread：读取打开文件的指定长度内容，参数1为文件句柄，参数2为读取的长度
* file_get_content：获取文件内容
* fgets：从打开的文件中获取一行
* fgets：从打开的文件中获取一个字符
* readfile：读取一个文件，并输出到输出缓冲区，返回长度
* file：把文件按行读到一个数组中
* ftell：返回文件指针的当前位置
* fseek：移动文件指针到指定位置
* rewind：重置文件指针到文件开头
* ftruncate：把文件截取到指定长度
* flock：文件锁，LOCK_SH共享锁、LOCK_EX排他锁、LOCK_UN释放锁、LOCK_NB附件锁（如果不希望flock在锁定时堵塞，则应该                                                                       加上该锁），锁也可被fclose释放

| 模式 | 描述 |
| --- | --- |
| r | 只读方式打开文件，从文件开头开始读 |
| r+ | 读写方式打开文件，从文件开头开始读写，逐个覆盖原内容 |
| w | 只写方式打开文件，从文件开头开始写。若文件已存在，删除原文件内容；若不存在，创建 |
| w+ | 读写方式打开文件，从文件开头开始读写。若文件已存在，删除原文件内容；若不存在，创建 |
| x | 创建并以写入方式打开，从文件开头开始写。若文件存在，报出E_WARNING级别的错误 |
| x+ | 创建并以读写方式打开，从文件开头开始读写。若文件存在，报出E_WARNING级别的错误 |
| a | 写入方式打开，从文件结尾开始写。若文件不存在，创建 |
| a+ | 读写方式打开，从文件结尾开始读写。若文件不存在，创建 |
| b | 以二进制模式打开文件 |
| t | 以文本模式打开文件，只在Windows下可用 |


### 目录

在UNIX系统中，必须使用'/'作为文件分隔符，而在Windows中，默认使用'\'作为分隔符，同时支持使用'/'作为分隔符。    

在PHP中可以使用DIRECTORY_SEPARATOR获取当前系统的分隔符（Windows为'\'，UNIX为'/'）

* basename：获取文件名部分，第二个参数可以指定扩展名，如果指定了，返回值则不带扩展名
* dirname：获取去掉文件名部分后的目录
* pathinfo：获取路径的目录部分dirname、带扩展的文件名basename、扩展extension、不带扩展的文件名filename
* dir：返回Directory类的实例
* opendir：打开目录，返回目录句柄
* readdir：读取目录
* closedir：关闭目录
* rewinddir：倒回目录句柄，将目录指针重置到开始处
* scandir：浏览目录内容
* glob：检索指定的目录
* mkdir：创建目录
* rmdir：删除目录

### 上传文件

需要注意的点：

* form表单设置enctype="multipart/form-data" method="data"
* PHP使用$_FILES获取上传文件信息
* 多文件上传，可指定name="file[]"的形式

处理上传文件可以用到的函数：

> is_uploaded_file：判断处理的文件是否是通过HTTP POST上传的。但参数必须是类似$_POST['file']['tmp_name']的变量。
>
> move_uploaded_file：将上传移动到新位置，且该函数会判断文件是否为合法的通过HTTP POST上传的文件。

### 下载文件

设置网页头部信息，以保证浏览器是下载文件而不是直接输出：

```php
header('Content-Type: image/gif');
header('Content-Disposition: attachment; filename="test.gif"'); // 说明是附件
header('Content-Length: 3390');

readfile('test.gif');
```

## 图像绘制

### 创建画布

```php
resource imagecreate($x_size, $y_size)          // 基于调色板
resource imagecreatetruecolor($x_size, $y_size) // 真彩
```
### 设置颜色

```php
int imagecolorallocate(resource $image, int $r, int $g, int $b)
```

### 绘制

图形区域填充

```php
bool imagefill(resource $image, int $x, int $y, int $color)
```

绘制点和线

```php
bool imagesetpixel(resource $image, int $x, int $y, int $color)
bool imageline(resource $image, int $x1, int $y1, int $x2, int $y2, int $color)
```

绘制矩阵

```php
// 画一个矩阵
bool imagerectangle(resource $image, int $x1, int $y1, int $x2, int $y2, int $color)
// 画一个矩阵并填充
bool imagefilledrectangle(resource $image, int $x1, int $y1, int $x2, int $y2, int $color)
```

绘制多边形

```php
bool imagepolygon(resource $image, array $point, int $num_points, int $color)
bool imagefilledpolygon(resource $image, array $point, int $num_points, int $color)
```

绘制椭圆

```php
bool imageellipse(resource $image, int $cx, int $cy, int $w, int $h, int $color)
bool imagefilledellipse(resource $image, int $cx, int $cy, int $w, int $h, int $color)
// ($cx, $cy) 中心坐标
// $w, $h 宽、高
```

绘制弧线

```php
bool imagearc(int $cx, int $cy, int $w, int $h, int $s, int $e, int $color)
```

绘制文字

```php
bool imagestring(resource $image, int $font, int $x, int $y, string $s, int $color) // 水平
bool imagestringup(resource $image, int $font, int $x, int $y, string $s, int $color) // 垂直
bool imagechar(resource $image, int $font, int $x, int $y, char $c, int $color)
bool imagecharup(resource $image, int $font, int $x, int $y, char $c, int $color)
// 以ttf字体绘制文本
array imagettftext(resource $image, float $size, float $angle, int $x, int $y, int $color, string $fontfile, string $text)
```

### 生成图像

```php
bool imagegif(resource $image [, string $filename])
bool imagejpeg(resource $image [, string $filename [, int $quality]])
bool imagepng(resource $image [, string $filename])
bool imagewbmp(resource $image [, string $filename [, int $foreground]])
```

### 销毁图像

```php
bool imagedestory(resource $img)
```

## 图像处理

以图片为背景创建图像

```php
resource imagecreatefromjpeg(string $file)
resource imagecreatefrompng(string $file)
resource imagecreatefromgif(string $file)
```

获取图片大小和类型

```php
array getimagesize(string $file [, array &$imageinfo])
```

图片缩放、剪切

```php
bool imagecopyresampled (resource $dst_img, resource $src_img, int $dst_x, int $dst_y, int $src_x, int $src_y, int $dst_w, int $dst_h, int $src_w, int $src_h)
// 该函数可以将图像中的一块正方形区域复制到另一个图像中
```

添加水印

```php
bool imagecopy(resource $des_img, resource $src_img, int $dst_x, int $dst_y, int $src_x, int $src_y, int $src_w, int $src_h)
// 将源图像的(src_x, src,y)的宽src_w，高src_h的一部分复制到目标图像的(dst_x, dst_y)处
```

图片旋转、翻转

```php
resource imagerotate(resource $src_img, float $angle, int $bg_color [, int $ignore_transport])
// 图片旋转angle角度，旋转后未覆盖到的部分使用bg_color填充，igno_transport为非零值，则透明色被忽略。
```

## PDO

### PDO驱动

| PDO驱动 | 对应访问的数据库 |
| --- | --- |
| PDO_BDLIB | FreeTDS/Microsoft SQL Server/Sybase |
| PDO_FIREBIRD | Firbird/Interbase 6 |
| PDO_MYSQL | MySQL 3.x/4.x/5.x |
| PDO_OCI | Oracle (OCI=Oracle Call Interface) |
| PDO_ODBC | ODBC v3 |
| PDO_PGSQL | PostgreSQL |
| PDO_SQLITE | SQLite 2.x/3.x |

### PDO类的方法

#### 构造方法

```php
__construct(string $dsn [, string $username [, string $pwd [, array $driver_options]]])
// dsn：Data Source Name，数据源名，如：
// mysql:dbname=test;host=127.0.0.1;port=3306;charset=utf8
// DSN也可指定文件为参数（如果该文件中存放着DSN字符串），如：
// uri:file:///user/local/dbconnect
```

#### Attribute

```php
// 获取或设置一个数据库连接对象的属性
getAttribute() // 参数：属性名
setAttribute() // 参数：属性名，属性值
```

PDO的一些属性

| 属性名 | 描述 |
| --- | --- |
| PDO::ATTR_AUTOCOMMIT | 确定PDO是否关闭自动提交功能，设置FALSE值时关闭 |
| PDO::ATTR_CASE | 强制PDO获取的表字段字符大小写转换，或原样使用列信息 |
| PDO::ATTR_ERRMODE | 设置错误处理模式，PDO::ERRMODE_SILENT, PDO::ERRMODE_WARNING, PDO::ERRMODE_EXCEPTION |
| PDO::ATTR_PERSISTENT | 确定连接是否为持久连接，默认为FALSE |
| PDO::ATTR_ORACLE_NULLS | 将返回的空字符串转换为SQL的NULL |
| PDO::ATTR_PREFETCH | 设置应用程序提前获取的数据大小(单位：K) |
| PDO::ATTR_TIMEOUT | 设置超时之前等待的时间(单位：s) |
| PDO::ATTR_SERVER_INFO | 包含与数据库特有的服务器信息 |
| PDO::ATTR_SERVER_VERSION | 包含与数据库服务器版本号有关的信息 |
| PDO::ATTR_CLIENT_VERSION | 包含与数据库客户端版本号有关的信息 |
| PDO::ATTR_CONNECTION_STATUS | 包含数据库特有的与连接状态有关的信息 |

#### 错误

```php
// 获取最后一次数据库操作的错误码
errorCode()
// 获取最后一次数据库操作的错误信息
errorInfo()
```

#### SQL

```php
exec() // 执行一条SQL语句，并返回受影响的行数
query() // 执行一条SQL语句，并返回PDOStatement对象（结果集）
quote() // 为SQL语句中的字符串添加引号
lastInsertId() // 获取插入到表中的最后一条数据的主键值
prepare() // 准备要执行的SQL语句，返回PDOStatement对象
getAvailableDrivers() // 获取有效的PDO驱动器名称
beginTransaction() // 开始事务
commit() //提交事务
rollback() // 回滚事务
```

### 预处理语句和PDOStatement类

通过PDO的prepare方法传入一条预处理语句，返回一个PDOStatement查询对象，可以使用该对象传入参数，执行语句。    


| PDOStatement类的成员方法 | 描述 |
| --- | --- |
| bindColumn() | 将列名绑定到一个PHP变量，这样获取到记录后，会将相应的列值赋给该变量 |
| bindParam() | 将参数绑定到相应的占位符 |
| bindValue() | 将一个值绑定到对应的一个参数中 |
| closeCursor() | 关闭游标，使该声明再次被执行 |
| columnCount() | 在结果集中返回列的数目 |
| errorCode() | 获取错误码 |
| errorInfo() | 获取错误信息 |
| execute() | 执行一个预处理查询 |
| fetch() | 获取结果集的下一行，到结尾返回false |
| fetchAll() | 获取结果集的所有行 |
| fetchColumn() | 获取某一列的下一个值 |
| fetchObject() | 获取结果集的下一行，作为对象返回 |
| getAttribute() | 获取属性 |
| setAttribute() | 设置属性 |
| nextRowset() | 获取下一行结果集 |
| rowCount() | 获取结果集的行数 |
| getColumnMeta() | 获取某一列的信息 |
| setFetchMode() | 设置获取结果集的类型 |

## MemCached

### MemCached命令

> stats: 当前所有memcached服务器运行的状态信息
> add: 添加一个数据到服务器
> set: 替换一个已经存在的数据，如果数据不存在，则与add命令相同
> get: 从服务器端获取指定的数据
> delete: 删除指定的单个数据
> flush_all: 删除所有数据

### 在PHP中使用MemCached

需要安装memcached扩展包

```php
Memcached::connect  // 打开一个到memcached的连接
Memcached::pconnect  // 打开一个到memcached的长连接
Memcached::addServer  // 分布式服务器添加一个服务器
Memcached::close  // 关闭一个memcached的连接
Memcached::getStats // 获取当前memcached服务器的状态
Memcached::add  // 添加一个值，若存在，返回false
Memcached::set  // 添加一个值，若存在，覆盖
Memcached::replace  // 替换一个值，类似于set                               
Memcached::get  // 获取一个数据                
Memcached::delete  // 删除一个数据
Memcached::flush   // 清空所有数据
```    

## 框架

一个框架需要的功能：

* 目录组织结构
* 类加载
* 基础类
* URL处理
* 输入处理
* 错误异常处理
* 扩展类

## PHP安全和优化

* 不信任用户输入、验证用户数据
* 及时修补漏洞
* 隐藏Apache和PHP（Apache的ServerSignature、ServerToken指令、PHP的expose_php()函数）
* 删除phpinfo()调用的所有实例
* 修改文档扩展名
* 拒绝访问某些类型的文件

优化

* 使用循环时，应该先计算长度，不要每次都计算一遍
* 使用```isset($str{10})```比```strlen($str) > 10```更快，对于数组也一样
* ```$i++```比```++$i```稍慢（仅适用PHP）


*Mission Complete!*