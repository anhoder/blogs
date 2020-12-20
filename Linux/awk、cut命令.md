# awk、cut命令

<!-- more -->

*Mission Start!*

## awk命令

### 参数

可以选用的参数：

> -F ',': 指定列分割符
> -v var=value: 赋值一个用户定义变量，将外部变量传递给awk
> -f scriptfile: 从脚本文件中读取awk命令

### 模式

* /正则表达式/：使用通配符的扩展集
* 关系表达式：使用运算符进行操作，可以是字符串或数字的比较测试
* 模式匹配表达式：用运算符~和~!分别表示匹配、不匹配
* BEGIN语句块、pattern语句块、END语句块

### 使用的基本结构

```sh
awk 'BEGIN{ print "start"} pattern { commands } END{ print "end" }' file
```

awk脚本由：**BEGIN语句块、能够使用模式匹配的通用语句块、END语句块**组成，例如：

```sh
awk 'BEGIN{ i=0 } { i++ } END{ print i }' filename
awk '/^2/ {print $0}' filename // 输出以2开头的行
```

### awk的工作原理

执行步骤：

> 1. 执行 ```BEGIN {commands}```语句块中的语句；
> 2. 从文件或标准输入（stdin）读取一行，然后执行 ```pattern {commands}``` 语句，逐行扫描文件，直至文件结尾；
> 3. 当读到文件结尾处时，执行 ```END {commands}``` 语句块；

<font style="color:red;">*注意：在awk的print语句中，双引号被当做拼接符使用*</font>

### awk的内置变量

> $n: 表示当前记录的第n个字段（列）
> $0: 该变量包含执行过程中当前行的文本内容
> FILENAME: 当前输入文件名
> FS: 字段分隔符
> NF: 每行的字段数（列数）
> NR: 记录数，即当前的行号
> OFMT: 数字的输出格式
> OFS: 输出字段分隔符
> ORS: 输出记录（行）分隔符，默认为换行
> RS: 记录（行）分隔符，默认为换行

使用实例：

```sh
awk '{print $NF}' filename
// 输出每一行最后一列的值
awk '{if($2=="abc") s+=1;} END {print s}' a.txt
// 计算第2列为abc的行数
```

## cut命令

### 参数

> -d ',': 指定列分隔符为逗号
> -f 9: 获取第9列的数据
> -c 1-3: 获取每行的第1至第三个字符

使用实例：

```sh
cut -d ',' -f 8 filename
// 输出第8列的内容
```

cut一般与grep配合使用，例如：

```sh
cut -d ',' -f 6 a.txt | grep -c 'abc'
// 计算第6列为abc的行数
```

*Mission Complete!*