# 面试问题之PHP

<!-- more -->

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

## 超全局变量

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

> PHP中预定义的常量（魔术常量）：__DIR__、__FILE__、__LINE__、__NAMESPACE__、__CLASS__、__TRAIT__、__FUNCTION__、__METHOD__

## static、self、$this、parent

* static表示当前实例化的类，不能用于访问非静态属性
* self表示当前语句所在的类，也不能用于访问非静态属性，
* $this指的是实际实例化的对象，其不能访问静态属性和常量，且不能出现于静态方法中
* parent指向父类，用于访问父类public的方法及静态常量

## require、include、require_once、include_once

require引用文件出错时报致命错误，并终止脚本执行，include引用文件出错时报警告，但继续执行后续脚本。

require_once、include_once与require、include类似，但文件只会被包含一次。因为包含前存在判断，所以性能比require、include较差。

> 应尽量不使用include_once、require_once

## 常见的数组处理函数

* array_count_values: 统计数组中各个value的个数
* array_diff: 判断一个数组与其他数组的差异，返回在第一个数组中但不在其他数组中的元素，例如array_duff([1,2,3,4], [1,2], [4])返回[2 => 3]
* array_key_exists: 判断key是否存在
* in_array: 判断值是否存在于数组中
* array_keys: 获取数组的所有键
* array_values: 获取数组的所有值
* array_merge: 合并数组，后者覆盖前者。使用“+”则是前者覆盖后者
* sort: 数组排序
* count: 数组长度，等同于sizeof
* array_shift: 删除数组第一个元素，并返回
* array_unshift: 在数组头部加入一个元素
* array_pop: 删除数组末尾最后一个元素，并返回
* array_push: 向数组末尾添加一个元素
* array_reverse: 翻转数组
* array_rand: 从数组中取随机值
* array_flip: 交换数组的键值
* array_search: 搜索数组的值，返回第一个匹配到的元素的键
* array_sum: 计算数组所有值的和
* array_slice: 根据开始位置和长度复制数组元素，返回复制的元素（即原数组不改变）
* array_splice: 根据开始位置和长度剪切数组元素，并返回剪切的元素（即原数组中匹配到的元素被删除） 
* array_map: 对数组进行遍历，并对每个元素使用提供的回调函数

## 常见的字符串处理函数

* chr: ASCII码转字符
* ord: 字符转ASCII码
* count_chars: 统计字符串中的字符数
* str_word_count: 统计字符串中的单词数
* explode: 按指定字符将字符串分解为数组
* implode: 按指定字符将数组合并为字符串
* htmlentities: 转义处理字符串中的HTML实体。html_entity_decode用于反向解码
* htmlspecialchars: 转义处理字符串中的特殊字符，包括&、"、'、<、>。htmlspecialchars_decode用于反向解码
* trim: 去除字符串两端的空格，ltrim、rtrim分别用于去除左边界的空格和右边界的空格
* str_len: 计算字符串长度
* strrev: 翻转字符串
* substr: 根据起始位置和长度获取子字符串，返回获取的子字符串
* preg_replace: 正则表达式替换
* preg_match: 正则表达式匹配
* strstr: 判断字符串2是否为字符串1的子字符串，如果是，返回匹配到的开始位置到父字符串末尾的字符串，如果不是，返回null
* strrchr: 与strstr一样，只是该函数从父字符串的末尾开始匹配
* strcmp: 字符串比较函数，如果字符串1>字符串2，返回1；如果字符串1<字符串2，返回-1；如果字符串1等于字符串2，返回0；如果两个字符串部分内容不一致，返回两个字符串中第一个不同的字符的差值
* str_replace: 搜索并替换字符串的内容

## 多台服务器session共享的解决方案

* 将session数据存储到MySQL数据库中
* 将session数据存储到Redis、MemCache这些内存型数据库中
* 使用NFS共享session文件
* 使用cookie保存session信息

## 魔术方法

* __construct()
* __destruct()
* __call(): 当调用不存在的方法时
* __callStatic(): 当调用不存在的惊天方法时
* __get(): 获取不存在的属性时
* __set(): 写入不存在的属性时
* __sleep(): 当对象被序列化时
* __wakeup(): 反序列化时
* __toString(): 对象作为字符串使用时
* __invoke(): 当对象被作为函数调用时
* __isset(): 当使用isset函数探测对象的私有属性时
* __unset(): 当使用unset函数删除对象的私有属性时
* __clone(): 当对象被克隆

## public、protected、private、final

* public可被外部访问，protected与private不可被外部访问
* public、protected可被子类继承，private不可被继承
* final描述的方法，不可被子类重写；final描述的类，不可被继承

## SERVER_ADDR、REMOTE_ADDR、HTTP_X_FORWARDED_FOR、HTTP_CLIENT_IP

* SERVER_ADDR获取服务器IP
* REMOTE_ADDR获取客户端IP，但当用户使用代理时，获取到的就是代理服务器的IP
* HTTP_X_FORWARDED_FOR、HTTP_CLIENT_IP可以获取到用户真实IP及使用的各个代理的IP，但该字段可能被伪造

> 当后端架构中有使用到代理服务器，可以使用HTTP_X_FORWARDED_FOR、HTTP_CLIENT_IP来进行获取用户IP。如果后端服务器直接暴露给用户，只需要使用REMOTE_ADDR获取客户端IP即可。

## php.ini配置


| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| short_open_tag | On | 是否开启缩写模式 |
| precision | 14 | 浮点数显示有效数字的位数 |
| disable_functions |  | 禁用的函数 |
| disable_classes |  | 禁用的类 |
| expose_php | On | 是否暴露PHP被安装在服务器上 |
| max_execution_time | 30 | 最大执行时间 |
| memory_limit | 128M | 每个脚本执行时内存限制 |
| error_reporting | E_ALL & ~E_DEPRECATED & ~E_STRICT | 设置错误报告级别 |
| display_errors | On | 显示错误 |
| log_errors | On | 设置是否将错误日志记录到error_log |
| error_log |  | PHP错误日志文件 |
| upload_max_filesize | 2M | 上传文件的最大限制 |
| post_max_size | 8M | 设置POST最大数据限制 |

## php-fpm.conf配置信息


| 配置项 | 默认 | 备注 |
| --- | --- | --- |
| pid |  | pid文件 |
| error_log |  | 错误日志的位置 |
| log_level | notice | 错误级别（alert: 必须立即处理；error: 错误信息；warning: 警告信息；notice: 一般重要信息；debug: 调试信息） |
| daemonize | yes | 设置FPM后台运行 |
| listen |  | FPM监听的地址及端口 |
| request_slowlog_timeout | 0 | 慢日志记录阈值 |
| slowlog |  | 慢日志记录文件 |

## 进程通讯方式

* 消息队列
* socket
* 信号量
* 共享内存
* 管道

## 发起HTTP请求的方式

* curl: curl_init -> curl_setopt -> curl_exec -> curl_close
* file_get_contents: http_build_query -> stream_context_create -> file_get_content(url, false, context)
* fopen: fopen -> fgets -> fclose
* fsockopen: fsocketopen -> fputs -> fgets -> fclose

## 生成器yield的基本原理

> 当一个生成器被调用的时候，它返回一个可以被遍历的对象。
> 当你遍历这个对象的时候 (例如通过一个 foreach 循环)，PHP 将会在每次需要值的时候调用生成器函数，并在产生一个值之后保存生成器的状态，这样它就可以在需要产生下一个值的时候恢复调用状态。
> 
> 一旦不再需要产生更多的值，生成器函数可以简单退出，而调用生成器的代码还可以继续执行，就像一个数组已经被遍历完了。

此外，yield还可以接受值：

```php
function printer()
{
    while (true) {
        printf("receive: %s\n", yield);
    }
}

$printer = printer();

$printer->send('hello');
$printer->send('world');
```

PHP的生成器还实现了一个send方法，该方法向yield处传递一个值，同时从 yied 语句处继续执行，直至再次遇到 yield 后控制权回到外部。   

Generator生成器提供的方法有：

* current: 获取生成器当前的值
* getReturn: 获取生成器返回的值
* key: 获取生成器当前的key
* next: 继续执行生成器
* rewind: 重置生成器
* send: 发送值给生成器
* valid: 验证生成器是否关闭
* throw: 抛出一个生成器异常

## 防止内存泄露的解决方法

* 过大的数据分段读取
* 尽可能减少静态变量的使用，可以考虑使用引用代替
* 数据库、文件操作完后，及时释放连接
* 对象、变量使用完后，及时unset释放
* unset()函数只能在变量值占用内存空间超过256字节时才会释放内存空间
* 当指向该变量的所有变量（如引用变量）都被销毁后，才会释放内存

## 模板引擎的原理

* 通过视图名找到相应的视图文件
* 通过正则表达式或其它方式将模板引擎的定界符及其他符号修改为PHP的定界符和符号
* 将转化好的文件内容存为缓存文件，并以视图文件的上一次修改时间作为缓存文件的命名依据，保证视图文件更新后，缓存文件也进行更新

## 反射类

* ReflectionClass
* ReflectionFunction
* ReflectionMethod
* ReflectionObject
* ReflectionGenerator
* ...

## 正则表达式

* \d: 数字
* \D: 非数字
* \w: 字母、数字、下划线
* \W: 非字母、数字、下划线
* \s: 空白符，空格、制表、换行
* \S: 非空白符
* .: 除换行外的任意字符
* *: 0~n个
* ?: 0或1个
* +: 1~n个
* {n}: n个
* {n,}: 大于等于n个
* {n,m}: n~m个
* ^: 开始
* $: 结束
* []: 匹配中括号中任意一个
* [^]: 除了括号中的
* (): 整体或后文引用

> 修正符:
> i: 区分大小写
> S: .包括换行
> m: 每一行分别进行匹配
> U: 取消贪婪模式，另一种方法就是在量词后面加问号，如*?
> u: utf8中文匹配

## CGI、FastCGI、PHP-FPM

* CGI: 通用网关接口，是Web服务器与服务端语言交互的标准接口。在一个请求到来时，服务器创建一个CGI进程用来执行程序，等待程序执行完成，该CGI进程被杀死。

* FastCGI: 是用于提升CGI性能的标准，FastCGI要求CGI进程在处理完请求后不被立刻杀死，而是保留下来，等待处理下一次请求。

* PHP-FPM: FastCGI Processor Manager，实现FastCGI的管理器，使用FastCGI标准管理CGI程序

## PHP操作MySQL数据库

### mysqli面向过程

```php
// 使用prepare（推荐）
$conn = mysqli_connect('127.0.0.1', 'root', '', 'test');
mysqli_set_charset($conn, 'utf8');
$stmt = mysqli_prepare($conn, 'SELECT `name`, `phone` FROM `aihailin` LIMIT ?');
mysqli_stmt_bind_param($stmt, 'i', $limit);
$limit = 2;
mysqli_execute($stmt);
$result = mysqli_stmt_get_result($stmt);
foreach ($result as $row) {
  var_dump($row);
}
// 或不使用prepare
$conn = mysqli_connect('127.0.0.1', 'root', '', 'test');
mysqli_set_charset($conn, 'utf8');
$res = mysqli_query($conn, 'SELECT * FROM `aihailin` LIMIT 2');
foreach ($res as $row) {
  var_dump($row);
}
```

### mysqli面向对象

```php
$mysqli = new mysqli('127.0.0.1', 'root', '', 'test');
$mysqli->set_charset('utf8');
$stmt = $mysqli->prepare('SELECT `name`, `phone` FROM `aihailin` LIMIT ?');
$stmt->bind_param('i', $limit);
$limit = 2;
$stmt->execute();
$result = $stmt->get_result();
foreach ($result as $row) {
  var_dump($row);
}
```

### PDO

```php
$dsn = 'mysql:host=127.0.0.1;dbname=test;charset=utf8';
$pdo = new PDO($dsn, 'root', '');
$stmt = $pdo->prepare('SELECT * FROM `aihailin` LIMIT :limit');
$stmt->bindParam(':limit', $limit);
$limit = 5;
if ($stmt->execute()) {
    $result = $stmt->fetchAll(PDO::FETCH_ASSOC);
    var_dump($result);
}
```