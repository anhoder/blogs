# React Native篇---（二）环境搭建与项目初始化

<!-- more -->

*Mission Start!*

以下环境搭建以Mac系统为例，Windows大同小异。

## 环境搭建

### 1、安装Node
首先需要安装Node，Mac下可以使用Homebrew（[安装Homebrew传送门](https://brew.sh/index_zh-cn)）进行安装：

```sh
brew install node
```
可以给npm设置淘宝镜像，可以加速国内下载：

```sh
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```
### 2、安装watchman
watchman是Facebook的开源项目，用于监控、记录文件的改动情况。其安装与Node的安装类似，都是使用Homebrew进行安装：

```sh
brew install watchman
```

### 3、安装React Native命令行工具(react-native-cli)、yarn包管理工具

```sh
npm install -g react-native-cli
```
yarn是Facebook推出的包管理工具，可以加速node模块的下载，安装后即可使用yarn来替代npm命令。

```sh
npm install -g yarn
```
或者，也可以使用Homebrew来进行安装：

```sh
brew install yarn
```
### 4、安装JDK、Android Studio和Xcode
Android开发需要安装Java Development Kit、Android Studio。JDK的安装不赘述，需要注意的是Android Studio安装时需要选择以下几项：

* Android SDK
* Android SDK Platform
* Performance (Intel ® HAXM)
* Android Virtual Device

而iOS的开发需要用到mac系统，在mac系统下的App Store中搜索下载Xcode即可。

## 项目初始化
终端进入目标目录，使用下列命令初始化React Native项目MyProject：

```sh
react-native init MyProject
```
该命令初始化一个最新版的React Native项目，但有时候我们需要初始化一个旧版本的React Native项目，这时可以使用rninit的工具：
> ① 安装rninit
>
> ```sh
> npm install -g rninit
> ```
> 
> ② 初始化指定版本的RN项目：
> 
> ```sh
> rninit init MyProject --source react-native@0.38.0
> ```
> 另外，可以使用npm查看React Native的所有版本：
> 
> ```sh
> npm view react-native versions -json
> ```
> 上述命令初始化一个名为MyProject的、版本为0.38.0的React Native项目。

## Android手机调试
使用真机进行调试，需要手机在“开发者选项”中打开“USB调试”，连接电脑后使用命令查看手机是否连接成功：

```sh
adb devices
```

结果中有设备则连接成功，如

```sh
List of devices attached
48a4deb2	device   
```

接下来，可以将应用安装到手机：

```sh
react-native run-android
```
这条命令会下载相关依赖包（可能时间稍长），然后编译项目得到安装包，并将安装包安装到手机中（手机可能会有提示窗口询问是否安装，选择“是”即可）。   

## iOS手机调试
本人没有iOS设备，所以使用虚拟机进行调试。   

① 使用Xcode打开：项目路径/ios/项目名.xcodeproj   

② 在Xcode中选择虚拟设备，点击运行便会打开对应的虚拟机，并安装项目应用。

## 注意事项
① 项目运行中会打开一个新的命令行窗口launchPackager，并处于运行状态，**请勿关闭它！**   
② 如果调试的手机中已经安装了项目软件（不是首次运行），再次进行调试时并不需要重新编译：
> 首先，使用adb反向代理端口，将电脑的端口反向代理到测试机上：
> 
> ```sh
> adb -s 48a4deb2 reverse tcp:8081 tcp:8081 
> # -s后接手机id（通过adb devices查看）
> # 若电脑只连接了一个设备，可以省略-s参数
> ```
> 然后，执行以下命令运行launchPackager：
> 
> ```sh
> react-native start
> ```

③ 需要重新进行编译的情况：当项目中的android或ios目录下的文件发生改动、电脑分配的IP地址发生变化。   
④ 只改动外部的JS代码可以在App中打开菜单，选择“Reload”项进行更新。   
⑤ 在App的菜单中选择Remote JS Debugging便可以在电脑浏览器中查看控制台的输出内容。   

*Mission Complete!*