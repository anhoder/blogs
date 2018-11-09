# React Native篇---（二）环境搭建与项目初始化

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

*Mission Complete!*