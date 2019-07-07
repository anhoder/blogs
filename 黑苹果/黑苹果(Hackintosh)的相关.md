# 黑苹果(Hackintosh)的相关

<!-- more -->

*Mission Start!*

## 0. 废话

玩黑苹果也有一段时间了，从一开始照着别人的教程一步一步安装的小白，到现在知道一些引导文件的、驱动文件的作用，可以自己组合EFI去驱动电脑，从10.12到10.14，这个过程中学到了很多。    

macOS 10.14正式版开始推送了，写个博客放下我的引导。

## 1. macOS Mojave的引导

关于EFI是什么我就不多说了，合理利用度娘、Google    

**首先说一下电脑的相关信息：**    
> 神船K610系列、大麦2S系列*（两个机型主板相同）* **---机型不同可能不能进入mac系统---**
> CPU: i5 4210m
> 显卡: GT960m*（笔记本的独显驱动不了）*
> 网卡：换了TB买的黑苹果免驱的网卡

OK，直接上我使用的黑苹果引导：
> **升级时用的引导：**链接: https://pan.baidu.com/s/1F8OG6gQ9gnXb0XcXQ7KmeA 密码: 4dwj
> 
> **升级后日常使用的引导：**链接: https://pan.baidu.com/s/1XkfYyDI8cRWUgShoumuugg 密码: ts8m
> 
> *这两个引导的区别在于前面一个更精简，删除了一些升级非必需的引导文件，可以提升升级的成功率（在远景论坛找的，不记得是哪个帖子了，当然也可以自己精简第二个引导用于升级，不过我不想搞了，能用就先用着吧😝）*

## 2. EFI目录结构

说一下efi目录下的各个文件夹的作用：

```shell
$ tree EFI -L 2

EFI
├── APPLE               // 白果用到的启动固件，由系统自动生成，删除后仍会生成，忽略即可
│   ├── EXTENSIONS
│   └── FIRMWARE
├── BOOT                
│   └── BOOTX64.efi     // UEFI必需引导文件
└── CLOVER
    ├── ACPI            // 必需
    ├── CLOVERX64.efi   // 四叶草Clover引导文件
    ├── config.plist    // Clover引导配置文件
    ├── drivers64UEFI   // 硬盘文件系统的驱动文件
    ├── kexts           // 硬件驱动文件
    ├── misc            
    ├── themes          // Clover引导界面的主题，可自定义
    └── tools           // 必需
```

kexts目录下存放的驱动可以根据自己的需求增删，驱动文件的具体作用直接输入文件名百度就行了。😆

<span style="color:red;">**在安装黑苹果时，ESP分区的大小需要不小于200M，否则会报错。**</span>

*Mission Complete!*



