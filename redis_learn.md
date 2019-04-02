   * [Redis Document](#redis-document)
      * [一. Redis 综合概述 //TODO](#一-redis-综合概述-todo)
      * [二. Redis 搭建教程 -- centos7.2/ubuntu](#二-redis-搭建教程----centos72ubuntu)
      * [三. Redis 配置文件 --&gt; redis.conf](#三-redis-配置文件----redisconf)
      * [四. Redis多个数据库  //TODO](#四-redis多个数据库--todo)
      * [五. Redis 基础数据结构](#五-redis-基础数据结构)
         * [5.1 键(keys)](#51-键keys)
         * [5.2 字符串类型(String)](#52-字符串类型string)
         * [5.3 散列类型(Hash)](#53-散列类型hash)
         * [5.4 列表类型(List)](#54-列表类型list)
         * [5.5 集合(Set)](#55-集合set)
         * [5.6  有序集合(sorted set)](#56--有序集合sorted-set)
         * [5.7 Sort排序](#57-sort排序)
         * [5.7.1 基本命令](#571-基本命令)
         * [5.7.2 BY参数](#572-by参数)
         * [5.7.3 GET参数 // TODO 学完Jedis使用Java创建键值对会方便点](#573-get参数--todo-学完jedis使用java创建键值对会方便点)
         * [5.7.4 STORE参数](#574-store参数)
         * [5.7.4 SORT性能优化](#574-sort性能优化)
      * [六. 事务](#六-事务)
         * [6.1 Redis 事务可以一次执行多个命令,  并且带有以下两个重要的保证:](#61-redis-事务可以一次执行多个命令--并且带有以下两个重要的保证)
         * [6.2 事务错误执行时会出现以下的操作:](#62-事务错误执行时会出现以下的操作)
      * [7 发布订阅](#7-发布订阅)
      * [8 Java连接Redis --&gt; Jedis](#8-java连接redis----jedis)
         * [8.1 Jedis简介](#81-jedis简介)
         * [8.2 测试Redis连接代码](#82-测试redis连接代码)
      * [9 Redisson -- 分布式锁 //TODO](#9-redisson----分布式锁-todo)
      * [10. 管道](#10-管道)
      * [11. Redis 脚本 //TODO 等学完集群再来看一下lua脚本与集群的问题](#11-redis-脚本-todo-等学完集群再来看一下lua脚本与集群的问题)
         * [1. 客户端脚本](#1-客户端脚本)
         * [2. Jedis脚本](#2-jedis脚本)
      * [12. 持久化](#12-持久化)
         * [12.1 Redis 提供了多种不同级别的持久化方式:](#121-redis-提供了多种不同级别的持久化方式)
         * [12.2 RDB](#122-rdb)
         * [12.3 AOF](#123-aof)
         * [12.4 持久化策略选择](#124-持久化策略选择)
      * [13. 集群](#13-集群)
      * [14. 管理](#14-管理)
         * [14.1 内部编码优化:](#141-内部编码优化)
         * [14.2](#142)

# Redis Document

## 一. Redis 综合概述 //TODO
## 二. Redis 搭建教程 -- centos7.2/ubuntu
- 下载地址:https://redis.io/download,下载最新稳定版本.
-  本教程使用的最新文档版本为 5.0.4, 下载并安装:
```shell
  $ wget http://download.redis.io/releases/redis-5.0.4.tar.gz
  $ tar -zxvf redis-5.0.4.tar.gz
  $ cd redis-5.0.4
  # 有些机器没有安装gcc, 会导致make报错, 可以yum install gcc(centos)或者 sudo apt install gcc(ubuntu) 删除原来的文件夹再解压一个
  $ make
```
- make完后 redis-5.0.4 目录下会出现编译后的redis服务程序redis-server, 还有用于测试的客户端程序redis-cli,两个程序位于安装目录 src 目录下. 下面启动redis服务:
``` shell
  $ cd src
  $ ./redis-server
```
- 注意这种方式启动redis 使用的是默认配置. 也可以通过启动参数告诉redis使用指定配置文件使用下面命令启动. 
```shell
  $ cd src
  $ ./redis-server ../redis.conf
```
- redis.conf 是一个默认的配置文件. 我们可以根据需要使用自己的配置文件.

- 启动redis服务进程后, 就可以使用测试客户端程序redis-cli和redis服务交互了.  比如:

```shell
  $ cd src
  $ ./redis-cli --raw  # 加上--raw可以正常显示中文字符, 而不会显示16进制字符
  127.0.0.1:6379> ping
  PONG
```
- 关闭 redis
 ```shell
    # 此代码只能关闭本地6379的redis
    ./redis-cli shutdown
    # 或者
    ps aux | grep redis | grep -v grep | awk '{print $2}'| xargs kill -9
  ```
## 三. Redis 配置文件 --> redis.conf
- Redis 的配置文件位于 Redis 安装目录下, 文件名为 redis.conf. 
> Redis CONFIG 命令格式如下:
```shell
    # 获取redis配置
    redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
    # 更改redis配置 --> 并不是所有配置项都可以在命令行里修改
    redis 127.0.0.1:6379> CONFIG SET CONFIG_SETTING_NAME CONFIG_SETTING_VALUE
```
> 使用 * 号获取所有配置项:
```shell
    redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
```
> 配置信息详解: http://www.runoob.com/redis/redis-conf.html

## 四. Redis多个数据库  //TODO

- 多个数据库的定义
- 删除单个数据库的数据
- 删除所有数据库的数据
  
## 五. Redis 基础数据结构
### 5.1 键(keys)
```shell
#该命令用于在 key 存在时删除 key.
# del 命令的参数不支持通配符, 但可以使用linux的管道和xargs命令实现删除所有符合条件的键.比如要删除user开头的键:
# 1. ./redis-cli keys "user:*" | xargs redis-cli del 
# 2. del 支持多个键作为参数, 还可以执行 ./redis-cli del `redis-cli keys "user:*"` --> 性能更好
del key

#序列化给定key, 并返回被序列化的值.
dump key 

#检查给定key是否存在. 存在返回1,  不存在返回0
exists key 

#为给定key设置过期时间, 以秒计. 过期后删除
expire key seconds

#EXPIREAT的作用和EXPIRE类似, 都用于为key设置过期时间. 不同在于EXPIREAT命令接受的时间参数是UNIX时间戳(unix timestamp).
expireat key timestamp

#设置key的过期时间以毫秒计.
pexpire key milliseconds 

#设置key过期时间的时间戳(unix timestamp) 以毫秒计.
pexpireat key milliseconds-timestamp 

#查找所有符合给定模式( pattern)的 key.
#通配符: ? -> 一个字符, * -> 任意个字符, [] -> 匹配括号内的任意字符a[b-c]:ab, ac, \x -> 转义字符
#keys需要遍历Redis所有的键, 生产环境上不建议使用. 
#Redis命令不区分大小写
keys pattern 

#将当前数据库的key移动到给定的数据库 db 当中.
move key db 

#移除key的过期时间, key将持久保持.
persist key 

#以毫秒为单位返回key的剩余的过期时间. 
pttl key 

#以秒为单位, 返回给定key的剩余生存时间(TT., time to live).
ttl key 

#从当前数据库中随机返回一个key.
randomkey  

#修改key的名称
rename key newkey 

#仅当newkey不存在时, 将key改名为 newkey.
renamenx key newkey 

#返回key所储存的值的类型.
type key 
```

### 5.2 字符串类型(String)
  字符串类型是redis最基本的类型, 能存储任意形式的字符串, 包括二进制数据, 允许最多的容量是512MB.
```shell
#获取指定key的值.
get key

#获取指定key的值.
set key value

#返回key中字符串值的子字符
getrange key start end

#将给定key的值设为value , 并返回key的旧值(old value).
getset key value

#对key所储存的字符串值, 获取指定偏移量上的位(bit).
getbit key offset

#将值value关联到key, 并将key的过期时间设为seconds(以秒为单位)
setex key seconds value

#只有在key不存在时设置key的值.
setnx key value

#用value参数覆写给定key所储存的字符串值, 从偏移量offset开始.
setrange key offset value

#返回key所储存的字符串值的长度.
strlen key

#同时设置一个或多个key-value对.
mset key value [key value...]

#这个命令和 SETEX 命令相似, 但它以毫秒为单位设置 key 的生存时间, 而不是像 SETEX 命令那样, 以秒为单位. 
psetex key milliseconds value

#将key中储存的数字值增一.
incr key

#将key所储存的值加上给定的增量值(increment)
incrby key increment

#将key所储存的值加上给定的浮点增量值(increment)
incrbyfloat key increment

#将key中储存的数字值减一
decr key

#key所储存的值减去给定的减量值(decrement).
decrby key decrement

#如果key已经存在并且是一个字符串, APPEND命令将指定的value追加到该key原来值(value)的末尾.
append key value
```

### 5.3 散列类型(Hash)
  Redis hash 是一个string类型的field和value的映射表, hash特别适合用于**存储对象**. value只能为字符串.

  Redis 中每个 hash 可以存储 $2^{32} - 1$ 键值对(40多亿).

  
```shell
#删除一个或多个哈希表字段
HDEL key field1 [field2] 

#查看哈希表key中, 指定的字段是否存在
HEXISTS key field 

#获取存储在哈希表中指定字段的值.
HGET key field 

#获取在哈希表中指定key的所有字段和值
HGETALL key 

#为哈希表key中的指定字段的整数值加上增量 increment.
HINCRBY key field increment 

#为哈希表key中的指定字段的浮点数值加上增量increment.
HINCRBYFLOAT key field increment 

#获取所有哈希表中的字段
HKEYS key 

#获取哈希表中字段的数量
HLEN key 

#获取所有给定字段的值
HMGET key field1 [field2] 

#同时将多个field-value(域-值)对设置到哈希表key中
HMSET key field1 value1 [field2 value2 ] 

#将哈希表key中的字段field的值设为value
HSET key field value 

#只有在字段field不存在时, 设置哈希表字段的值
HSETNX key field value 

#获取哈希表中所有值
HVALS key 

#迭代哈希表中的键值对, cursor: 游标分页, 每次都会返回一个cursor的值, cursor=0表示迭代结束, match: 通配符, count: 表示每次返回的数据量, 默认值是10. 当field的数量大于某个值时, 分页才会生效
HSCAN key cursor [MATCH pattern] [COUNT count] 
hscan myhash 0 match a* count 1
```

### 5.4 列表类型(List)
Redis列表是简单的字符串列表, 按照插入顺序排序. 你可以添加一个元素到列表的头部(左边)或者尾部(右边)

一个列表最多可以包含 $2^{32} - 1$ 个元素 (4294967295, 每个列表超过40亿个元素).

由于是链表结构, 插入复杂度$ O(1) $, 获取越接近两端的数据速度越快. 

```shell
#移出并获取列表的第一个元素, 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
BLPOP key1 [key2 ] timeout 

#移出并获取列表的最后一个元素,  如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
BRPOP key1 [key2 ] timeout 

#从列表中弹出一个值, 将弹出的元素插入到另外一个列表中并返回它. 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
BRPOPLPUSH source destination timeout 

#通过索引获取列表中的元素
LINDEX key index 

#在列表的元素前或者后插入元素
LINSERT key BEFORE|AFTER pivot value 

#获取列表长度
LLEN key 

#移出并获取列表的第一个元素
LPOP key 

#将一个或多个值插入到列表头部
LPUSH key value1 [value2] 

#将一个值插入到已存在的列表头部
LPUSHX key value 

#获取列表指定范围内的元素
LRANGE key start stop 
#获取列表所有元素
lrange key 0 -1

#移除列表元素
LREM key count value 

#通过索引设置列表元素的值
LSET key index value 

#对一个列表进行修剪(trim), 就是说, 让列表只保留指定区间内的元素, 不在指定区间之内的元素都将被删除
LTRIM key start stop 

#移除列表的最后一个元素, 返回值为移除的元素
RPOP key 

#移除列表的最后一个元素, 并将该元素添加到另一个列表并返回
RPOPLPUSH source destination 

#在列表中添加一个或多个值
RPUSH key value1 [value2] 

#为已存在的列表添加值
RPUSHX key value 

```

### 5.5 集合(Set)
Redis 的 Set 是 String 类型的无序集合. 集合成员是唯一的, 这就意味着集合中不能出现重复的数据.

Redis 中集合是通过哈希表实现的, 所以添加, 删除, 查找的复杂度都是 $O(1)$.

集合中最大的成员数为$2^{32}-1$ (4294967295, 每个集合可存储40多亿个成员).

Set最常用的操作是向集合加入或删除元素, 判断某个元素是否存在, 多个集合类型之间还可以做并集, 交集和差集运算.\

```shell
#向集合添加一个或多个成员
SADD key member1 [member2] 

#获取集合的成员数
SCARD key 

#返回给定所有集合的差集
SDIFF key1 [key2] 

#返回给定所有集合的差集并存储在destination中
SDIFFSTORE destination key1 [key2] 

#返回给定所有集合的交集
SINTER key1 [key2] 

#返回给定所有集合的交集并存储在destination中
SINTERSTORE destination key1 [key2] 

#判断 member 元素是否是集合 key 的成员
SISMEMBER key member 

#返回集合中的所有成员
SMEMBERS key 

#将 member 元素从 source 集合移动到 destination 集合
SMOVE source destination member 

#移除并返回集合中的一个随机元素
SPOP key 

#返回集合中一个或多个随机数
SRANDMEMBER key [count] 

#移除集合中一个或多个成员
SREM key member1 [member2] 

#返回所有给定集合的并集
SUNION key1 [key2] 

#所有给定集合的并集存储在 destination 集合中
SUNIONSTORE destination key1 [key2] 

#迭代集合中的元素
SSCAN key cursor [MATCH pattern] [COUNT count] 
```
### 5.6  有序集合(sorted set) 
有序集合和集合一样也是string类型元素的集合,且不允许重复的成员. 

不同的是每个元素都会关联一个double类型的分数. redis正是通过分数来为集合中的成员进行从小到大的排序. 

有序集合的成员是唯一的,但分数(score)却可以重复. 

集合是通过哈希表实现的, 所以添加, 删除, 查找的复杂度都是$ O(1) $.  集合中最大的成员数为 $2^{32} - 1$ (4294967295, 每个集合可存储40多亿个成员)

> - 集合和有序集合的区别:
>> 1. 列表类型是通过链表实现的, 获取靠近两端的数据速度极快, 但是元素增多后, 访问中间数据的速度会很慢, 所以他更加适合实现如"新鲜事"或者"日志"这样很少访问中间元素的应用
>> 2. 有序集合是使用散列表和跳跃表(skip list)实现的, 所以读取中间的数据也很快$O(log(N))$
>>>其实在redis sorted sets里面当items内容大于64的时候同时使用了hash和skiplist两种设计实现. 这也会为了排序和查找性能做的优化. 所以如上可知: 
添加和删除都需要修改skiplist, 所以复杂度为$O(log(N))$.
但是如果仅仅是查找元素的话可以直接使用hash, 其复杂度为$O(1)$
其他的range操作复杂度一般为$O(log(N))$
当然如果是小于64的时候, 因为是采用了ziplist的设计, 其时间复杂度为$O(n)$
>> 3. 列表不能简单的调整某个元素的位置, 但是有序集合可以(通过更改元素分数)
>> 4. 有序类型要比列表类型更耗费内存
> - 集合和有序集合的相似点:
>> 1. 二者都是有序的
>> 2. 二者都可以获得某一范围的元素

```shell
#向有序集合添加一个或多个成员, 或者更新已存在成员的分数
ZADD key score1 member1 [score2 member2]

#获取有序集合的成员数
ZCARD key

#计算在有序集合中指定区间分数的成员数
ZCOUNT key min max

#有序集合中对指定成员的分数加上增量increment
ZINCRBY key increment member

#计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合key中
ZINTERSTORE destination numkeys key [key ...]

#在有序集合中计算指定字典区间内成员数量
ZLEXCOUNT key min max

#通过索引区间返回有序集合成指定区间内的成员 O(log n+m)
ZRANGE key start stop [WITHSCORES]

#通过字典区间返回有序集合的成员
ZRANGEBYLEX key min max [LIMIT offset count]

#通过分数返回有序集合指定区间内的成员
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]
zrangebyscore key 10 (100 # 带上括号不包含: 包含10, 不包含100

#返回有序集合中指定成员的索引
ZRANK key member

#移除有序集合中的一个或多个成员
ZREM key member [member ...]

#移除有序集合中给定的字典区间的所有成员
ZREMRANGEBYLEX key min max

#移除有序集合中给定的排名区间的所有成员
ZREMRANGEBYRANK key start stop

#移除有序集合中给定的分数区间的所有成员
ZREMRANGEBYSCORE key min max

#返回有序集中指定区间内的成员, 通过索引, 分数从高到底
ZREVRANGE key start stop [WITHSCORES]

#返回有序集中指定分数区间内的成员, 分数从高到低排序
ZREVRANGEBYSCORE key max min [WITHSCORES]

#返回有序集合中指定成员的排名, 有序集成员按分数值递减(从大到小)排序
ZREVRANK key member

#返回有序集中, 成员的分数值
ZSCORE key member

#计算给定的一个或多个有序集的并集, 并存储在新的 key 中
ZUNIONSTORE destination numkeys key [key ...]

#迭代有序集合中的元素(包括元素成员和元素分值)
ZSCAN key cursor [MATCH pattern] [COUNT count]
```
### 5.7 Sort排序
除了使用有序集合外, 还可以借助redis自带的sort命令解决排序问题. sort命令可以对列表类型, 集合类型和有序集合类型键进行排序, 并且可以完成数据库的连接查询相类似的任务. sort命令并不会改变原有元素的顺序, 排序后返回.
### 5.7.1 基本命令 
```shell
  # 1. 集合类型排序
  127.0.0.1:6379> sadd myset 3 1 5 4
  (integer) 4
  127.0.0.1:6379> sort myset
  1) "1"
  2) "3"
  3) "4"
  4) "5" 

  # 2. 列表
  127.0.0.1:6379> rpush mylist 2 6 4 8
  (integer) 4
  127.0.0.1:6379> sort mylist
  1) "2"
  2) "4"
  3) "6"
  4) "8"

  # 3. 有序类型 会忽略分数排序直接对元素的值进行排序
  127.0.0.1:6379> zadd myzset 50 3 31 8 23 6
  (integer) 3
  127.0.0.1:6379> zscan myzset 0
  1) "0"
  2) 1) "6"
    2) "23"
    3) "8"
    4) "31"
    5) "3"
    6) "50"
  127.0.0.1:6379> sort myzset
  1) "3"
  2) "6"
  3) "8"

  # 4. sort还可以按照字典顺序对字母进行排序
  127.0.0.1:6379> lpush mylistalpha a b c B C A
  (integer) 6
  # 没有加alpha参数会出错
  127.0.0.1:6379> sort mylistalpha
  (error) ERR One or more scores can't be converted into double
  127.0.0.1:6379> sort mylistalpha alpha
  1) "a"
  2) "A"
  3) "b"
  4) "B"
  5) "c"
  6) "C"

```
### 5.7.2 BY参数
  BY参数的语法为BY参考键. 其中参考键可以是字符串类型键或者是散列类型的某个字段(表示为键名 -> 字段名). 提供了BY参数, sort命令将不在依据元素自身的值进行培训, 而是对每个元素的值替换参考键中的第一个"*"并获取其值, 然后依据该值对元素进行排序.
  ```shell
  # list 的值按照hash中某个属性的值排序;
  # 1. 首先在redis插入以下hash数据:
  personName:小明0 score:88.73963 age:9 id:0 
  personName:小明1 score:89.54764 age:9 id:1 
  personName:小明2 score:83.778534 age:9 id:2 
  personName:小明3 score:80.06653 age:2 id:3 
  personName:小明4 score:84.61355 age:5 id:4 
  personName:小明5 score:83.65758 age:6 id:5 
  personName:小明6 score: age:9 id:6 
  personName:小明7 score:84.49216 age:0 id:7 
  personName:小明8 score:86.20994 age:7 id:8 
  personName:小明9 score:84.179184 age:8 id:9 

  # 2. 建立要排序的列表
  127.0.0.1:6379> lrange sortedbymyhash 0 -1
  9
  8
  7
  6
  5
  4
  3
  2
  1
  0

  # 3. 将list的顺序按照hash里面的score属性值排序
  127.0.0.1:6379> sort sortedbymyhash by person:*->score desc
  1  # 89.5476481.55472
  0  # 88.73963
  8  # 86.20994
  4  # 84.61355
  7  # 84.49216
  9  # 84.179184
  2  # 83.778534
  5  # 83.65758
  6  # 81.55472
  3  # 80.06653
  

  # list中按照某个属性排序, 值要和score:*的*值对应, 后面的值相当于分数
  # 当score:*中不包含mylist对应的值就不排序, 存在score里面的值相同才会按照本身的值排序
 127.0.0.1:6379> lrange mylist 0 -1
  1) "2"
  2) "6"
  3) "4"
  4) "8"
  127.0.0.1:6379> set score:2 10
  OK
  127.0.0.1:6379> set score:4 11
  OK
  127.0.0.1:6379> set score:6 12
  OK
  127.0.0.1:6379> set score:8 13
  OK
  127.0.0.1:6379> sort mylist by score:*
  1) "2"
  2) "4"
  3) "6"
  4) "8"
  ```

  ### 5.7.3 GET参数 // TODO 学完Jedis使用Java创建键值对会方便点
  GET参数不影响排序, 他的作用是使sort命令的返回结果不再是元素自身的值, 而是GET参数中指定的键值. GET同样支持字符串类型和散列类型, 并且使用*作为占位符. 要实现刚才我们在将list的顺序按照hash里面的score属性值排序中人的名字可以如下获取:
  ```shell
  127.0.0.1:6379> sort sortedbymyhash by person:*->score desc get person:*->personName
  小明1
  小明0
  小明8
  小明4
  小明7
  小明9
  小明2
  小明5
  小明6
  小明3

  # GET可以使用多个获取多个属性, get # 代表获取自身的值
  127.0.0.1:6379> sort sortedbymyhash by person:*->score desc get # get person:*->personName get person:*->score
  1
  小明1
  89.54764
  0
  小明0
  88.73963
  8
  小明8
  86.20994
  4
  小明4
  84.61355
  7
  小明7
  84.49216
  9
  小明9
  84.179184
  2
  小明2
  83.778534
  5
  小明5
  83.65758
  6
  小明6
  81.55472
  3
  小明3
  80.06653
  ```
  ### 5.7.4 STORE参数
  STORE参数可以将SORT的排序结果以**列表**存储下来.

  ```shell
  127.0.0.1:6379> sort sortedbymyhash by person:*->score desc get person:*->personName store sortedbyscoreName
  10

  127.0.0.1:6379> lrange sortedbyscoreName 0 -1
  小明1
  小明0
  小明8
  小明4
  小明7
  小明9
  小明2
  小明5
  小明6
  小明3
  ```

  ### 5.7.4 SORT性能优化
  sort是Redis中最强大最复杂的命令之一, 使用不好很容易会成为瓶颈. sort命令的时间复杂度是$O(n+mlog(m))$, 其中n表示要排序的列表(集合或有序集合)中的元素个数, m表示要返回的元素个数. 当n较大的时候sort命令的性能相对较低, 并且Redis在排序前会建立一个长度为n的容器来存储待排序的元素, 虽然是一个临时过程, 但如果进行较多的大数据量排序的操作则会严重影响性能.
  
  所以使用过程中要注意一下几点:
  - 1). 尽可能减少待排序键中元素的数量(使N尽可能的少)
  - 2). 使用limit参数只获取需要的数据(使M尽可能的小)
  - 3). 如果要排序的数据量较大, 尽可能使用score参数将结果缓存
    


## 六. 事务
### 6.1 Redis 事务可以一次执行多个命令,  并且带有以下两个重要的保证:

- 批量操作在发送 EXEC 命令前被放入队列缓存. 
- 收到 EXEC 命令后进入事务执行, 事务中任意命令执行失败, 其余的命令依然被执行. 
- 在事务执行过程, 其他客户端提交的命令请求不会插入到事务执行命令序列中. 

一个事务从开始到执行会经历以下三个阶段:
- 开始事务. 
- 命令入队. 
- 执行事务. 

### 6.2 事务错误执行时会出现以下的操作:
- 语法错误
  > 只要有一个命令有语法错误, 执行exec之后redis就会直接返回错误, 连语法正确的命令也不会执行
- 运行错误
  > 单个 Redis 命令的执行是原子性的, 但 Redis 没有在事务上增加任何维持原子性的机制, 所以 Redis 事务的执行并不是原子性的. 事务可以理解为一个打包的批量执行脚本, 但批量指令并非原子化的操作, 中间某条指令的失败不会导致前面已做指令的回滚, 也不会造成后续的指令不做.

```shell
#取消事务, 放弃执行事务块内的所有命令
DISCARD 
#执行所有事务块内的命令
EXEC 
#标记一个事务块的开始
MULTI 
#取消WATCH命令对所有key的监视
ATCH 
#监视一个(或多个)key, 如果在事务执行之前这个(或这些)key被其他命令所改动, 那么事务将被打断
WATCH key [key ...] 
# watch例子如下:
redis>set a 1
redis>watch a
redis>set a 2
redis>multi
redis>set a 3
redis>exec
redis>get a # a = 2 因为watch之后a发生了改变, 不会执行事务
```
## 7 发布订阅
Redis 发布订阅(pub/sub)是一种消息通信模式:发送者(pub)发送消息, 订阅者(sub)接收消息. 

Redis 客户端可以订阅任意数量的频道.
```shell
#订阅一个或多个符合给定模式的频道. 
PSUBSCRIBE pattern [pattern ...] 

#查看订阅与发布系统状态. 
PUBSUB subcommand [argument [argument ...]] 

#将信息发送到指定的频道. 
PUBLISH channel message 

#退订所有给定模式的频道. 
PUNSUBSCRIBE [pattern [pattern ...]] 

#订阅给定的一个或多个频道的信息. 
SUBSCRIBE channel [channel ...] 

# 指退订给定的频道
UNSUBSCRIBE [channel [channel ...]] 

```

同样, 订阅可以按照规则订阅. 
```shell
# 可以匹配channel.1 / channel.10
psubscribe channel.?*

```

## 8 Java连接Redis --> Jedis
### 8.1 Jedis简介
  Java连接Redis的工具叫Jedis, 引入依赖包即可使用.
  ```java
    <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.0.1</version>
    </dependency>
  ```
### 8.2 测试Redis连接代码
```java
package com.rediscli;

import redis.clients.jedis.Jedis;

import java.util.Set;


public class RedisCliManager {

    public static void main(String[] args) {
        // centos01 是我用VM开的虚拟机IP地址
        Jedis jedis = new Jedis("centos01", 6379);

        jedis.connect();

        Set<String> keys = jedis.keys("*");

        for (String key : keys) {
            System.out.println(key);
        }
        jedis.disconnect();
    }
}
```
可能会出现以下错误:
```java
redis.clients.jedis.exceptions.JedisConnectionException: java.net.ConnectException: Connection refused: connect
```
解决方案:
1. 关闭防火墙或开放端口
2. 更改redis.conf 配置
   bind 0.0.0.0 // 接收所有ip请求的访问
   protected-mode no  // 安全模式改为0
   
    
<font color=red>**由于Jedis只要创建之后就可连接redis进行操作, 除了Jedis不会涉及到其他类, 所以看Redis的各个类型的命令行和Jedis的API即可.**</font>

## 9 Redisson -- 分布式锁 //TODO

## 10. 管道
Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务. 这意味着通常情况下一个请求会遵循以下步骤:

- 客户端向服务端发送一个查询请求, 并监听Socket返回, 通常是以阻塞模式, 等待服务端响应.
- 服务端处理命令, 并将结果返回给客户端.
  
Redis 管道技术可以在服务端未响应时, 客户端可以继续向服务端发送请求, 并最终一次性读取所有服务端的响应. 

```java
package com.rediscli.test;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Pipeline;
import redis.clients.jedis.Response;

import java.util.List;

public class TestRedisPipeline {



    private Jedis jedis;

    @Before
    public void before() {
        this.jedis = new Jedis("centos01", 6379);
        this.jedis.connect();

        //System.out.println(jedis.isConnected());
    }

    @Test
    public void test() {

        Pipeline pipelined = jedis.pipelined();
        pipelined.set("pipeline", String.valueOf(1));
        pipelined.set("pipeline", String.valueOf(3));
        pipelined.set("pipeline", String.valueOf(2));
        // 可以拿单个的返回结果
        Response<String> pipeline = pipelined.get("pipeline");
        List<Object> objects = pipelined.syncAndReturnAll();
        // 2
        System.out.println(pipeline.get().toString());
        // [OK, OK, OK, 2]
        System.out.println(objects);

    }


    @After
    public void after() {
        this.jedis.disconnect();
    }

}
```




## 11. Redis 脚本 //TODO 等学完集群再来看一下lua脚本与集群的问题
### 1. 客户端脚本
### 2. Jedis脚本



## 12. 持久化
### 12.1 Redis 提供了多种不同级别的持久化方式:

- RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照(point-in-time snapshot). 
- AOF 持久化记录服务器执行的所有写操作命令, 并在服务器启动时, 通过重新执行这些命令来还原数据集.  AOF 文件中的命令全部以 Redis 协议的格式来保存, 新命令会被追加到文件的末尾.  Redis 还可以在后台对 AOF 文件进行重写(rewrite), 使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小. 
- Redis 还可以同时使用 AOF 持久化和 RDB 持久化.  在这种情况下,  当 Redis 重启时,  它会优先使用 AOF 文件来还原数据集,  因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整. 
- 你甚至可以关闭持久化功能, 让数据只在服务器运行时存在. 

### 12.2 RDB
> 优点
> - RDB 是一个非常紧凑(compact)的文件, 它保存了 Redis 在某个时间点上的数据集.  这种文件非常适合用于进行备份: 比如说, 你可以在最近的 24 小时内, 每小时备份一次 RDB 文件, 并且在每个月的每一天, 也备份一个 RDB 文件.  这样的话, 即使遇上问题, 也可以随时将数据集还原到不同的版本. 
> - RDB 非常适用于灾难恢复(disaster recovery):它只有一个文件, 并且内容都非常紧凑, 可以(在加密后)将它传送到别的数据中心, 或者亚马逊 S3 中. 
> - RDB 可以最大化 Redis 的性能:父进程在保存 RDB 文件时唯一要做的就是 fork 出一个子进程, 然后这个子进程就会处理接下来的所有保存工作, 父进程无须执行任何磁盘 I/O 操作. 
> - RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快. 


> 缺点
> - 如果你需要尽量避免在服务器故障时丢失数据, 那么 RDB 不适合你.  虽然 Redis 允许你设置不同的保存点(save point)来控制保存 RDB 文件的频率,  但是,  因为RDB 文件需要保存整个数据集的状态,  所以它并不是一个轻松的操作.  因此你可能会至少 5 分钟才保存一次 RDB 文件.  在这种情况下,  一旦发生故障停机,  你就可能会丢失好几分钟的数据. 
> - 每次保存 RDB 的时候, Redis 都要 fork() 出一个子进程, 并由子进程来进行实际的持久化工作.  在数据集比较庞大时,  fork() 可能会非常耗时, 造成服务器在某某毫秒内停止处理客户端;  如果数据集非常巨大, 并且 CPU 时间非常紧张的话, 那么这种停止时间甚至可能会长达整整一秒.  虽然 AOF 重写也需要进行 fork() , 但无论 AOF 重写的执行间隔有多长, 数据的耐久性都不会有任何损失. 

### 12.3 AOF
> 优点
> 使用 AOF 持久化会让 Redis 变得非常耐久(much more durable):你可以设置不同的 fsync 策略, 比如无 fsync, 每秒钟一次 fsync , 或者每次执行写入命令时 fsync .  AOF 的默认策略为每秒钟 fsync 一次, 在这种配置下, Redis 仍然可以保持良好的性能, 并且就算发生故障停机, 也最多只会丢失一秒钟的数据(fsync 会在后台线程执行, 所以主线程可以继续努力地处理命令请求). 
AOF 文件是一个只进行追加操作的日志文件(append only log),  因此对 AOF 文件的写入不需要进行 seek ,  即使日志因为某些原因而包含了未写入完整的命令(比如写入时磁盘已满, 写入中途停机, 等等),  redis-check-aof 工具也可以轻易地修复这种问题. 
> - Redis 可以在 AOF 文件体积变得过大时, 自动地在后台对 AOF 进行重写: 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合.  整个重写操作是绝对安全的, 因为 Redis 在创建新 AOF 文件的过程中, 会继续将命令追加到现有的 AOF 文件里面, 即使重写过程中发生停机, 现有的 AOF 文件也不会丢失.  而一旦新 AOF 文件创建完毕, Redis 就会从旧 AOF 文件切换到新 AOF 文件, 并开始对新 AOF 文件进行追加操作. 
> - AOF 文件有序地保存了对数据库执行的所有写入操作,  这些写入操作以 Redis 协议的格式保存,  因此 AOF 文件的内容非常容易被人读懂,  对文件进行分析(parse)也很轻松.  导出(export) AOF 文件也非常简单: 举个例子,  如果你不小心执行了 FLUSHALL 命令,  但只要 AOF 文件未被重写,  那么只要停止服务器,  移除 AOF 文件末尾的 FLUSHALL 命令,  并重启 Redis,  就可以将数据集恢复到 FLUSHALL 执行之前的状态. 

> 缺点
> - 对于相同的数据集来说, AOF 文件的体积通常要大于 RDB 文件的体积. 
根据所使用的 fsync 策略, AOF 的速度可能会慢于 RDB .  在一般情况下,  每秒 fsync 的性能依然非常高,  而关闭 fsync 可以让 AOF 的速度和 RDB 一样快,  即使在高负荷之下也是如此.  不过在处理巨大的写入载入时, RDB 可以提供更有保证的最大延迟时间(latency). 
> - AOF 在过去曾经发生过这样的 bug : 因为个别命令的原因, 导致 AOF 文件在重新载入时, 无法将数据集恢复成保存时的原样.  (举个例子, 阻塞命令 BRPOPLPUSH 就曾经引起过这样的 bug . ) 测试套件里为这种情况添加了测试: 它们会自动生成随机的、复杂的数据集,  并通过重新载入这些数据来确保一切正常.  虽然这种 bug 在 AOF 文件中并不常见,  但是对比来说,  RDB 几乎是不可能出现这种 bug 的. 

### 12.4 持久化策略选择
在介绍持久化策略之前, 首先要明白无论是RDB还是AOF, 持久化的开启都是要付出性能方面代价的:对于RDB持久化, 一方面是bgsave在进行fork操作时Redis主进程会阻塞, 另一方面, 子进程向硬盘写数据也会带来IO压力; 对于AOF持久化, 向硬盘写数据的频率大大提高(everysec策略下为秒级), IO压力更大, 甚至可能造成AOF追加阻塞问题(后面会详细介绍这种阻塞), 此外, AOF文件的重写与RDB的bgsave类似, 会有fork时的阻塞和子进程的IO压力问题. 相对来说, 由于AOF向硬盘中写数据的频率更高, 因此对Redis主进程性能的影响会更大. 

在实际生产环境中, 根据数据量、应用对数据的安全要求、预算限制等不同情况, 会有各种各样的持久化策略; 如完全不使用任何持久化、使用RDB或AOF的一种, 或同时开启RDB和AOF持久化等. 此外, 持久化的选择必须与Redis的主从策略一起考虑, 因为主从复制与持久化同样具有数据备份的功能, 而且主机master和从机slave可以独立的选择持久化方案. 

下面分场景来讨论持久化策略的选择, 下面的讨论也只是作为参考, 实际方案可能更复杂更具多样性. 

- 1). 如果Redis中的数据完全丢弃也没有关系(如Redis完全用作DB层数据的cache, 那么无论是单机, 还是主从架构, 都可以不进行任何持久化. 

- 2). 在单机环境下(对于个人开发者, 这种情况可能比较常见, 如果可以接受十几分钟或更多的数据丢失, 选择RDB对Redis的性能更加有利; 如果只能接受秒级别的数据丢失, 应该选择AOF. 

- 3). 在多数情况下, 我们都会配置主从环境, slave的存在既可以实现数据的热备, 也可以进行读写分离分担Redis读请求, 以及在master宕掉后继续提供服务. 

> 在这种情况下, 一种可行的做法是:
> master:完全关闭持久化(包括RDB和AOF), 这样可以让master的性能达到最好; 
> slave:关闭RDB, 开启AOF(如果对数据安全要求不高, 开启RDB关闭AOF也可以), 并定时对持久化文件进行备份(如备份到其他文件夹, 并标记好备份的时间); 然后关闭AOF的自动重写, 然后添加定时任务, 在每天Redis闲时(如凌晨12点)调用bgrewriteaof. 
> 这里需要解释一下, 为什么开启了主从复制, 可以实现数据的热备份, 还需要设置持久化呢？因为在一些特殊情况下, 主从复制仍然不足以保证数据的安全, 例如:
> master和slave进程同时停止:考虑这样一种场景, 如果master和slave在同一栋大楼或同一个机房, 则一次停电事故就可能导致master和slave机器同时关机, Redis进程停止; 如果没有持久化, 则面临的是数据的完全丢失. 
> master误重启:考虑这样一种场景, master服务因为故障宕掉了, 如果系统中有自动拉起机制(即检测到服务停止后重启该服务)将master自动重启, 由于没有持久化文件, 那么master重启后数据是空的, slave同步数据也变成了空的; 如果master和slave都没有持久化, 同样会面临数据的完全丢失. 需要注意的是, 即便是使用了哨兵(关于哨兵后面会有文章介绍)进行自动的主从切换, 也有可能在哨兵轮询到master之前, 便被自动拉起机制重启了. 因此, 应尽量避免“自动拉起机制”和“不做持久化”同时出现. 

- 4). 异地灾备:上述讨论的几种持久化策略, 针对的都是一般的系统故障, 如进程异常退出、宕机、断电等, 这些故障不会损坏硬盘. 但是对于一些可能导致硬盘损坏的灾难情况, 如火灾地震, 就需要进行异地灾备. 

> 例如对于单机的情形, 可以定时将RDB文件或重写后的AOF文件, 通过scp拷贝到远程机器, 如阿里云、AWS等; 对于主从的情形, 可以定时在master上执行bgsave, 然后将RDB文件拷贝到远程机器, 或者在slave上执行bgrewriteaof重写AOF文件后, 将AOF文件拷贝到远程机器上. 
> 一般来说, 由于RDB文件文件小、恢复快, 因此灾难恢复常用RDB文件; 异地备份的频率根据数据安全性的需要及其它条件来确定, 但最好不要低于一天一次. 

## 13. 集群
### 13.1 主从模式
- > 在主从复制中，数据库分为俩类，主数据库(master)和从数据库(slave)。其中主从复制有如下特点：
  > - 主数据库可以进行读写操作，当读写操作导致数据变化时会自动将数据同步给从数据库
  > - 从数据库一般都是只读的，并且接收主数据库同步过来的数据
  > - 一个master可以拥有多个slave，但是一个slave只能对应一个master

- >主从复制工作机制
  >  - 当slave启动后，主动向master发送SYNC命令。master接收到SYNC命令后在后台保存快照（RDB持久化）和缓存保存快照这段时间的命令，然后将保存的快照文件和缓存的命令发送给slave。slave接收到快照文件和命令后加载快照文件和缓存的执行命令。

  > - 复制初始化后，master每次接收到的写命令都会同步发送给slave，保证主从数据一致性。

 - 主从配置
    > redis默认是主数据，所以master无需配置，我们只需要修改slave的配置即可。

```shell
    # 1. 首先启动主数据库
    $  ./redis-server ../redis.conf $

    # 2. 查看主数据库的key
    $ ./redis-cli -p 6379
    127.0.0.1:6379> keys *
    1) "person:3"
    2) "person:2"
    3) "mylist"
    4) "score:6"
    5) "pipeline"
    6) "person:6"
    7) "person:4"
    8) "sortedbymyhash"
    9) "person:9"
    10) "person:1"
    11) "person:7"
    12) "score:4"
    13) "score:2"
    14) "sortedbyscoreName"
    15) "score:8"
    16) "itemscore:2"
    17) "itemscore:4"
    18) "itemscore:1"
    19) "person:8"
    20) "itemscore:6"
    21) "person:5"
    22) "person:0"
    23) "myset"
    24) "itemscore:8"
    25) "myzset"
    26) "mylistalpha"

    # 3. 开启从数据库
    $ ./redis-server --port 6380 --slaveof 127.0.0.1 6379
    $ ./redis-cli -p 6380
    127.0.0.1:6380> keys *
    1) "mylistalpha"
    2) "myset"
    3) "sortedbymyhash"
    4) "person:9"
    5) "person:3"
    6) "person:0"
    7) "itemscore:2"
    8) "pipeline"
    9) "score:6"
    10) "score:8"
    11) "person:8"
    12) "score:2"
    13) "person:7"
    14) "person:2"
    15) "itemscore:6"
    16) "a"
    17) "score:4"
    18) "person:6"
    19) "myzset"
    20) "itemscore:4"
    21) "itemscore:1"
    22) "itemscore:8"
    23) "mylist"
    24) "person:1"
    25) "sortedbyscoreName"
    26) "person:5"
    27) "person:4"

    # 默认从数据库只读
    127.0.0.1:6380> set a 2
    (error) READONLY You can't write against a read only replica.
    
```

-  从数据库持久化
    > 另一个相对耗时的操作时持久化, 为了提高性能, 可以通过复制功能建立一个或者若干个从数据库, 并在从数据库中启动持久化, 同时在主数据库中禁用持久化. 当从数据库奔溃重启后自然会再次同步主数据库的数据. 但是当主数据库奔溃时, 情况就不一样了, 需要手工恢复:
    > - 在从数据库中使用slaveof no one 命令将从数据库提升为主数据库继续服务
    > - 启动之前奔溃的主数据库, 然后使用slaveof 将其设置成新的主数据库的从数据库, 即可将数据同步过来.

    但是, 手工恢复都是一件非常麻烦的事, 好在Redis为我们提供了自动化方案**哨兵**来解决这一问题.

- 无硬盘复制
    > master在内存中直接创建rdb，然后发送给slave，不会在自己本地落地磁盘了

    > repl-diskless-sync

    > repl-diskless-sync-delay，等待一定时长再开始复制，因为要等更多slave重新连接过来
### 13.2 哨兵模式


  

## 14. 管理
### 14.1 内部编码优化:
Redis 为每种数据类型都提供了两种内部编码方式. 比如散列类型是通过散列表实现的, 这样就可以实现$ O(1) $时间复杂度的查找或者赋值, 然而当键中元素很少的时候, $ O(1) $ 的操作并不会比$ O(n) $ 有明显的性能提高. 所以这种情况下Redis会采用一种更为紧凑但性能稍差的内部编码方式. 开发者并不需要去关注, 因为Redis会根据实际情况自动调整. 当键中元素变多redis会自动将该键的内部编码方式转换为散列表.
```shell
# 查看键内部编码方式
redis>OBJECT ENCODING
```
- 1. 字符串的内部编码

 字符串类型的内部编码有3种：

int：8个字节的长整型. 
embstr：小于等于39个字节的字符串. 
raw：大于39个字节的字符串. 
Redis会根据当前值的类型和长度决定使用内部编码实现. 

```shell
#整数类型示例如下：
127.0.0.1:6379> set str 1234567 
OK
127.0.0.1:6379> object encoding str
"int"
 
#短字符串示例如下：
127.0.0.1:6379> set str "hello world"
OK
127.0.0.1:6379> object encoding str
"embstr"
 
#长字符串示例如下：
127.0.0.1:6379> set str "Tranquil,unbeatable to the outside. -- yangming"  #“凝聚于内, 无敌于外. --王阳明”
OK
127.0.0.1:6379> object encoding str
"raw"
```

- 2. 哈希的内部编码

哈希类型的内部编码有两种：

ziplist(压缩列表)：当哈希类型元素个数小于hash-max-ziplist-entries配置(默认512个), 
　　　　同时所有值都小于hash-max-ziplist-value配置(默认64个字节)时, Redis会使用ziplist作为哈希的内部实现

　　　　ziplist使用更加紧凑的结构实现多个元素的连续存储, 所以在节省内存方面比hashtable更加优秀. 

hashtable(哈希表)：当哈希类型无法满足ziplist的条件时, Redis会使用hashtable作为哈希的内部实现. 
　　　　因为此时ziplist的读写效率会下降, 而hashtable的读写时间复杂度为O(1). 

下面演示哈希类型的内部编码, 及相应的变化. 
```shell
#当field个数比较少且没有大的value时, 内部编码为ziplist：
127.0.0.1:6379> hmset user:2 name kebi age 26
OK
127.0.0.1:6379> object encoding user:2
"ziplist"
 
#当有value大于64个字节, 内部编码会由ziplist变为hashtable：
127.0.0.1:6379> hmset user:1 info "沐春风, 惹一身红尘；望秋月, 化半缕轻烟. 顾盼间乾坤倒转, 一霎时沧海桑田. 方晓, 弹指红颜老, 刹那芳华逝. "
127.0.0.1:6379> object encoding user:1
"hashtable"
 
 #当field个数超过512, 内部编码也会由ziplist变为hashtable：
...待插入内容...
注意：当一个哈希的编码由ziplist变为hashtable的时候, 即使在替换掉所有值, 它一直都会是hashtable类型. 
```
 

- 3. 列表的内部编码
列表类型的内部编码有两种：

ziplist(压缩列表)：当哈希类型元素个数小于hash-max-ziplist-entries配置(默认512个)
　　　　同时所有值都小于hash-max-ziplist-value配置(默认64个字节)时, Redis会使用ziplist作为哈希的内部实现. 

linkedlist(链表)：当列表类型无法满足ziplist的条件时, Redis会使用linkedlist作为列表的内部实现. 
下面演示列表类型的内部编码, 以及相应的变化：
```shell
# 当元素个数较少且没有大元素时, 内部编码为ziplist：
127.0.0.1:6379> rpush list:2 a b c
(integer) 3
127.0.0.1:6379> object encoding list:2
"ziplist"
 
# 当元素个数超过512个, 内部编码变为linkedlist：

127.0.0.1:6379>lpush setkey 1 2 3 ... 513
OK
127.0.0.1:6379> object encoding listkey
"linkedlist"
 

# 当某个元素超过64个字节, 内部编码也会变为linkedlist：
127.0.0.1:6379> rpush list:1 a b "我不再说话, 不再思索, 但无尽的爱从灵魂中升起, 我将远行, 走得很远, 如同一个吉普塞人, 穿过大自然——幸福得如有一位女子同行. . "
(integer) 6
127.0.0.1:6379> object encoding list:1
"linkedlist"
#只能升级, 不能自动变回ziplist类型
```
 
- 4.集合的内部编码
集合类型的内部编码有两种：

intset(整数集合)：当集合中的元素都是整数且元素个数小于set-max-intset-entries配置(默认512个)时, 
　　　　Redis会选用intset来作为集合内部实现, 从而减少内存的使用. 

hashtable(哈希表)：当集合类型无法满足intset的条件时, Redis会使用hashtable作为集合的内部实现. 
下面用示例来说明：
```shell
#当元素个数较少且都为整数时, 内部编码为intset：
127.0.0.1:6379> sadd setkey 2 3 4 5
(integer) 4
127.0.0.1:6379> object encoding setkey
"intset"

#当元素个数超过512个, 内部编码变为hashtable：
127.0.0.1:6379>sadd setkey2 1 2 3 4 5 6 7...  511 512 513
OK
127.0.0.1:6379> object encoding setkey2
"hashtable"

#当某个元素不为整数时, 内部编码也会变为hashtable：
127.0.0.1:6379> sadd setkey3 a b c
(integer) 3
127.0.0.1:6379> object encoding setkey2
"hashtable"
 ```

- 5. 有序集合的内部编码

有序集合类型的内部编码有两种

ziplist(压缩列表)：当有序集合的元素个数小于zset-max-ziplist-entries配置(默认128个). 同时每个元素的值小于zset-max-ziplist-value配置(默认64个字节)时, Redis会用ziplist来作为有序集合的内部实现, ziplist可以有效减少内存使用. 

skiplist(跳跃表)：当ziplist条件不满足时, 有序集合会使用skiplist作为内部实现, 因为此时zip的读写效率会下降. 
下面用示例来说明:

```shell
  # 当元素个数较少且每个元素较小时，内部编码为ziplist：
  127.0.0.1:6379> zadd zsetkey 50 a 60 b 30 c
  (integer) 3
  127.0.0.1:6379> object encoding zsetkey
  "ziplist"

  # 当元素个数超过128个，内部编码变为skiplist：
  # ...待输入...

 # 当某个元素大于64个字节时，内部编码也会变为skiplist：
  127.0.0.1:6379> zadd zsetkey 50 a 60 b 30 '闪烁的太阳已越过高傲的山峦，幽谷中的光点有若泡沫浮起. '
  (integer) 1
  127.0.0.1:6379> object encoding zsetkey
  "skiplist"

```

### 14.2 安全

### 14.3 通信协议

```shell
  # 

```


