# 安装linux(CentOS)的步骤及遇到的问题

<!-- more -->

以前都是使用腾讯云服务器来学习linux的，但是由于最近课程需要，而我的服务器linux版本又达不到要求，所以打算在电脑上安装我的第三个系统（已经装了win10和黑苹果）——CentOS（属于Red Hat系列）。

<h2>第一步 下载CentOS的镜像包</h2>

在CentOS官网下载镜像包（<a style="color: red;" href="https://www.centos.org/download/">https://www.centos.org/download/</a>）

官网有三个不同的版本：<b>DVD ISO、Everything ISO、Minimal ISO</b>

<blockquote><b>① DVD ISO</b>：系统标准安装包，一般使用这个<br>

<b>② Everything ISO</b>：集成了全部软件的镜像<br>

<b>③ Minimal ISO</b>：最小的安装包，只有很基本的操作系统，功能最少</blockquote>

<h2>第二步 将镜像写入到U盘</h2>

准备好一个U盘、刚下好的CentOS镜像以及UltraISO制作工具

使用UltraISO将CentOS镜像写入到U盘（UltraISO使用方法自行百度）

<h2>第三步 设置BIOS启动项</h2>

① 重启电脑，开机出现Logo时按特定的键进入BIOS（不同品牌电脑方式不同，我的是按ESC）<br>

② 选择U盘启动

<h2>第四步 进行配置安装</h2>

电脑启动后，选择Install CentOS，然后便会进入centos的图形化安装界面，你可以根据自己的需求选择相对应的功能模块进行安装。

<h3 style="color: red;">遇到的问题：</h3>

<blockquote style="color: red;">选择Install CentOS后，出现几项OK代码后，就停下不动了，经过几分钟就出现很多代码，最后进入了一个奇怪的命令行模式，上面就是一堆报错。</blockquote>

<h3 style="color:green;">解决方案：</h3>

> （1）在install centos界面，光标移至Install CentOS行，按下Tab键或E键，进入系统引导命令行，其中有一行显示：

> ```sh
> vmlinuz initrd=initrd.img inst.stage2=hd:LABLE...
> ```

> 将其修改为

> ```sh
> vmlinuz initrd=initrd.img dd linux quiet
> ```

> 按下Ctrl+X开始执行，然后系统会列出你的设备挂载信息，找到你的U盘（LABEL一般为CentOS开头），并记住第一列的值（例如：sdb4）
> （2）重启电脑，同样在Install CentOS按下Tab键或E键，将

> ```sh
> vmlinuz initrd=initrd.img inst.stage2=hd:LABLE...
> ```

> 修改为：

> ```sh
> vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdb4(修改为你记住的U盘第一列的信息) quiet
> ```

Ctrl+X开始执行，然后就会进入CentOS的安装界面了</blockquote>