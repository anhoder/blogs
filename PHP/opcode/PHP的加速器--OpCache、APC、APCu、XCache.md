# PHP的加速器--OpCache、APC、APCu、XCache

<!-- more -->

首先，我们要知道，这些工具都是给PHP做加速用的。

在介绍这些工具之前，先简单的描述一下PHP的工作流程：

> 1. PHP请求到来
> 2. 读取PHP文件
> 3. Zend引擎进行词法分析与语法分析
> 4. 生成Zend可执行的指令opcode
> 5. 执行opcode
> 6. 返回响应

而OpCache、APC等此类工具就是通过缓存opcode指令来提升PHP的效率。

## OpCache

### 开启OpCache

OpCache在PHP 5.5及之后的版本是默认安装的，不需要我们去下载源码进行编译了，只要在PHP配置文件中开启即可：

```ini
[opcache]
zend_extension=opcache.so
opcache.enable=1
opcache.enable_cli=1
```

### OpCache是否生效

以下是opCache的更多配置（摘自[https://www.zybuluo.com/phper/note/1016714](https://www.zybuluo.com/phper/note/1016714)）：

```ini
opcache.enable=1 (default "1")
;OPcache打开/关闭开关。当设置为Off或者0时，会关闭Opcache, 代码没有被优化和缓存。

opcache.enable_cli=1 (default "0")
;CLI环境下，PHP启用OPcache。这主要是为了测试和调试。从 PHP 7.1.2 开始，默认启用。

opcache.memory_consumption=128 (default "64")
;OPcache共享内存存储大小。用于存储预编译的opcode（以MB为单位）。

opcache.interned_strings_buffer=8 (default "4")
;这是一个很有用的选项，但是似乎完全没有文档说明。PHP使用了一种叫做字符串驻留（string interning）的技术来改善性能。例如，如果你在代码中使用了1000次字符串“foobar”，在PHP内部只会在第一使用这个字符串的时候分配一个不可变的内存区域来存储这个字符串，其他的999次使用都会直接指向这个内存区域。这个选项则会把这个特性提升一个层次——默认情况下这个不可变的内存区域只会存在于单个php-fpm的进程中，如果设置了这个选项，那么它将会在所有的php-fpm进程中共享。在比较大的应用中，这可以非常有效地节约内存，提高应用的性能。
;这个选项的值是以兆字节（megabytes）作为单位，如果把它设置为16，则表示16MB，默认是4MB，这是一个比较低的值。

opcache.max_accelerated_files (default "2000")
;这个选项用于控制内存中最多可以缓存多少个PHP文件。这个选项必须得设置得足够大，大于你的项目中的所有PHP文件的总和。设置值取值范围最小值是 200，最大值在 PHP 5.5.6 之前是 100000，PHP 5.5.6 及之后是 1000000。也就是说在200到1000000之间。你可以运行“find . -type f -print | grep php | wc -l”这个命令来快速计算你的代码库中的PHP文件数。

opcache.max_wasted_percentage (default "5")
;计划重新启动之前，“浪费”内存的最大百分比。

opcache.use_cwd (default "1")
;如果启用，OPcache将在哈希表的脚本键之后附加改脚本的工作目录， 以避免同名脚本冲突的问题。禁用此选项可以提高性能，但是可能会导致应用崩溃

opcache.validate_timestamps (default "1")
;如果启用（设置为1），OPcache会在
opcache.revalidate_freq设置的秒数去检测文件的时间戳（timestamp）检查脚本是否更新。
;如果这个选项被禁用（设置为0），opcache.revalidate_freq会被忽略，PHP文件永远不会被检查。这意味着如果你修改了你的代码，然后你把它更新到服务器上，再在浏览器上请求更新的代码对应的功能，你会看不到更新的效果，你必须使用 `opcache_reset()` 或者 `opcache_invalidate()` 函数来手动重置 OPcache。或者重重你的web服务器或者php-fpm 来使文件系统更改生效。
;我强烈建议你在生产环境中设置为0，why？因为当你在更新服务器代码的时候，如果代码较多，更新操作是有些延迟的，在这个延迟的过程中必然出现老代码和新代码混合的情况，这个时候对用户请求的处理必然存在不确定性。最后，等所有的代码更新完毕后，再平滑重启PHP和web服务器。
    
opcache.revalidate_freq (default "2")
;这个选项用于设置缓存的过期时间（单位是秒），当这个时间达到后，opcache会检查你的代码是否改变，如果改变了PHP会重新编译它，生成新的opcode，并且更新缓存。值为“0”表示每次请求都会检查你的PHP代码是否更新（这意味着会增加很多次stat系统调用，译注：stat系统调用是读取文件的状态，这里主要是获取最近修改时间，这个系统调用会发生磁盘I/O，所以必然会消耗一些CPU时间，当然系统调用本身也会消耗一些CPU时间）。可以在开发环境中把它设置为0，生产环境下不用管。
;如果 `opcache.validate_timestamps` 配置指令设置为禁用（设置为0），那么此设置项将会被忽略。
    
opcache.revalidate_path (default "0")
;在include_path优化中启用或禁用文件搜索
;如果被禁用，并且找到了使用的缓存文件相同的include_path，该文件不被再次搜索。因此，如果一个文件与include_path中的其他地方相同的名称出现将不会被发现。如果此优化对此有效，请启用此指令你的应用程序，这个指令的默认值是禁用的，这意味着该优化是活跃的。
    
opcache.fast_shutdown（默认“0”）
;如果启用，则会使用快速停止续发事件。 所谓快速停止续发事件是指依赖 Zend 引擎的内存管理模块 一次释放全部请求变量的内存，而不是依次释放每一个已分配的内存块。该指令已在PHP 7.2.0中被删除。快速关机序列的一个变种已经被集成到PHP中，并且如果可能的话将被自动使用。
```

### OpCache其他内容

除了配置项以外，opCache还提供一些函数：

* opcache_compile_file — 无需运行，即可编译并缓存 PHP 脚本，该函数可用于在 Web 服务器重启之后初始化缓存，俗称缓存预热
* opcache_get_configuration — 获取php.ini中的配置信息
* opcache_get_status — 获取缓存的状态信息
* opcache_invalidate — 废除脚本缓存
* opcache_is_script_cached — 一个php文件是否被缓存
* opcache_reset — 重置情况所有的缓存内容

## APC（Alternative PHP Cache 可选的PHP缓存）

APC包含两个功能：用户缓存（用户数据缓存，类似MemCache）、opcode缓存

> 注意：apc的3.1.14版本存在严重的内存问题，被官方废弃。最新可用的apc版本为3.1.13，仅支持php 5.3.* 。之后的PHP版本中自带了opCache

## APCu（APC user cache，APC的用户缓存）

APCu保留了APC的用户数据缓存部分，去掉了opcode缓存。且提供的接口与APC一致。

### APCu的安装

1. 前往[pecl下载页面](http://pecl.php.net/package/APCu)下载源码包
2. 解压并进入源代码根目录
3. 使用PHP的 `phpize` 命令工具读取config.m4文件，并生成configure
4. 执行生成的 `configure` 文件: `./configure with-php-config=/usr/bin/php-config`
5. 使用 `make` 及 `sudo make install` 进行编译安装
6. 在php.ini中添加扩展:     

   ```ini
   [apcu]
   extension      = apcu.so
   apc.enabled    = on
   apc.shm_size   = 256m
   apc.enable_cli = on
   ```

### APCu的使用

APCu的使用可以参考[https://www.php.net/manual/zh/book.apcu.php](https://www.php.net/manual/zh/book.apcu.php)

## XCache

XCache的安装与前面的安装方法类似：

1. 前往[https://xcache.lighttpd.net/pub/Releases/](https://xcache.lighttpd.net/pub/Releases/)下载源代码压缩包
2. 解压并进入根目录
3. 使用`phpize`、`./configure with-php-config=/usr/bin/php-config`、`make`、`sudo make install`进行编译安装
4. 修改php.ini，添加扩展配置

