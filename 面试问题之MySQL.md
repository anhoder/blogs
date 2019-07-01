# 面试问题之MySQL

> 博客内容为[PHP-Interview-QA](https://github.com/colinlet/PHP-Interview-QA)读后笔记

## left join、inner join、right join、full join

* left join: 用左边所有的数据在右边中查找匹配的值，如果没有匹配的，用null代替
* right join: 与left join类似
* inner join: 左右两边匹配到的行
* full join: 左边、右边的所有数据，未匹配到的用null代替

## union、union all

* union: 连接两个结果集，并去重
* union all: 不去重

## MySQL常用函数

* floor向下取整、ceil向上取整、round四舍五入、rand随机数、abs绝对值
* concat连接字符串、length字符串长度
* now当前时间、curdate当前日期、unix_timestamp转时间戳、from_unixtime时间戳转格式化日期
* password、md5加密
* format数字格式化

## 锁

* 读锁（共享锁）：同一时刻可以同时读取同一个资源，阻塞写
* 写锁（排他锁）：阻塞其他写和读

共享锁和排他锁都属于悲观锁（悲观锁认为此操作会出现数据冲突，所以在进行每次操作时都要通过获取锁才能进行对相同数据的操作），乐观锁则认为这次的操作不会导致冲突，在操作数据时，并不进行任何其他的特殊处理（也就是不加锁），而在进行更新后，再去判断是否有冲突了。

例如：

```sql
# 1.查询出商品信息

select (status,version) from t_goods where id=#{id}

# 2.根据商品信息生成订单

# 3.修改商品status为2

update t_goods 

set status=2,version=version+1

where id=#{id} and version=#{version};
```
> 悲观锁由数据库内部实现
> 乐观锁则是在外部需要手动实现

* 表锁：开销小，锁定整张表
* 行级锁：开销大，锁定一行

## 事务隔离级别

* 未提交读：事务中未提交的修改，对其他事务可见
* 提交读：事务中未提交修改仅自己可见，但两次查询可能得到不同结果（解决脏读：一个线程中的事务读取到了另外一个线程中未提交的数据）
* 可重复读：MySQL默认，同一事务多次读取结果一致（解决不可重复读：一个线程中的事务读取到了另外一个线程中提交的update的数据）
* 可串行化：强制事务串行执行（解决幻读：一个线程中的事务读取到了另外一个线程中提交的insert的数据）

## InnoDB、MyISAM

* InnoDB: MySQL默认存储引擎，支持事务、行级锁、外键，支持崩溃后安全恢复，性能优秀
* MyISAM: 支持全文索引、表锁，不支持事务、外键、崩溃后安全恢复

其他存储引擎：Blackhole、Memory

## 索引

通过二叉查找树等数据结构来提升无序数据表的查询效率。

> 优势：提升数据的扫描量、避免创建临时表、提升查询速度
> 劣势：增删改需要额外维护索引，写入速度降低，索引会导致占用更多磁盘

* 普通索引
* 唯一索引
* 主键索引
* 外建索引
* 组合索引：应遵循最左前缀原则
* 全文索引

按物理存储分类：聚簇索引、非聚簇索引

* 聚簇索引：叶节点就是数据节点
* 非聚簇索引：叶节点仍是索引节点，存在指向数据块的指针

