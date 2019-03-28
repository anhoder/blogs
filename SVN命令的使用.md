# SVN命令的使用

*Mission Start!*

## svnadmin

### 1、使用svnadmin初始化SVN项目:

```sh
svnadmin create /home/user1/project
```

### 2、修改配置文件

① 修改passwd，添加用户密码，例如：

```ini
[users]
alan = 123456
```

② 修改权限文件authz，给用户设置权限：

```ini
[aliases]

[groups]

[/]
alan = rw
```

③ 修改SVN服务器配置文件：

```ini
[general]
# 以下两个配置的可选选项有read、write、none，read为只读权限，write为可读可写权限，none为不可读不可写
# 未认证用户权限
anon-access = read    
# 认证用户权限
auth-access = write  

# 用户密码数据，指定为第一步修改的文件
password-db = passwd
# 用户权限数据，指定为第二步修改的文件
authz-db = authz
```

## svnserve

svnserve命令用于启动SVN服务。可选参数有：

> -r: 指定SVN服务的root目录
> -d: Daemon守护程序方式运行
> -R: Read Only
> -i: 超级服务器模式
> -t: 隧道模式

启动SVN服务：

```sh
svnserve -d -r /home/user1/project
```

## svn

### 1、从SVN服务器下拉代码

```sh
svn chekout svn://127.0.0.1
// 或 svn co svn://127.0.0.1
```

### 2、查看当前工作目录下的文件状态

```sh
svn status
// 或 svn st
// ? 表示未在SVN的控制下
// M 表示已修改
// A 表示已添加
// D 表示已删除
// C 表示冲突文件
// I 表示忽略的文件
// R 表示被替换
```

### 3、查看当前工作目录下文件的版本差异

```sh
svn diff [file]
// file为可选参数，表示查看指定文件的差异
svn diff -r m:n file
// 查看m版本和n版本的文件差异

```

### 4、提交文件到SVN服务器

```sh
svn commit -m '【新增】a.php'
// 或 svn ci -m '【新增】a.php'
```

### 5、从SVN服务器更新文件到本地

```sh
svn update [file]
// 或 svn up [file]
// file为可选参数，表示只更新指定文件内容
```

### 6、加锁与解锁

```sh
// 加锁，锁定目录，让别人无法提交更新到这里
svn lock file -m '【锁定】a.php'
// 解锁
svn unlock file
```

### 7、查看SVN日志

```sh
svn log [file]
// 查看版本日志信息
```

### 8、查看文件或项目详细信息

```sh
svn info [file]
```

### 9、合并分支

```sh
svn merge -r m:n file
// 合并m版本和n版本的file文件差异，例如：svn merge -r 500:502 a.php
```

### 10、恢复本地修改

```sh
svn revert file
```

### 11、移除冲突状态

在处理完文件冲突后，可以使用该命令移除文件冲突的状态：

```sh
svn resolved file
```

### SVN代码库表更

```sh
svn switch URL
```

*Mission Complete!*