# Redis Document



## 一. Redis 综合概述 //TODO
## 二. Redis 搭建教程 -- centos7.2/ubuntu
- 下载地址：https://redis.io/download,下载最新稳定版本.
-  本教程使用的最新文档版本为 5.0.4, 下载并安装：
```shell
  $ wget http://download.redis.io/releases/redis-5.0.4.tar.gz
  $ tar -zxvf redis-5.0.4.tar.gz
  $ cd redis-5.0.4
  # 有些机器没有安装gcc, 会导致make报错, 可以yum install gcc 或者 sudo apt install gcc 删除原来的文件夹再解压一个
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

- 启动redis服务进程后, 就可以使用测试客户端程序redis-cli和redis服务交互了.  比如：

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
> Redis CONFIG 命令格式如下：
```shell
    # 获取redis配置
    redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
    # 更改redis配置 --> 并不是所有配置项都可以在命令行里修改
    redis 127.0.0.1:6379> CONFIG SET CONFIG_SETTING_NAME CONFIG_SETTING_VALUE
```
> 使用 * 号获取所有配置项：
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

  Redis 中每个 hash 可以存储 $2^{32} - 1$ 键值对（40多亿）.

  
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
Redis列表是简单的字符串列表, 按照插入顺序排序. 你可以添加一个元素到列表的头部（左边）或者尾部（右边）

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

> - 集合和有序集合的区别：
>> 1. 列表类型是通过链表实现的, 获取靠近两端的数据速度极快, 但是元素增多后, 访问中间数据的速度会很慢, 所以他更加适合实现如"新鲜事"或者"日志"这样很少访问中间元素的应用
>> 2. 有序集合是使用散列表和跳跃表(skip list)实现的, 所以读取中间的数据也很快$O(log(N))$
>>>其实在redis sorted sets里面当items内容大于64的时候同时使用了hash和skiplist两种设计实现. 这也会为了排序和查找性能做的优化. 所以如上可知： 
添加和删除都需要修改skiplist, 所以复杂度为$O(log(N))$.
但是如果仅仅是查找元素的话可以直接使用hash, 其复杂度为$O(1)$
其他的range操作复杂度一般为$O(log(N))$
当然如果是小于64的时候, 因为是采用了ziplist的设计, 其时间复杂度为$O(n)$
>> 3. 列表不能简单的调整某个元素的位置, 但是有序集合可以(通过更改元素分数)
>> 4. 有序类型要比列表类型更耗费内存
> - 集合和有序集合的相似点：
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

#迭代有序集合中的元素（包括元素成员和元素分值）
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
### 6.1 Redis 事务可以一次执行多个命令,  并且带有以下两个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存. 
- 收到 EXEC 命令后进入事务执行, 事务中任意命令执行失败, 其余的命令依然被执行. 
- 在事务执行过程, 其他客户端提交的命令请求不会插入到事务执行命令序列中. 

一个事务从开始到执行会经历以下三个阶段：
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
## 7 发布订阅 //TODO



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

// 输出结果:
score:4
score:6
itemscore:1
itemscore:2
mylist
myzset
score:2
itemscore:6
itemscore:4
score:8
mylistalpha
myset
itemscore:8
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

## 10. 管道 // TODO

## 11. Redis 脚本 //TODO
### 1. 客户端脚本
### 2. Jedis脚本
















