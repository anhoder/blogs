# Git补坑

这段时间一直忙着写作业和论文，心力交瘁，今天突然发现好久没写博客了（心痛的感觉QAQ），所以来给网站擦擦灰。。

*Mission Start!*

其实以前是很耐心学过Git的，只是没有写博客记录相关内容，现在来补上这个坑 :)   
   
<span style="color:red;">**值得注意的是：Git只能跟踪文本文件的内容变化，而不能跟踪二进制文件的内容变化（包括Word文档）**</span>
## 一、安装Git
> ### Windows
> 在Windows下，可以在[官网](https://git-scm.com/downloads)上下载Windows的安装程序进行安装，安装完成打开Git Bash即可。
> ### Mac
> 第一种方法：使用Homebrew进行安装（如果已经安装了[Homebrew](https://brew.sh/index_zh-cn)的话），使用命令进行安装：

> ```sh
> brew install git
> ```
> 第二种方法：安装Xcode，Xcode集成了Git，但是默认未安装，你可以在Xcode的Preferences->Download中安装Command Line Tools。
> ### Linux
> Linux不同的发行版有不同的安装方式，可以尝试使用以下方法：

> ```sh
> sudo apt-get install git
> ```
> 或者

> ```sh
> sudo yum install git
> ```
 
安装完成后，使用

```sh
git config --global user.name "Your Name"
git config --global user.email "user@example.com"
```
配置用户名和邮箱。

## 二、Git相关操作

### 1.创建版本库(仓库，Repository)
```sh
$ mkdir Test  
$ cd Test
$ git init    // 创建仓库
Initialized empty Git repository in /Volumes/办公/Alan/Test/.git/
```
OK, 仓库已经建好，Git会在该目录下生成一个目录.git（.开头为隐藏目录），其用于跟踪管理这个仓库。   
   
如果使用的是某些特殊的shell，当切换到该目录时，可能会出现明显的变化，例如我使用的zsh，当执行git init创建好仓库后，切换到该目录下时（即Test），路径显示变成

```sh
Test git:(master) ✗ $
```

### 2.添加文件到版本库

```sh
$ touch readme.md     // 创建readme.md文件
$ git add readme.md   // 添加readme.md到仓库
$ git add test.php    // 添加test.php到仓库
```
### 3.提交文件到版本库

```sh
$ git commit -m "add readme.me test.php" // 提交到仓库（-m参数是提交的相关说明，添加提交说明至关重要）
```

可多次使用git add添加多个文件，然后使用git commit一次进行提交。每次的git commit都会生成一个新的版本。

### 4.查看提交日志

```sh
$ git log
```
显示结果为：

```sh
commit bc916412b5b6ca835a21c72a530598db3ae38fb0 (HEAD -> master)
Author: AlanAlbert <1766447919@qq.com>
Date:   Thu Jun 21 09:27:42 2018 +0800

    upload Sort

commit ee45db03f09b212562520be5c1a4e1d853064a2e
Author: AlanAlbert <1766447919@qq.com>
Date:   Thu Jun 21 09:17:41 2018 +0800

    upload readme.md test.php
```
git log会根据时间倒序显示每条提交记录，包括提交者、提交时间以及提交说明，也可以给git log加上参数--pretty=oneline让其只显示一行，只包括提交说明：

```sh
bc916412b5b6ca835a21c72a530598db3ae38fb0 (HEAD -> master) upload Sort
ee45db03f09b212562520be5c1a4e1d853064a2e upload readme.md test.php
```
上面两次都出现的很长一串的字符串是每次提交的id，其通过SHA1计算出来的十六进制数字，保证唯一性。

### 5.重置版本
当前版本可以在git log的结果中看到，其用HEAD指向当前版本。上一个版本则为HEAD^，上上个版本为HEAD^^，上5个版本可以表示为HEAD~5。因此上一个版本可以写作：

```sh
$ git reset --hard HEAD^
HEAD is now at ee45db0 upload readme.md test.php
```
使用git log --pretty=oneline显示结果：

```sh
ee45db03f09b212562520be5c1a4e1d853064a2e (HEAD -> master) upload readme.md test.php
```
可见HEAD指向了上一个commit版本。
> git reset命令将HEAD指向另一个commit版本，并将工作目录进行更新。
> 那么--hard参数是干什么的呢？这在后面对Git有了更好的理解后将会对其进行说明，现在我们只需要了解git reset命令的参数有：--soft、--hard、--mixed，默认为mixed。

在回退到上一个版本后，最新的那个版本不见了，如果又想回到最新的版本怎么办？
办法是有的，只要你知道最新版本的commit id，可以使用：

```sh
$ git reset --hard bc91641
HEAD is now at bc91641 upload Sort
```
commit id并不需要输入完整，只要能够唯一区分目标版本即可。
> 如果实在不记得版本的id，有两种方法可以进行查找，
> 第一种较简单：使用命令git reflog，第一列显示就是版本id
> 第二种比较麻烦：在日志文件.git/logs/refs/heads/master中查看，其内容如下：

> ```sh
> 0000000000000000000000000000000000000000 ee45db03f09b212562520be5c1a4e1d853064a2e username <user@example.com> 1529543861 +0800	commit (initial): upload readme.md test.php
> ee45db03f09b212562520be5c1a4e1d853064a2e bc916412b5b6ca835a21c72a530598db3ae38fb0 username <user@example.com> 1529544462 +0800	commit: upload Sort
> /*结构组成如下：
> 上一个版本id 当前版本id 用户名 用户邮箱 时间戳 时区(+0800表示正8时区，北京所在时区) 执行的操作: 操作说明
> ```

### 6.工作区和暂存区
工作区不用多说，其就是我们可见的目录。
而在工作区下的隐藏文件夹.git下包含一个叫做stage或index的暂存区、第一个分支master以及指向master的指针HEAD，git add就是将文件添加至暂存区，git commit将暂存区的所有内容提交到当前分支，提交后暂存区的内容清空。   
    
### 7.查看状态
使用git status可以查看状态：

```sh
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

  new file:   test1.php
	modified:   test2.php
	
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   test3.php

Untracked files:
  (use "git add <file>..." to include in what will be committed)
  
	test4.php

no changes added to commit (use "git add" and/or "git commit -a")
```
master分支下状态结果显示分三个状态：
① Changes to be committed表示已添加到暂存区但未提交的修改
② Changes not staged for commit表示被修改但未未添加到暂存区的修改，执行git add filename后该修改便会出现在状态① Changes to be committed中，并显示为modified: filename
③ Untracked files表示未跟踪的文件，执行git add filename后该文件便会出现在状态① Changes to be committed中，并显示为new file: filename

### 8.查看差异
使用git diff HEAD可以显示最新提交版本和工作区的差异：

```sh
$ git diff HEAD
diff --git a/test.php b/test.php
index 94fd6d7..46a8d83 100644
--- a/test.php
+++ b/test.php
@@ -1,7 +1,6 @@
 <?php
 /**
- * add line
+ * test line
  */
    
diff --git a/test2.php b/test2.php
index 10d6ee9..4a3f8ca 100644
--- a/test2.php
+++ b/test2.php
@@ -1,6 +1,6 @@
 <?php
-
+// test.line
```
因此，git diff HEAD会显示工作区和最新版本之间所有的差异，如果要检查某一个文件在工作区和最新提交版本之间的差异则使用git diff HEAD -- test.php：

```sh
$git diff HEAD -- test.php
diff --git a/test.php b/test.php
index 94fd6d7..46a8d83 100644
--- a/test.php
+++ b/test.php
@@ -1,7 +1,6 @@
 <?php
 /**
- * add line
+ * test line
  */
```

### 9.撤销修改
对工作区进行修改后，如果想要放弃这次的修改可以使用：

```sh
$ git checkout -- test.php
```
该命令会让工作区的文件回到最近一次git commit或git add时的状态。

### 10.删除文件
当删除工作区的文件后，使用git status查看状态：

```sh
$ git status 
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	deleted:    readme.md
	
```
可以使用以下命令删除版本库的filename文件：

```sh
$ git rm filename
```
然后使用git commit将修改提交到版本库。












