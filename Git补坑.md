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
添加文件到仓库，让Git跟踪该文件：

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
> 
> 那么--hard参数是干什么的呢？git reset的参数有三个：--soft、--hard、--mixed，默认为mixed。
> 
> ① **--soft**: HEAD指向另一个commit版本，而工作区和暂存区不发生变化。即，将HEAD头指针恢复，而add的缓存和工作区的所有东西都不变。
> 
> ② **--mixed**: HEAD的指向、暂存区都发生变化，工作区不发生变化。即，将HEAD头指针恢复，add的缓存也会丢失，但工作区文件不变。
> 
> ③ **--hard**: HEAD的指向、暂存区、工作区都发生变化。即，HEAD头指针恢复、add的缓存丢失、工作区的代码修改将恢复。

在回退到上一个版本后，最新的那个版本不见了，如果又想回到最新的版本怎么办？
办法是有的，只要你知道最新版本的commit id，可以使用：

```sh
$ git reset --hard bc91641
HEAD is now at bc91641 upload Sort
```
commit id并不需要输入完整，只要能够唯一区分目标版本即可。

> 如果实在不记得版本的id，有两种方法可以进行查找，
> 
> 第一种较简单：使用命令git reflog，第一列显示就是版本id
> 
> 第二种比较麻烦：在日志文件.git/logs/refs/heads/master中查看，其内容如下：

> ```sh
> 0000000000000000000000000000000000000000 ee45db03f09b212562520be5c1a4e1d853064a2e username <user@example.com> 1529543861 +0800	commit (initial): upload readme.md test.php
> ee45db03f09b212562520be5c1a4e1d853064a2e bc916412b5b6ca835a21c72a530598db3ae38fb0 username <user@example.com> 1529544462 +0800	commit: upload Sort
> /*结构组成如下：
> 上一个版本id 当前版本id 用户名 用户邮箱 时间戳 时区(+0800表示正8时区，北京所在时区) 执行的操作: 操作说明
> ```

### 6.工作区 （working copy）和暂存区（staging copy）
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

### 11.添加远程仓库
以使用的较多的GitHub为例。   
首先，需要在GitHub上为你的账户配置ssh密钥，确保在电脑上可以通过GitHub的验证，可以在命令行或终端使用如下命令进行测试：

```sh
$ ssh git@github.com
```
若结果显示为：

```sh
PTY allocation request failed on channel 0
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```
则表示通过GitHub的验证；   

然后，使用命令关联远程仓库：

```sh
$ git remote add origin git@github.com:username/test.git 
```
其中，origin为远程库的名字，然后使用git push将本地库内容推送到远程库：

```sh
$ git push -u origin master
```
第一次推送时加上-u参数，Git不但会把master分支内容推送给远程新的master分支，还会把本地master分支和远程master分支关联起来。在以后将本地内容推送到远程仓库时，只需要使用以下命令即可：

```sh
$ git push origin master
```
将远程库的更改合并至当前库：

```sh
$ git pull origin master:test // 远程master与test分支合并
$ git pull origin master // 远程master和当前分支合并
```
### 11.从远程仓库克隆
远程仓库的内容克隆到本地：

```sh
$ git clone git@github.com:username/test.git
```
然后，Git就会下载远程仓库的内容到本地。   
Git默认使用的是ssh协议，除此之外还可以使用https等协议，例如：

```sh
$ git clone https://github.com/username/test.git
```
### 12.分支管理
首先了解一下分支是什么？每个分支都是一条timeline(时间线)，前面用到的分支就是master主分支，用户可以在分支上单独操作互不影响。   
创建test分支：

```sh
$ git branch test
```
切换到test分支：

```sh
$ git checkout test
Switched to branch 'test'
```
以上两步也可以使用一个命令完成：

```sh
$ git checkout -b test
```
查看所有分支（*号表示当前分支）：

```sh
$ git branch
  master
* test
```
合并分支：

```sh
$ git checkout master
$ git merge test
Updating 8d8593e..4b01548
Fast-forward
 google.md | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 google.md
```
删除分支：

```sh
$ git branch -d test
Deleted branch test (was cfc8d84).
```
为了更好的理解Git的分支，我们做一个小试验：
> ① git init创建一个仓库
> 
> ② 在master分支中新建readme.md，使用git add和git commit提交至仓库
> 
> ③ 使用git checkout -b test1新建并切换至test1分支，在test1分支中新建并提交test1.md
> 
> ④ 使用git checkout -b test2新建并切换至test2分支，在test2分支中新建并提交test2.md
> 
> ⑤ 使用git checkout master切换至master分支，在master分支中新建test.md文件，并提交   
> 
> ⑥ 此时，各分支状况如下图所示：
> 
> ![分支状况1](https://media.alan123.xyz/imgs/blogs/git/1.png)
>   
> ⑦ 在master、test1、test2分支中切换时，工作区的文件也会随之变化：切换到master时，工作区有readme.md和test.md；切换至test1分支时，工作区有readme.md和test1.md；切换至test2分支时，工作区有readme.md和test2.md文件
> 
> ⑧ 切换至master分支，使用git merge test1 test2将test1和test2分支合并至master
> 
> ⑨ 此时，master分支下的文件包含原来的readme.md、test.md以及加入的test1.md、test2.md，而test1和test2分支未发生变化
> 
> ⑩ 这时的test1和test2可以用git branch -d test1 test2进行删除 

那么，如果存在两个分支master、test，它们对同一个文件readme.md进行修改（master在文件末尾加入master line，test在文件末尾加入test line），这时，当将test分支合并到master分支时Git会报错。

```sh
$ git merge test
Auto-merging readme.md
CONFLICT (content): Merge conflict in readme.md
Automatic merge failed; fix conflicts and then commit the result.
```
使用git status查看状态也会提示文件在两个分支都修改了：

```sh
On branch master
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   readme.md

no changes added to commit (use "git add" and/or "git commit -a")
```
此时，需要手动对master主分支下的readme.md文件进行修改，使用vim readme.md打开文件：

```sh
  1 <<<<<<< HEAD
  2 master line
  3 =======
  4 test line
  5 >>>>>>> test
```
Git将两个分支的readme.md的内容使用<<<<<<<、=======、>>>>>>>区分开来。将内容修改为目标内容，然后git add、git commit即可完成合并。（使用git log --graph可以查看分支合并图）

### 13.分支管理策略
在合并分支时，如果不指定合并模式，Git会默认使用Fast forward模式，在这个模式下，删除分支信息后，会丢失分支信息。我们可以在合并分支时加上--no-ff参数禁用Fast forward模式，此时Git在合并时会生成一个新的commit：

```sh
$ git merge --no-ff -m 'merge test' test
```

使用git log --graph --pretty=oneline分别查看禁用Fast forward和不禁用时的区别：

```sh
# 不禁用时：
$ git merge test
* 06cf0607846548da37edf16cba68a4b46ba95835 (HEAD -> master) add test.md
* 9ee02e9c78cde794543df93ab508fefb2b7f7667 add readme.md

# 禁用时：
% git merge --no-ff -m 'merge test' test
*   2e6de5d4c6f4c919d63b553615181f51411b3a99 (HEAD -> master) merge test
|\  
| * bbfb8e492523ba516e69813ce7b6b09b7ccd3778 test.md
|/  
* 9ee02e9c78cde794543df93ab508fefb2b7f7667 add readme.md

```

### 14.隐藏改变
当正在为一个Web应用程序进行升级时，突然发现当前版本的程序存在Bug，需要优先修复Bug，此时，升级的工作未完成，尚不能commit，而由于升级修改的文件会影响原来版本，怎么办？  
   
使用git stash命令即可隐藏工作区未提交的工作（前提需要将文件添加到仓库，使得Git能够追踪这些文件）。例如，在当前目录下创建test.md文件并添加至仓库，此时使用

```sh
$ git stash
Saved working directory and index state WIP on master: 9ee02e9 add readme.md
```
此时，那些未上传的文件在工作区消失了，可以使用以下命令进行查看：

```sh
$ git stash list
stash@{0}: WIP on master: 9ee02e9 add readme.md
```
Git将相对于最近commit的修改保存至一个类似队列的结构中，可以尝试在使用git stash后再次添加一个文件并使用git stash，然后使用git stash list即可查看到两个stash。   
   
隐藏后如何恢复呢？   
① 使用git stash apply，使用后stash的记录和内容不会删除，可以使用git stash drop进行删除；   
② 使用git stash pop弹出stash，在恢复的同时，删除stash   

### 15.标签管理
添加标签：

```sh
$ git tag v1.0
```
Git默认是为当前分支下的最新commit版本添加标签，如果想为历史提交的版本添加标签，添加上commit id即可：

```sh
$ git tag v1.0 commitid
```
查看所有标签：

```sh
$ git tag
v1.0
```
查看标签详细信息：

```sh
$ git show v1.0
```
删除标签：

```sh
$ git tag -d v1.0
```
本地标签不会自动推送至远程，可以手动推送至远程库：

```sh
$ git push origin v1.0   // 推送标签名v1.0至远程库
$ git push origin --tags // 推送所有标签至远程库
```
删除远程库标签：

```sh
$ git push origin :refs/tags/v1.0  // 删除标签v1.0
```

### 16.多人协作的工作流程
多人协作的工作模式通常是这样：

1. 首先，可以试图用git push origin <branch-name>推送自己的修改；

2. 如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

3. 如果合并有冲突，则解决冲突，并在本地提交；

4. 没有冲突或者解决掉冲突后，再用git push origin <branch-name>推送就能成功！

如果git pull提示no tracking information，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream-to <branch-name> origin/<branch-name>。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

## 三、总结

善用Git的--help可以帮助更好的使用命令，例如：

```sh
$ git --help
$ git pull --help
```

| 命令 | 说明 |
| --- | --- |
| git config —global user.name “< username >" | 配置用户名 |
| git config —global user.email “< user@example.com >" | 配置邮箱 |
| git init | 创建版本库 |
| git add < file > | 添加文件到版本库（Git追踪该文件） |
| git commit -m “< commit message >" | 将改动提交到版本库 |
| git log [--graph --pretty=oneline ...] | 查看版本库日志信息 |
| git reset --hard < version > | 重置版本,HEAD^,HEAD~5,commit id |
| git status | 查看状态 |
| git diff < version >  | 查看版本和当前工作区差异 |
| git checkout — < file > | 恢复至最近commit或add时的状态 |
| git rm < file > | 从版本库删除文件 |
| git remote add < 远程库名 > < address > | 关联远程仓库（远程库名、地址） |
| git push [-u] < 远程库名 > < branch > | 本地库内容推送至远程，第一次加-u（，分支） |
| git pull origin < 远程分支名 >:< 本地分支名 > | 将远程库的更改合并至当前库 |
| git clone < address > | 远程仓库的内容克隆到本地 |
| git branch < branch > | 添加分支 |
| git branch -d < branch > | 删除分支 |
| git branch | 显示所有分支 |
| git checkout < branch > | 切换分支 |
| git checkout -b < branch > | 新建并切换分支 |
| git merge < branch > | 合并分支 |
| git merge --no-ff -m “< commit message >” < branch > | 禁用Fast forward模式合并分支 |
| git stash | 隐藏已跟踪未提交的工作区内容 |
| git stash list | 显示stash |
| git stash apply | 恢复隐藏内容，但不删除 |
| git stash drop | 删除stash记录 |
| git stash pop | 恢复隐藏内容，并删除 |
| git tag < tag name > [< commit id >] | 添加标签 |
| git tag | 显示所有标签 |
| git show < tag name > | 显示tag详细信息 |
| git tag -d < tag name > | 删除标签 |
| git push < 远程库名 > < tag name > | 推送标签至远程库 |
| git push < 远程库名 > --tags | 推送所有标签至远程库 |
| git push < 远程库名 > :refs/tags/< tag name > | 删除远程库标签 |

## 四、实际使用

### 1、在GitHub上，修改他人项目的代码
① 在GitHub上，将项目从他人仓库fork到自己仓库；

② 使用git clone将自己仓库的项目clone到本地；

③ 修改完代码后，将修改push到自己远程仓库；

④ 在GitHub上，创建Pull Request，等待原仓库管理员的审核；

### 2、Pull Request被合并后
最近在GitHub上修改别人的代码，遇到一个问题：我在第二次提交Pull Request时，将第一次已合并的Pull Request再次提交了上去。（毕竟第一次嘛😂   

需要注意的是，在我们的Pull Request被合并时，我们需要做的工作有：

> ① 给fork配置远程库：
> 
> 使用```git remote -v```可以查看远程仓库状态。如果结果中没有原仓库的配置，可以使用```git remote add 自定义名 原仓库地址```进行配置，例如：```git remote add nickname https://github.com/username/repostory.git```
> 
> ② 使用```git fetch nickname```从原仓库fetch分支和提交点到本地，并存储到本地分支nickname/master
> 
> ③ 切换到master分支：```git checkout master```
> 
> ④ 将nickname/master分支合并到本地master：```git merge nickname/master```
> 
> ⑤ 更新到GitHub的fork上：```git push origin master```
> 
> **（原仓库即上游仓库）**


*Mission Complete!*


