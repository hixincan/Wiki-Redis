# Redis

## 简介

官网：https://redis.io/

中文：

* https://www.redis.net.cn/ （推荐）
* http://www.redis.cn/



Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作**数据库**、**缓存**和**消息中间件**。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询，如 [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)（集群）提供高可用性（high availability）。

## 安装

<<参考 Linux 笔记>>

## 远端连接

默认只允许本机连接，如果开发测试，可以开放连接（不安全），设置如下

* 开放防火墙和安全组的端口
* 更改 redis.config 配置
    * 注释 bind 127.0.0.1 
    * 保护模式设置为 no

![image-20210529164905576](JavaFramework.assets/image-20210529164905576.png)



## 测试性能

参数参考：https://www.runoob.com/redis/redis-benchmarks.html

| 序号 | 选项                      | 描述                                       | 默认值    |
| :--- | :------------------------ | :----------------------------------------- | :-------- |
| 1    | **-h**                    | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**                    | 指定服务器端口                             | 6379      |
| 3    | **-s**                    | 指定服务器 socket                          |           |
| 4    | **-c**                    | 指定并发连接数                             | 50        |
| 5    | **-n**                    | 指定请求数                                 | 10000     |
| 6    | **-d**                    | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| 7    | **-k**                    | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**                    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**                    | 通过管道传输 <numreq> 请求                 | 1         |
| 10   | **-q**                    | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **--csv**                 | 以 CSV 格式输出                            |           |
| 12   | ***-l\*（L 的小写字母）** | 生成循环，永久执行测试                     |           |
| 13   | **-t**                    | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | ***-I\*（i 的大写字母）** | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

```bash
# 测试：100个并发连接 100000请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

## 基础知识

Redis 有16个数据库

默认在 index 0 号数据库中，可以通过 `select` 命令切换数据库，且数据库之间的数据是隔离的。

![image-20210527173507149](JavaFramework.assets/image-20210527173507149.png)	

查看当前数据库的所有键值 `keys *`

清除当前数据库 `flushdb`
清除全部数据库的内容 `flushall`

==Redis 命令不区分大小写==

Redis 命令文档：http://redis.cn/commands.html

> Redis 是单线程的！

明白Redis是很快的，官方表示，Redis是基于内存操作，CPU不是Redis性能瓶颈，Redis的瓶颈是根据
机器的内存和网络带宽，既然可以使用单线程来实现，就使用单线程了！

Redis 是C 语言写的，官方提供的数据为 100000+ 的QPS，完全不比同样是使用 key-value的
Memecache差！  

**为什么单线程还那么快？**

1、误区1：高性能的服务器一定是多线程的？
2、误区2：多线程（CPU上下文会切换！）一定比单线程效率高！

==注意==TODO 参考更多文献

先去CPU>内存>硬盘的速度要有所了解！
核心：redis 是将所有的数据全部放在内存中的，所以说使用单线程去操作效率就是最高的，多线程
（CPU上下文会切换：耗时的操作！！！），对于内存系统来说，如果没有上下文切换效率就是最高
的！多次读写都是在一个CPU上的，在内存情况下，这个就是最佳的方案 ！

>Redis的内存性能

合理的使用数据结构是优化性能，如内存性能的方式之一

## 五大数据类型  

### Redis-Key 

```shell
127.0.0.1:6379[2]> set name xincan #设置 key-value
OK
127.0.0.1:6379[2]> keys * # 查看当前数据库的所有key
1) "name"
127.0.0.1:6379[2]> move name 1 #从当前数据库移除，并且移动到指定的序号的数据库
(integer) 0
127.0.0.1:6379[2]> exists name #判断当前数据库的key是否存在
(integer) 1
127.0.0.1:6379[2]> select 1 #切换到 1号数据库
OK
127.0.0.1:6379[1]> expire foo 4 #设置key过期时间，单位秒
(integer) 1
127.0.0.1:6379[1]> ttl foo #查看key的剩余有效时间
(integer) 1
127.0.0.1:6379[1]> ttl foo
(integer) -2
127.0.0.1:6379[1]> get foo
(nil)
127.0.0.1:6379> type age #查看数据类型
string
```

### String（字符串+数字）

* 常规处理  strlen, append
* 对**数字值**的处理 incr decr, incrby, decrby
* 截取字符串 getrange
* 替换 setrange
* setex, setnx, mset, mget, msetex（原子操作）, msetnx（原子操作）

```shell
# setex (set with expire) # 设置过期时间
# setnx (set if not exist) # 不存在则设置 （在分布式锁中会常常使用！）
127.0.0.1:6379> setex key3 30 "hello" # 设置key3 的值为 hello,30秒后过期
OK
127.0.0.1:6379> ttl key3
(integer) 26
127.0.0.1:6379> get key3
"hello"
127.0.0.1:6379> setnx lock asdfasdf
(integer) 1 # 成功
127.0.0.1:6379> setnx lock asd
(integer) 0 # 失败
127.0.0.1:6379> get lock
"asdfasdf"
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3 #批量设置
OK
127.0.0.1:6379> keys *
1) "k1"
2) "lock"
3) "k3"
4) "k2"
127.0.0.1:6379> mget k1 k2 #批量读取
1) "v1"
2) "v2"
127.0.0.1:6379> msetnx k1 v1 k5 v5 # msetnx 是一个原子性操作
(integer) 0 #失败
```

* 设置对象

在key上做设计，示例 {语义key}:{id}:{业务key}

应用场景一：保存用户对象（mset 快速设置）

user:{id}:name

user:{id}:phone

应用场景二：记录公众号浏览量

article:{id}:views

* getset

```shell
getset # 先get然后在set
127.0.0.1:6379> getset db redis # 如果不存在值，则返回 nil
(nil)
127.0.0.1:6379> get db
"redis 
127.0.0.1:6379> getset db mongodb # 如果存在值，获取原来的值，并设置新的值
"redis" # 返回上一次的值
127.0.0.1:6379> get db
"mongodb"
```

### List

==在redis里面，我们可以把list玩成 ，栈、队列、阻塞队列！==

list 命令以 **L** 开头（少部分 R 开头）

```shell
# lpush 从列表左端插入元素
# rpush 从列表右端插入元素
# lrange 读取列表
127.0.0.1:6379> flushall
OK
127.0.0.1:6379> lpush list 1 2 3 4 #从左依次插入 4 个元素
(integer) 4
127.0.0.1:6379> get list # 不能通过get读取list
(error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> lrange list 0 -1 # 得用range读取list
1) "4"
2) "3"
3) "2"
4) "1"
127.0.0.1:6379> rpush list 5 6 7 #从右依次插入 3 个元素
(integer) 7
########################################
# lpop 从列表左端移除元素
# rpop 从列表右端移除元素
127.0.0.1:6379> lpop list 1 #从左移除1个元素
1) "4"
127.0.0.1:6379> rpop list 2 #从右移除 2 个元素
1) "7"
2) "6"
########################################
# lindex 根据下标获取列表值
127.0.0.1:6379> lpush list s1 s2 s3
(integer) 3
127.0.0.1:6379> lindex list 1
"s2"
########################################
# lrem 精确根据列表值来移除
127.0.0.1:6379> lrange list 0 -1
1) "a2"
2) "a2"
3) "a2"
4) "a5"
5) "a4"
6) "a3"
7) "a1"
127.0.0.1:6379> lrem list 4 a2 #移除4个a2，实际上只有3个a2，此操作能成功
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "a5"
2) "a4"
3) "a3"
4) "a1"
########################################
# ltrim 将 list 裁剪
127.0.0.1:6379> lpush list a1 a2 a3 a4
(integer) 4
127.0.0.1:6379> ltrim list 1 2 #按范围截取
OK
127.0.0.1:6379> lrange list 0 -1
1) "a3"
2) "a2"
########################################
# rpoplpush 移除列表的最后一个元素，并将其放入另一个列表的头部
127.0.0.1:6379> lpush list1 s1 s2 s3
(integer) 3
127.0.0.1:6379> rpoplpush list1 list2
"s1"
127.0.0.1:6379> lrange list2 0 -1
1) "s1"
127.0.0.1:6379> lrange list1 0 -1
1) "s3"
2) "s2"
########################################
# lset 将列表下标的值替换为新的值
127.0.0.1:6379> lpush list s1 s2 s3 s4
(integer) 4
127.0.0.1:6379> lset list 1 s2plus # 更新下标1的值为s2plus
OK
127.0.0.1:6379> lrange list 0 -1
1) "s4"
2) "s2plus"
3) "s2"
4) "s1"
127.0.0.1:6379> lset list 99 test # 超出数组长度报错
(error) ERR index out of range
########################################
# linsert 将新值插入到列表指定值（若重复则取第一个）的前或后的位置
127.0.0.1:6379> lpush list s1 s2 s3 s1 s2 s3
(integer) 6
127.0.0.1:6379> lrange list  0 -1
1) "s3"
2) "s2"
3) "s1"
4) "s3"
5) "s2"
6) "s1"
127.0.0.1:6379> linsert list before s1 s1.5
(integer) 7
127.0.0.1:6379> lrange list  0 -1
1) "s3"
2) "s2"
3) "s1.5"
4) "s1"
5) "s3"
6) "s2"
7) "s1"
```



> 小结

在两边插入或者改动值，效率最高 

消息队列 （Lpush Rpop）， 栈（ Lpush Lpop）



### Set

Set 无序不重复集合；

命令以 **s** 开头

```bash
# sadd 没有key则创建，有key则追加
127.0.0.1:6379> sadd myset s1 s2 # 添加值
(integer) 2
127.0.0.1:6379> smembers myset # 查看set集合
1) "s1"
2) "s2"
127.0.0.1:6379> sismember myset s1 #判断 set 集合的值是否存在
(integer) 1 # 存在返回1
127.0.0.1:6379> sismember myset s3
(integer) 0 # 不存在返回0
127.0.0.1:6379> scard myset # 获取 set内容中值的个数
(integer) 2
127.0.0.1:6379> srem myset s1 # 移除set值
(integer) 1
127.0.0.1:6379> smembers myset
1) "s2"
########################################
# srandmember 获取随机值，模拟抽奖
127.0.0.1:6379> sadd myset s1 s2 s3 s4 s5 s6
(integer) 6
127.0.0.1:6379> srandmember myset 1
1) "s2"
127.0.0.1:6379> srandmember myset 1
1) "s5"
########################################
# spop 随机删除
127.0.0.1:6379> sadd myset s1 s2 s3
(integer) 3
127.0.0.1:6379> spop myset 1
1) "s3"
########################################
# 集合运算：差集、交集、并集
127.0.0.1:6379> sadd myset s1 s2 s3
(integer) 3
127.0.0.1:6379> sadd myset2 s3 s4 s5
(integer) 3
127.0.0.1:6379> sdiff myset myset2 #差集
1) "s1"
2) "s2"
127.0.0.1:6379> sinter myset myset2 #交集
1) "s3"
127.0.0.1:6379> sunion myset myset2 #并集
1) "s5"
2) "s3"
3) "s2"
4) "s1"
5) "s4"
```



### Hash

即Map集合，key - map，值是map集合（==本质和 String 没有太大的区别==），hash key 可以理解是数据作用域

命令以 **h** 开头

```shell
127.0.0.1:6379> hset user:1 name xincan
(integer) 1
127.0.0.1:6379> hset user:1 name xincan age 18 # 等价于 hmset
(integer) 1
127.0.0.1:6379> hget user:1 name
"xincan"
127.0.0.1:6379> hgetall user:1 #将hash的 field-value，平铺展示
1) "name"
2) "xincan"
3) "age"
4) "18"
########################################
# hdel 删除hash的key，没有key也即没有了value
127.0.0.1:6379> hdel user:1 name age #支持批量删除
(integer) 2
127.0.0.1:6379> hgetall user:1
(empty array)
########################################
# hlen 获取hash的长度
127.0.0.1:6379> hlen user:1
(integer) 0 # 返回hash的key长度
########################################
# hexists 判断hash的field是否存在
127.0.0.1:6379> hexists user:1 name
(integer) 1 # 1为存在
########################################
# hkeys 获取hash的key集合
# hvals 获取hash的value集合
127.0.0.1:6379> hkeys user:1
1) "name"
2) "age"
127.0.0.1:6379> hvals user:1
1) "xincan"
2) "18"
########################################
# 有类似string的操作
127.0.0.1:6379> hset user:1 views 1
(integer) 1
127.0.0.1:6379> hincrby user:1 views 2 #步长
(integer) 3
127.0.0.1:6379> hincrby user:1 views -1 #步长，可以为负
(integer) 2
127.0.0.1:6379> hsetnx user:2 views 0 #用于初始化，或应用于锁
(integer) 1 # 1表示成功
127.0.0.1:6379> hsetnx user:2 views 0
(integer) 0
```

hash变更的数据 user name age,尤其是是用户信息之类的，经常变动的信息！ hash 更适合于对象的
存储，String更加适合字符串存储！  



### Zset（有序集合）

在set的基础上，增加了一个 score（权重/顺序） `zeset k1 score v1`

命令以 **z** 开头

```shell
127.0.0.1:6379> zadd myset 10 s1 5 s2 8 s3 4 s4
(integer) 4
127.0.0.1:6379> zrange myset 0 -1 #默认按 score 升序排序
1) "s4"
2) "s2"
3) "s3"
4) "s1"
########################################
# zrangebyscore 获取一段范围
127.0.0.1:6379> zrangebyscore myset 4 9 withscores # withscores 把分值也返回出来
1) "s4"
2) "4"
3) "s2"
4) "5"
5) "s3"
6) "8"
########################################
# zrem 移除某个元素
127.0.0.1:6379> zrem myset s2
(integer) 1
127.0.0.1:6379> zrange myset 0 -1
1) "s4"
2) "s3"
3) "s1"
127.0.0.1:6379> zcard myset # 返回集合大小
(integer) 3
########################################
# 反向（降序）
127.0.0.1:6379> zrange myset 0 -1 withscores
1) "s4"
2) "4"
3) "s3"
4) "8"
5) "s1"
6) "10"
127.0.0.1:6379> zrevrange myset 0 -1 withscores #降序，且指定范围
1) "s1"
2) "10"
3) "s3"
4) "8"
5) "s4"
6) "4"
########################################
# zcount 范围间的数量
127.0.0.1:6379> zcount myset 5 9 #在score范围内的个数
(integer) 1
```

## 三种特殊数据类型

### geospatial 地理位置

城市经纬度：

* http://www.hao828.com/chaxun/zhongguochengshijingweidu/

>geoadd

有效的经度从-180度到180度。

有效的纬度从-85.05112878度到85.05112878度。

```shell
127.0.0.1:6379> geoadd china:city 116.408 39.904 beijing(integer) 1127.0.0.1:6379> geoadd china:city 121.445 31.213 shanghai(integer) 1127.0.0.1:6379> geoadd china:city 117.246 39.117 tianjin(integer) 1127.0.0.1:6379> geoadd china:city 106.549 29.581 chongqing(integer) 1127.0.0.1:6379> geoadd china:city 120.165 30.319 hangzhou(integer) 1
```

> geopos

```shell
127.0.0.1:6379> geopos china:city hangzhou1) 1) "120.16499966382980347"   2) "30.31899997732214302"
```

> geodist

坐标之间的距离

单位：

* m 表示单位为米
* km 表示单位为千米

```shell
127.0.0.1:6379> geodist china:city beijing shanghai km"1068.2320" # 直线的距离
```

> georadius

附近的人，通过半径搜索

```shell
127.0.0.1:6379> georadius china:city 111 31 500 km #以指定经纬度，在 500 km 半径内的城市信息1) "chongqing"127.0.0.1:6379> georadius china:city 111 31 1000 km1) "chongqing"2) "hangzhou"3) "shanghai"127.0.0.1:6379> georadius china:city 111 31 5000 km withcoord withdist count 31) 1) "chongqing"   2) "455.6402"  # withdist 直线距离   3) 1) "106.54900163412094116"      2) "29.58100070345364685"2) 1) "hangzhou"   2) "879.9037"   3) 1) "120.16499966382980347"      2) "30.31899997732214302"3) 1) "shanghai"   2) "994.6200"   3) 1) "121.44499808549880981"      2) "31.213001199663303"
```

> georadiusbymember

等同于 georadius，区别是通过已有元素 key member 来定位

> geohash

```shell
127.0.0.1:6379> geohash china:city beijing chongqing1) "wx4g0bm9xh0" #经纬度映射成 11 位字符串2) "wm7b2949gd0"
```

> 删除坐标

geo 底层是 zset 实现的，==geo的命令很少，是因为只是对基础数据结构做的扩展==

```bash
127.0.0.1:6379> zrange china:city 0 -11) "chongqing"2) "hangzhou"3) "shanghai"4) "tianjin"5) "beijing"127.0.0.1:6379> zrem china:city chongqing(integer) 1
```

### Hyperloglog 基数

> 什么是基数？

A {1,3,5,7,8,7}
B {1,3,5,7,8}
？？？基数（不重复的元素） = 5，可以接受误差

> 简介

Redis 2.8.9 版本就更新了 Hyperloglog 数据结构
Redis Hyperloglog 基数统计的算法！

特征：

* 占用的内存是固定，2^64 不同的元素的基数，只需要废 12KB内存
* 官方提供误差数据：0.81% 错误率  （前提是业务允许容错）

**应用场景：统计网页的 UV**
传统的方式， set 保存用户的id，然后就可以统计 set 中的元素数量作为标准判断 。缺点：这个方式如果保存大量的用户id，会占用大量内存。如果目的只是计数，set 方案就比较占用内存。

> 命令

命令以 **pf** 开头

```bash
127.0.0.1:6379> PFADD mykey e1 e2 e3 e4 e5 e6 e7 e8 e9(integer) 1127.0.0.1:6379> PFADD mykey2 e1 e2 e3 e4 e5 e10 e11(integer) 1127.0.0.1:6379> PFMERGE newkey mykey mykey2 #合并mykey, mykey2 ==> newkeyOK127.0.0.1:6379> PFCOUNT newkey #查看数量(integer) 11
```

### Bitmap 位存储

> 位存储

Bitmap 位图，是一种数据结构，用二进制位记录，只有 0 和 1 两种状态。

适用场景，如累计签到情况

```bash
127.0.0.1:6379> SETBIT checkin 0 0 #第一天，未签到，0表示无，1表示有(integer) 0127.0.0.1:6379> SETBIT checkin 1 1(integer) 0127.0.0.1:6379> SETBIT checkin 2 1(integer) 0127.0.0.1:6379> SETBIT checkin 3 0(integer) 0127.0.0.1:6379> SETBIT checkin 4 1(integer) 0127.0.0.1:6379> SETBIT checkin 5 0(integer) 0127.0.0.1:6379> SETBIT checkin 6 1(integer) 0127.0.0.1:6379> getbit checkin 2 #查看某天签到情况(integer) 1127.0.0.1:6379> getbit checkin 3(integer) 0127.0.0.1:6379> bitcount checkin #统计签到天数(integer) 4
```

## 事务

> 与Mysql事务的区别

一、事务的ACID特性：

  1）原子性——不被打断

  2）一致性

  3）隔离性

  4）持久性

> Redis事务

Redis 事务本质：一组命令的集合！ 一个事务中的所有命令都会被序列化，在事务执行过程中，会按
照顺序执行！

特征：

* 一次性——执行完即结束
* 顺序性
* 排他性——不被打断
* 没有原子性——单条命令有原子性，但事务上没有原子性，允许运行时异常发生
* 没有隔离级别的概念：所有的命令在事务中，并没有直接被执行！只有发起执行命令的时候才会执行

Redis的事务：

* 开启事务（multi）
* 命令入队（......）
* 取消事务（discard）中途可以取消事务
* 执行事务（exec）  

> 对照代码理解

```shell
127.0.0.1:6379> MULTI # 开启事务OK127.0.0.1:6379(TX)> set k1 v1QUEUED # 命令入队127.0.0.1:6379(TX)> set k2 v2QUEUED127.0.0.1:6379(TX)> get k2QUEUED127.0.0.1:6379(TX)> exec # 真正执行，事务结束1) OK2) OK3) "v2"
```

> 编译型异常（代码有问题！ 命令有错！） ，事务中所有的命令都不会被执行！  

```bash
127.0.0.1:6379> multiOK127.0.0.1:6379> set k1 v1QUEUED127.0.0.1:6379> set k2 v2QUEUED127.0.0.1:6379> set k3 v3QUEUED127.0.0.1:6379> getset k3 # 错误的命令(error) ERR wrong number of arguments for 'getset' command127.0.0.1:6379> set k4 v4QUEUED127.0.0.1:6379> set k5 v5QUEUED127.0.0.1:6379> exec # 执行事务报错！(error) EXECABORT Transaction discarded because of previous errors.127.0.0.1:6379> get k5 # 所有的命令都不会被执行！(nil)
```

>运行时异常（1/0）， 如果事务队列中存在语法错误，那么执行命令的时候，其他命令是可以正常执行
>的，错误命令抛出异常！  

```bash
127.0.0.1:6379> set k1 "v1"OK127.0.0.1:6379> multiOK127.0.0.1:6379> incr k1 # 会执行的时候失败！QUEUED127.0.0.1:6379> set k2 v2QUEUED127.0.0.1:6379> set k3 v3QUEUED127.0.0.1:6379> get k3QUEUED127.0.0.1:6379> exec1) (error) ERR value is not an integer or out of range # 虽然第一条命令报错了，但是依旧正常执行成功了！2) OK3) OK4) "v3"127.0.0.1:6379> get k2"v2"127.0.0.1:6379> get k3"v3"
```



### 乐观锁

> 什么是乐观锁

悲观锁：（性能差）

* 认为什么时候都会出问题，无论做什么都会加锁

乐观锁：（推荐）

* 很乐观，认为什么时候都不会出问题，所以不会上锁！ 更新数据的时候去判断一下，在此期间是否
    有人修改过这个数据
* 获取version
* 更新的时候比较 version ，如果version正确才会提交

> watch命令监视，watch+multi 实现乐观锁

正常执行

```bash
127.0.0.1:6379> set money 100OK127.0.0.1:6379> watch moneyOK127.0.0.1:6379> multiOK127.0.0.1:6379(TX)> decrby money 10QUEUED127.0.0.1:6379(TX)> exec # 事务执行完，释放 watch1) (integer) 90
```

测试修改值

```bash
127.0.0.1:6379> watch money # watch后，仅允许在事务中修改OK127.0.0.1:6379> incrby money 10 # 修改被watch的值(integer) 30127.0.0.1:6379> multiOK127.0.0.1:6379(TX)> incrby money 20 QUEUED127.0.0.1:6379(TX)> exec # 此时比较 watch 值是否发生变化，若变化则不执行事务(nil) # 空 表示事务失败，不管成功与否都会释放 watch
```

其他

```bash
unwatch 是取消所有监视？？watch key 可以反复执行
```



## Jedis

我们要使用 Java 来操作 Redis，知其然并知其所以然，授人以渔！ 学习不能急躁，慢慢来会很快！

> 什么是Jedis 是 Redis 官方推荐的 java连接开发工具！ 使用Java 操作Redis 中间件！如果你要使用
> java操作redis，那么一定要对Jedis 十分的熟悉！  





## Redis 持久化



## Redis 发布订阅

订阅者

```bash
27.0.0.1:6379> SUBSCRIBE live #订阅live频道Reading messages... (press Ctrl-C to quit)1) "subscribe"2) "live"3) (integer) 1#等待推送的信息1) "message" #消息2) "live"3) "hello"
```

发布者

```bash
127.0.0.1:6379> publish live hello(integer) 1
```









## 应用场景

==TODO 思考：Redis 的优势==

* 高并发读写场景
* 持续性变动
* 数据结构易变，跟着业务/运营指标而变化，不能提前设计好表结构



> 单点登录

单点登录就是**在多个系统中，用户只需一次登录，各个系统即可感知该用户已经登录。**



> 关注数、粉丝数、在看数、获赞数、播放数、阅读数

可通过 String 数字值实现数据统计



> 消息队列

可通过 List 实现队列



> 共同关注；推荐好友

可通过 Set 集合运算得到



> 保存用户的信息和经常变动的信息

可通过 Hash 存储对象



> ​	B站排行榜、消息等级、成绩表、工资表

可通过 Zset 实现带权重的业务



> 附近的人、距离你多远

可通过 geospatial 实现，以你为中心的半径来搜索



> 签到

可通过 Bitmap 实现



> 公众号推送；简单聊天室

可通过发布订阅来实现推送场景

简单场景：

* 消息中心
* 聊天室
* 关注系统

复杂场景，需要使用专业的消息中间件 MQ 消息队列（==TODO== Kafka、==TODO== RabbitMQ）





## 语言库

jedis：java 的 redis 库











