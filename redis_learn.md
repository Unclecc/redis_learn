# Redis Document
## 一. Redis 综合概述
## 二. Redis 搭建教程 -- centos7.2/ubuntu
- 下载地址：https://redis.io/download,下载最新稳定版本.
-  本教程使用的最新文档版本为 5.0.4，下载并安装：
```shell
  $ wget http://download.redis.io/releases/redis-5.0.4.tar.gz
  $ tar -zxvf redis-5.0.4.tar.gz
  $ cd redis-5.0.4
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

- 启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了.  比如：

```shell
  $ cd src
  $ ./redis-cli
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
- Redis 的配置文件位于 Redis 安装目录下，文件名为 redis.conf. 
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

#检查给定key是否存在.
exists key 

#为给定key设置过期时间, 以秒计.
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

#以毫秒为单位返回key的剩余的过期时间。
pttl key 

#以秒为单位，返回给定key的剩余生存时间(TT., time to live).
ttl key 

#从当前数据库中随机返回一个key.
randomkey  

#修改key的名称
rename key newkey 

#仅当newkey不存在时，将key改名为 newkey.
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

#将给定key的值设为value ，并返回key的旧值(old value).
getset key value

#对key所储存的字符串值，获取指定偏移量上的位(bit).
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

#这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。
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

#迭代哈希表中的键值对, cursor: 游标分页, 每次都会返回一个cursor的值, cursor=0表示迭代结束, match: 通配符, count: 表示每次返回的数据量, 默认值是10
HSCAN key cursor [MATCH pattern] [COUNT count] 
hscan myhash 0 match a* count 1
```

### 5.4 列表类型(List)
Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

一个列表最多可以包含 $2^{32} - 1$ 个元素 (4294967295, 每个列表超过40亿个元素).

由于是链表结构, 插入复杂度O(1), 获取越接近两端的数据速度越快. 

```shell
#移出并获取列表的第一个元素, 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
BLPOP key1 [key2 ] timeout 

#移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
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

#移除列表元素
LREM key count value 

#通过索引设置列表元素的值
LSET key index value 

#对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除
LTRIM key start stop 

#移除列表的最后一个元素，返回值为移除的元素
RPOP key 

#移除列表的最后一个元素，并将该元素添加到另一个列表并返回
RPOPLPUSH source destination 

#在列表中添加一个或多个值
RPUSH key value1 [value2] 

#为已存在的列表添加值
RPUSHX key value 

```

### 5.5 集合(Set)
Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据.

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1).

集合中最大的成员数为$2^{32}-1$ (4294967295, 每个集合可存储40多亿个成员).







