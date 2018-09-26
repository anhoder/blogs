# Redis内存数据库基本操作

*Mission Start!*

```
redis-server [redis.conf配置文件路径] // 启动Redis
redis-cli // 连接Redis
```

## 一、数据库操作
```
select dbindex          // 选择数据库，使用config get databases可以获取Redis配置的数据库个数，默认为16个(0-15)
dbsize                  // 返回当前数据库中键值的数量
flushdb                 // 删除当前数据库中所有的键值
flushall				// 删除所有数据库中所有的键值
```

## 二、对键key的操作
```
exists key              // 判断key是否存在
del key1 key2...        // 删除key1、key2...
type key                // 返回key对应值的类型
keys pattern            // 返回所有匹配的键，pattern为匹配格式
randomkey               // 随机返回一个键
rename key1 key2		// 将key1的键名改为key2，值不改变	
expire key seconds      // 为key设置过期时间(秒)
ttl key                 // 返回key的剩余过期秒数
move key dbindex        // 将key移动到dbindex数据库下
```

## 三、对值--String的操作
```
set key value           // 设置key对应的值为value
get key         		// 根据key获取对应的值
mset key1 value1 ...	// 一次设置多个键值
mget key1 value1 ...	// 一次获取多个键值
incr key				// key的值加上1
decr key				// key的值减去1
incrby key integer      // key的值加上integer(整数)
decrby key integer      // key的值减去integer(整数)
append key value        // key的值后追加value
substr key start end    // 返回截取到的key的值(从start开始，end结束)
```

## 四、对值--List(双向链表)的操作
```
lpush key value         // 向key对应的链表头部添加value
rpush key value         // 向key对应的链表尾部添加value
lpop key                // 删除key对应链表的头部元素
rpop key                // 删除key对应链表的尾部元素
llen key                // 返回key对应链表的长度（不存在返回0；若key对应的不是链表，返回错误）
lrange key start end    // 返回key对应链表的start~end内的元素
ltrim key start end     // 截取key对应链表，保留指定区间内的元素
lset key index value    // 设置key对应链表下标是index的值为value
lrem key count value    // 从key对应链表中删除count个值为value的元素 (若count为0，则删除全部值为value的元素)
```

## 五、对值--Set(集合)的操作
```
sadd key member ...     // 向key对应集合中添加成员member
smembers key            // 返回key对应集合的所有元素(无序)
srem key member ...     // 移除key对应集合中值为member的元素，成功则返回1
smove key1 key2 member  // 将key1对应集合中值为member的元素移除，并添加至key2对应的集合
scard key               // 返回key对应集合的元素个数
sismember key member    // 判断key对应集合中是否有member
sinter key1 key2 ...    // 返回key1,key2...对应集合的交集
sunion key1 key2 ...    // 返回key1,key2...对应集合的并集
sdiff key1 key2 ...     // 返回key1,key2...对应集合的差集
sinterstore new_key key1 key2   //将key1,key2...对应集合的交集保存至new_key对应的集合
sunionstore new_key key1 key2   //将key1,key2...对应集合的并集保存至new_key对应的集合
sdiffstore new_key key1 key2    //将key1,key2...对应集合的差集保存至new_key对应的集合
```

## 六、对值--SortSet(排序集合)的操作
```
zadd key score id   // 向key对应排序集合中添加成员(键为key，权为score，值为value)，如：zadd stu_age 18 1
zincrby key int id  // 给值为id的元素的权加int
zcard key           // 返回排序集合中元素个数
zscore key id       // 返回排序集合中值为id的元素的权
zrange key start end    // 返回key对应排序集合的指定区间的元素的值id（正序，根据权value进行排序）
zrevrange key start end // 返回key对应排序集合的指定区间的元素的值id（逆序，根据权value进行排序）
zrangebyscore key min max   // 返回排序集合中权score在[min,max]之间的元素的值id
zrank key id            // 返回值为id的元素在集合中的下标(正序排序)
zrevrank key id         // 返回值为id的元素在集合中的下标(逆序排序)
zcount key min max      // 返回在区间[min,max]的元素个数
zrem key id         // 根据值id删除指定元素，返回1表示成功，返回0表示元素不存在       
zremrangebyrank key min max //删除排序集合中排名在[min,max]中的元素
zremrangebyscore key min max    //删除排序集合中权在[min,max]中的元素
```

## 七、对值--Hash的操作
```
hset key field value
hget key field
hmset key field1 value1
hmget key field1 field2
hincrby key field
hexists key field
hdel key field
hkeys key 
hlen key
hvals key
hgetall key
```

*Mission Complete!*

