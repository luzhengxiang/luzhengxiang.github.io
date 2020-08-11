---
title: redis
date: 2020-03-18 12:03:20
tags: redis
---
# redis 的基础数据类型
## String 字符串
### 存储类型
可以用来存储字符串、整数、浮点数。
### 操作命令
```
SET runoobkey redis 设置单个值
mset aaa 2673 bbb 666 设置多个值
setnx aaa   设置值，如果 key 存在，则不成功。 

incr aaa        (整数)值递增+1
incrby aaa 100  (整数)值递增+100

decr aaa        (整数)值递减-1
decrby aaa 100  (整数)值递减-100

浮点数增量
set f 2.6
incrbyfloat f 7.3

获取多个值
mget aaa bbb

获取值长度
strlen aaa

字符串追加内容
append aaa good

获取指定范围的字符
getrange qingshan 0 8
```
### 应用场景
#### 缓存 String 类型
例如:热点数据缓存(例如报表，明星出轨)，对象缓存，全页缓存。
#### 数据共享
redis是独立部署的数据服务，可以在多个应用之间共享数据。
#### 分布式锁
STRING 类型 setnx 方法，只有不存在时才能添加成功，返回 true。基于这个特性做分布式锁。
#### 全局 ID
INT 类型，INCRBY，利用原子性

incrby userid 1000

#### 计数器
>INT 类型，INCR 方法

例如:文章的阅读量，微博点赞数，允许一定的延迟，先写入 Redis 再定时同步到 数据库。
#### 限流
> INT 类型，INCR 方法

以访问者的 IP 和其他信息作为 key，访问一次增加一次计数，超过次数则返回 false。

#### 位统计
String 类型的 BITCOUNT(1.6.6 的 bitmap 数据结构介绍)。

字符是以 8 位二进制存储的。
## Hash 哈希
### 存储类型

包含键值对的无序散列表。value 只能是字符串，不能嵌套其他类型。

同样是存储字符串，Hash 与 String 的主要区别?
1. 把所有相关的值聚集到一个 key 中，节省内存空间
2. 只使用一个 key，减少 key 冲突 
3. 当需要批量获取值的时候，只需要使用一个命令，减少内存/IO/CPU 的消耗

Hash 不适合的场景:
1. Field 不能单独设置过期时间
2. 没有 bit 操作
3. 需要考虑数据量分布的问题(value 值非常大的时候，无法分布到多个节点)
### 应用场景
String 可以做的事情，Hash 都可以做。
#### 存储对象类型的数据
比如对象或者一张表的数据，比 String 节省了更多 key 的空间，也更加便于集中管理。
## List 列表
### 存储类型
存储有序的字符串(从左到右)，元素可以重复。可以充当队列和栈的角色。
操作命令
![](/img/redis/1.jpg)
## Set 集合
### 存储类型
String 类型的无序集合，最大存储数量 2^32-1(40 亿左右)。
### 操作命令
```
sadd myset a b c d e f g   添加多个元素
smembers myset             获取所有元素
scard myset                统计元素个数

srandmember key            随机获取一个元素
spop myset                 随机弹出一个元素
srem myset d e f           移除一个或者多个元素
sismember myset a          查看元素是否存在

两个集合操作
sdiff set1 set2            获取差集
sinter set1 set2           获取交集(intersection )
sunion set1 set2           获取并集
```

### 应用场景
#### 抽奖
随机获取元素 spop myset
#### 点赞、签到、打卡
![](/img/redis/2.jpg)
```
这条微博的 ID 是 t1001，用户 ID 是 u3001。
用 like:t1001 来维护 t1001 这条微博的所有点赞用户。
点赞了这条微博:sadd like:t1001 u3001
取消点赞:srem like:t1001 u3001 
是否点赞:sismember like:t1001 u3001 
点赞的所有用户:smembers like:t1001
点赞数:scard like:t1001
```

#### 商品标签
用 tags:i5001 来维护商品所有的标签。
![](/img/redis/3.jpg)
```
sadd tags:i5001 画面清晰细腻 
sadd tags:i5001 真彩清晰显示屏 
sadd tags:i5001 流畅至极
```

#### 商品筛选

![](/img/redis/4.jpg)
```
iPhone11 上市了。
sadd brand:apple iPhone11
sadd brand:ios iPhone11
sad screensize:6.0-6.24 iPhone11 sad screentype:lcd iPhone11
筛选商品，苹果的，iOS 的，屏幕在 6.0-6.24 之间的，屏幕材质是 LCD 屏幕 sinter brand:apple brand:ios screensize:6.0-6.24 screentype:lcd
```



## zset 有序集合

### 存储类型
sorted set，有序的 set，每个元素有个 score。 score 相同时，按照 key 的 ASCII 码排序。
数据结构对比:
![](/img/redis/5.jpg)

### 操作命令
```
zadd myzset 10 java 20 php 30 ruby 40 cpp 50 python 添加元素
获取全部元素
zrange myzset 0 -1 withscores
zrevrange myzset 0 -1 withscores
根据分值区间获取元素
zrangebyscore myzset 20 30

移除元素
也可以根据 score rank 删除
zrem myzset php cpp

统计元素个数
zcard myzset

分值递增
zincrby myzset 5 python

根据分值统计个数
zcount myzset 20 60

获取元素 rank
zrank myzset java

获取元素 score
zsocre myzset java
```

### 应用场景

#### 排行榜
id 为 6001 的新闻点击数加 1:zincrby hotNews:20190926 1 n6001

获取今天点击最多的 15 条:zrevrange hotNews:20190926 0 15 withscores


