# Redis

## 源码安装与运行

```shell
wget https://github.com/redis/redis/archive/7.2.3.tar.gz

tar -zxvf redis-7.2.3.tar.gz

yum -y install cpp binutils glibc glibc-kernheaders glibc-common glibc-devel gcc make

cd redis-7.2.3

make

make install
```

```
redis-server

redis-cli
```

## NoSQL

redis  mongodb  hbase  neo4j

## 简介

remote dictionary server 

- 键值型
- 单线程,具备原子性
- 低延迟,速度快(基于内存,io多路复用,编码良好)
- 数据持久化
- 主从集群,分片集群
- 多语言客户端

## 数据结构

Key-value  key常用String  value种类很多

- String
- Hash
- List
- Set
- sortedSet
- Geo
- BitMap
- HyperLog

## 常用命令:

帮助命令 **help**

```
help

To get help about Redis commands type:
      "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit

To set redis-cli preferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc
```

```
//举例  查看string generic
help @string
help @generic
```

### Keys 

查看所有符合pattern的键. (不要再生产环境使用)

```properties
keys *
keys *a*
keys a??
```

### del 

删除一个key,或者批量删除,返回删除数量

```properties
del k1 k2 k3
```

### exists 

判断key是否存在

### expire 

给key设置过期时间 (redis 缓存数据库)

### TTL

查询key的有效时间

## String 常用命令

value有3种 普通字符串  int  float

最大空间不超过512m

### set

设置key val

```properties
set name zbq
set age 22
```

### get

通过key获取val

```properties
get name
get age
```

### mset

批量添加

```properties
mset k1 v1 k2 v2 k3 v3
```

### mget

批量获取

```properties
mget k1 k2 k3
```

### incr    

自增1

```properties
incr age
```

### incrby

自增一个值

```properties
incrby age 10
```

### decr 

自减1

### decrby

自减一个值

### incrbyfloat

对浮点数自增一个值

```properties
set money 100.9
incrbyfloat money 0.1
```

### setnx

set if not exists

如果key不存在添加kv,否则不执行

成功返回1 失败返回0

```properties
setnx car audi
```

### senex 

设置kv并添加过期时间

```properties
setex name 10 zbq
```



不同对象的id区分

用户id  订单id区分

- 使用prefix来区分,比如 user_1  order_1
- redis提供冒号 :  来提供key的层级结构

## Hash类型

其value是一个字典,类似java中的hashmap. value存放json,并且可以单独操作json里面的单个字段,做crud

### hset

添加或修改hash类型key的field值

```properties
hset key field value
hset response code 200
hset response msg success
```

### hget

获取hash类型key的field

```properties
hget response code
```

### hmset

设置hash类型的多个field

```properties
hmset response code 200 msg success data 192.8
```

###  hmget

一次获取hash类型多个field的值

```properties
hmget response code data msg
```

### hgetall

获取hash类型所有field和值

```properties
hgetall response
```

### hkeys

获取hash类型key所有field

```properties
hkeys response
```

### hvals

获取hash类型key所有value

```properties
hvals response
```

### hincrby   hincrbyfloat

让hash类型key的字段值自增指定步长

### hsetnx

set if not exists  添加hash类型key一个field (之前不存在)

```properties
127.0.0.1:6379> HSETNX response code 201
(integer) 0
127.0.0.1:6379> HSETNX response state 0
(integer) 1
```

## List类型

可以看作java中的LinkedList

常用来保存有序数据,如朋友圈点赞,评论列表

### lpush

左侧插入一个或多个

```properties
LPUSH intnums 1 2 3
```

### lpop

左侧移除列表第一个元素

```properties
lpop intnums
```

### rpush

右侧侧插入一个或多个

```properties
rpush intnums 3 2 1
```

### rpop

右侧移除列表第一个元素

```properties
rpop intnums
```

### lrange 

返回范围内的元素  lrange key start end (下标从0开始,支持负数)

```properties
lrange intnums 0 2
```

### BLPOP和BRPOP

在没有元素pop时阻塞,等待指定时间,而并不是直接返回nil

## Set类型

与hashset类似 , 无序,元素不重复, 查找快, 支持并交差集

### sadd

向set种添加一个或多个元素

```properties
SADD myset 1 2 3
```

### srem

移除set中指定元素

```properties
srem myset 2
```

### scard 

返回set中元素数量

```properties
scard myset
```

### sismember

判断一个元素是否在set中

```properties
sismember myset 4
```

### smembers

返回set中所有元素

```properties
SMEMBERS myset
```

### sinter

求key1和key2的交集

```properties
sadd myset2 3 4 5
SINTER myset myset2
```

### sdiff

求key1和key2的差集

```properties
sdiff myset myset2
```

### sunion

求key1和key2的并集

```properties
sunion myset myset2
```

## sortedset类型

与java中treeset类似, 但底层数据结构差别很大, 其中每个元素带有一个score属性, 基于score对元素排序,底层实现时跳表加hash表

经常用来实现排行榜

**默认升序, 需要降序在命令z后面添加rev**

### zadd

添加一个或多个元素到sortedset,如果已经存在就更新其score值

```properties
zadd key score member
```

### zrem

删除sorted set中指定元素

```properties
zrem key member
```

### zscore

获取sorted set中指定元素的score值

```properties
zscore key member
```

### zrank

获取元素排名

```properties
zrank key member
```

### zcard

获取元素个数

```properties
zcard key
```

### zcount

统计score值在指定范围内元素个数

```properties
zcount key min max
```

### zincyby

元素score自增指定步长

```properties
zincyby key increment member
```

### zrange

按照score排序后,获取指定范围内的元素

```properties
zrange key min max
```

### zrangebyscore

按照score排序后, 获取指定score范围内的元素

```properties
zrangebyscore key min max
```

### zdiff zinter zunion

求差集,交集,并集