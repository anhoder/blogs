# 面试问题之Redis

<!-- more -->

> 博客内容为[PHP-Interview-QA](https://github.com/colinlet/PHP-Interview-QA)读后笔记

## Redis特点

* 支持数据持久化
* 支持key-value类型数据及其他数据类型
* 支持主从结构

## 数据类型

* String字符串
* List链表
* Hash哈希表
* Set集合
* ZSet有序集合

## Redis与MemCache

* Redis支持丰富的数据类型，MemCache支持k-v数据
* Redis支持数据持久化，MemCache不支持
* Redis支持事务，MemCache用cas维持数据一致性

## Redis事务

* MULTI: 后续的命令进入事务队列
* DISCARD: 取消一个事务，清空事务队列
* EXEC: 执行事务
* WATCH: 用于在事务开始之前监视任意数量的键：当调用 EXEC 命令执行事务时，如果任意一个被监视的键已经被其他客户端修改了，那么整个事务不再执行，直接返回失败。

## 持久化策略

* RDB快照持久化：将所有数据写入硬盘，BGSAVE
* AOF持久化：以追加形式写入，将写过的命令保存到磁盘