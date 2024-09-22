# redis教程



## 1、简介

Redis 是一个高性能的 key/value 数据库。它是完全开源免费的，并且遵守 BSD 协议。

### Redis 特点

1. 不仅支持 key/value 类型的数据，也支持 list,hash,set,zset 等等数据结构。
2. 支持持久化，可以把内存数据保存到磁盘上，重启后再次加载到内存中使用。
3. 支持备份，可以使用主/从模式进行数据备份。

### Redis 优势

1. **数据类型丰富 ：** Redis 支持 Strings, Lists, Hash, Set 及 Ordered Sets 数据类型操作。
2. **高性能 ：** Redis 能读的速度是 110000 次/s ,写的速度是 81000 次/s。
3. **原子型操作 :** Redis 的所有操作都是原子性的，还支持对几个操作完成后的原子性执行。
4. **丰富的特性 :** Redis 支持 publish/subscribe, 通知, key 过期等等特性。



### Redis 安装

**redis.conf** 是一个默认的配置文件 我们可以根据需要使用自己的配置文件

redis.conf

```shell
# Redis configuration file example.
#
# Note that in order to read the configuration file, Redis must be
# started with the file path as first argument:
#
# 开始启动时必须如下指定配置文件

# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 存储单位如下所示

# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes

################################## INCLUDES ###################################

# 如果需要使用多配置文件配置redis，请用include
#
# include /path/to/local.conf
# include /path/to/other.conf

################################## MODULES ##################################### modules

# 手动设置加载模块（当服务无法自动加载时设置）
#
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so

################################## NETWORK #####################################

# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# 设置绑定的ip
bind 127.0.0.1

# 保护模式：不允许外部网络连接redis服务
protected-mode yes

# 设置端口号
port 6379

# TCP listen() backlog.
#
# TCP 连接数，此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度
tcp-backlog 511

# Unix socket.
#
# 通信协议设置，本机通信使用此协议不适用tcp协议可大大提升性能
# unixsocket /tmp/redis.sock
# unixsocketperm 700



# TCP keepalive.
#
# 定期检测cli连接是否存活
tcp-keepalive 300

################################# GENERAL #####################################

# 是否守护进程运行（后台运行）
daemonize yes

# 是否通过upstart和systemd管理Redis守护进程
supervised no

# 以后台进程方式运行redis，则需要指定pid 文件
pidfile /var/run/redis_6379.pid

# 日志级别
# 可选项有： # debug（记录大量日志信息，适用于开发、测试阶段）； # verbose（较多日志信息）； # notice（适量日志信息，使用于生产环境）；
# warning（仅有部分重要、关键信息才会被记录）。
loglevel notice

# 日志文件的位置
logfile ""

# 数据库的个数
databases 16

# 是否显示logo
always-show-logo yes

################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
# 持久化操作设置 900秒内触发一次请求进行持久化，300秒内触发10次请求进行持久化操作，60s内触发10000次请求进行持久化操作

save 900 1
save 300 10
save 60 10000

# 持久化出现错误后，是否依然进行继续进行工作
stop-writes-on-bgsave-error yes

# 使用压缩rdb文件 yes：压缩，但是需要一些cpu的消耗。no：不压缩，需要更多的磁盘空间
rdbcompression yes

# 是否校验rdb文件，更有利于文件的容错性，但是在保存rdb文件的时候，会有大概10%的性能损耗
rdbchecksum yes

# dbfilename的文件名
dbfilename dump.rdb

# dbfilename文件的存放位置
dir ./

################################# REPLICATION #################################

# replicaof 即slaveof 设置主结点的ip和端口
# replicaof <masterip> <masterport>

# 集群节点访问密码
# masterauth <master-password>

# 从结点断开后是否仍然提供数据
replica-serve-stale-data yes

# 设置从节点是否只读
replica-read-only yes

# 是或否创建新进程进行磁盘同步设置
repl-diskless-sync no

# master节点创建子进程前等待的时间
repl-diskless-sync-delay 5

# Replicas发送PING到master的间隔，默认值为10秒。
# repl-ping-replica-period 10

#
# repl-timeout 60

#
repl-disable-tcp-nodelay no

#
# repl-backlog-size 1mb

#
# repl-backlog-ttl 3600

#
replica-priority 100

#
# min-replicas-to-write 3
# min-replicas-max-lag 10
#
# replica-announce-ip 5.5.5.5
# replica-announce-port 1234

################################## SECURITY ###################################

# 设置连接时密码
# requirepass 123456

################################### CLIENTS ####################################

# 最大连接数
# maxclients 10000

############################## MEMORY MANAGEMENT ################################

# redis配置的最大内存容量
# maxmemory <bytes>

# 内存达到上限的处理策略
# maxmemory-policy noeviction

# 处理策略设置的采样值
# maxmemory-samples 5

# 是否开启 replica 最大内存限制
# replica-ignore-maxmemory yes

############################# LAZY FREEING ####################################

# 惰性删除或延迟释放
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

############################## APPEND ONLY MODE ###############################

# 是否使用AOF持久化方式
appendonly no

# appendfilename的文件名

appendfilename "appendonly.aof"

# 持久化策略
# appendfsync always
appendfsync everysec
# appendfsync no

# 持久化时（RDB的save | aof重写）是否可以运用Appendfsync，用默认no即可，保证数据安全性
no-appendfsync-on-rewrite no

# 设置重写的基准值
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 指定当发生AOF文件末尾截断时，加载文件还是报错退出
aof-load-truncated yes

# 开启混合持久化，更快的AOF重写和启动时数据恢复
aof-use-rdb-preamble yes

################################ REDIS CLUSTER  ###############################

# 是否开启集群
# cluster-enabled yes

# 集群结点信息文件
# cluster-config-file nodes-6379.conf

# 等待节点回复的时限
# cluster-node-timeout 15000

# 结点重连规则参数
# cluster-replica-validity-factor 10

#
# cluster-migration-barrier 1

#
# cluster-require-full-coverage yes

#
# cluster-replica-no-failover no
```

### 远程 redis-cli 语法

```shell
$ redis-cli -h host -p port -a password
```

```shell
h ip 地址

p – 端口

a – 密码
```





## 2、数据类型

### Redis 支持七种数据类型

1. string ( 字符串 )
2. hash ( 哈希 )
3. list ( 列表 )
4. set ( 集合 )
5. zset ( sorted set：有序集合 )
6. Bitmaps ( 位图 )
7. HyperLogLogs ( 基数统计 )



### String（字符串）

1. string 是 Redis 最基本的数据类型，key/value。
2. string 类型的一个键最大能存储**512 MB** 数据。
3. Redis 的 string 可以包含任何数据，比如 jpg 图片或者序列化的对象。
4. string 类型是二进制安全的。

使用 Redis 的 **SET** 和 **GET** 命令来进行设置和读取字符串。

```shell
127.0.0.1:6379> set name 面向加薪学习
OK
127.0.0.1:6379> get name
"\xe9\x9d\xa2\xe5\x90\x91\xe5\x8a\xa0\xe8\x96\xaa\xe5\xad\xa6\xe4\xb9\xa0"
```



### Hash（哈希）

1. Redis Hash 是一个 string 类型的 field 和 value 的映射表。
2. 每个 hash 可以存储 232-1 键值对（40 多亿）。
3. Hash 适合用于存储对象。

```shell
127.0.0.1:6379> hmset account:1 name huanxi password 123456 fav travel
OK
127.0.0.1:6379> hgetall account:1
1) "name"
2) "huanxi"
3) "password"
4) "123456"
5) "fav"
6) "travel"
```

使用 Redis **HMSET, HGETALL** 命令， **account:1** 为键。



### List（列表）

1. List 是简单的字符串列表，按照插入顺序排序。
2. 向列表中添加一个元素，可以从列表的头部 ( 左边 ) 或者尾部 ( 右边 )开始。

```shell
127.0.0.1:6379> rpush study Go语言极简一本通 Go语言微服务核心架构22讲 从0到Go语言微服务架构师
(integer) 3
127.0.0.1:6379> lrange study 0 10
1) "Go\xe8\xaf\xad\xe8\xa8\x80\xe6\x9e\x81\xe7\xae\x80\xe4\xb8\x80\xe6\x9c\xac\xe9\x80\x9a"
2) "Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\xa0\xb8\xe5\xbf\x83\xe6\x9e\xb6\xe6\x9e\x8422\xe8\xae\xb2"
3) "\xe4\xbb\x8e0\xe5\x88\xb0Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe5\xb8\x88"

1 Go语言极简一本通
2 Go语言微服务核心架构22讲
3 从0到Go语言微服务架构师
```



### Set（集合）

1. Set 是 string 类型的无序集合
2. Set 内元素不可重复，无论插入多少次，只会保留一份。
3. Set 是通过哈希表实现的，所以添加，删除，查找的时间复杂度都是 O(1)。

#### sadd 命令

sadd 添加一个 string 元素到 set 集合中。

Redis sadd 语法

```shell
sadd key member
```

例子

```shell
127.0.0.1:6379> sadd gostudy Go语言极简一本通
(integer) 1
127.0.0.1:6379> sadd gostudy Go语言微服务核心架构22讲
(integer) 1
127.0.0.1:6379> sadd gostudy 从0到Go语言微服务架构师
(integer) 1
127.0.0.1:6379> sadd gostudy Go语言极简一本通
(integer) 0

127.0.0.1:6379> smembers gostudy
1) "\xe4\xbb\x8e0\xe5\x88\xb0Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe5\xb8\x88"
2) "Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\xa0\xb8\xe5\xbf\x83\xe6\x9e\xb6\xe6\x9e\x8422\xe8\xae\xb2"
3) "Go\xe8\xaf\xad\xe8\xa8\x80\xe6\x9e\x81\xe7\xae\x80\xe4\xb8\x80\xe6\x9c\xac\xe9\x80\x9a"
```

Go 语言极简一本通 添加了两次，但最后只存储了一份。



### zset ( sorted set：有序集合 )

zset 和 set 一样也是 string 类型元素的集合。不同的是 zset 中的每个元素都有自己的分数（double 类型），通过分数来对集合中的元素进行排序。这个分数是不重复的。

#### Redis zadd 命令

zset 添加元素到集合中，如果元素在集合中存在则更新对应分数（score）。

#### Redis zadd 命令语法格式

```
zadd key score member
```

例子

```shell
127.0.0.1:6379> zadd studymap 1 Go语言极简一本通
(integer) 1
127.0.0.1:6379> zadd studymap 2 Go语言微服务架构核心22讲
(integer) 1
127.0.0.1:6379> zadd studymap 3 从0到Go语言微服务架构师
(integer) 1
127.0.0.1:6379> zadd studymap 4 Go语言极简一本通
(integer) 0

zrangebyscore studymap 0 10
1) "Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe6\xa0\xb8\xe5\xbf\x8322\xe8\xae\xb2"
2) "\xe4\xbb\x8e0\xe5\x88\xb0Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe5\xb8\x88"
3) "Go\xe8\xaf\xad\xe8\xa8\x80\xe6\x9e\x81\xe7\xae\x80\xe4\xb8\x80\xe6\x9c\xac\xe9\x80\x9a"
```

golang 被添加 2 次，但是最后只存储了一份。



### Redis Bitmap ( 位图 )

Bitmap 通过类似 map 结构存放 0 或 1 ( bit 位 ) 作为值。可以用来统计状态，如 日活，打卡，浏览量等。

#### setbit 命令

setbit 命令用于设置或者清除一个 bit 位



#### setbit 命令语法格式

```shell
SETBIT key offset value
```

例子

```shell
127.0.0.1:6379> setbit aa 10001 1 # 返回操作之前的数值
(integer) 0
127.0.0.1:6379> setbit aa 10001 2 # 如果值不是0或1就报错
(error) ERR bit is not an integer or out of range
127.0.0.1:6379> setbit aa 10001 0
(integer) 1
127.0.0.1:6379> setbit aa 10001 1
(integer) 0
127.0.0.1:6379> getbit aa 10001
(integer) 1
```



## 3、命令

更多命令请参考：<https://redis.io/commands>

下表列出了 Redis 键相关的命令

| 命令      | 描述                                                 |
| --------- | ---------------------------------------------------- |
| DEL       | 用于删除 key                                         |
| DUMP      | 序列化给定 key ，并返回被序列化的值                  |
| EXISTS    | 检查给定 key 是否存在                                |
| EXPIRE    | 为给定 key 设置过期时间                              |
| EXPIREAT  | 用于为 key 设置过期时间 接受的时间参数是 UNIX 时间戳 |
| PEXPIRE   | 设置 key 的过期时间，以毫秒计                        |
| PEXPIREAT | 设置 key 过期时间的时间戳(unix timestamp)，以毫秒计  |
| KEYS      | 查找所有符合给定模式的 key                           |
| MOVE      | 将当前数据库的 key 移动到给定的数据库中              |
| PERSIST   | 移除 key 的过期时间，key 将持久保持                  |
| PTTL      | 以毫秒为单位返回 key 的剩余的过期时间                |
| TTL       | 以秒为单位，返回给定 key 的剩余生存时间(             |
| RANDOMKEY | 从当前数据库中随机返回一个 key                       |
| RENAME    | 修改 key 的名称                                      |
| RENAMENX  | 仅当 newkey 不存在时，将 key 改名为 newkey           |
| TYPE      | 返回 key 所储存的值的类型                            |



### 操作 key 语法

```
COMMAND KEY的名称
```

例子

```
127.0.0.1:6379> SET web www.go-edu.cn
OK
127.0.0.1:6379> get web
"www.go-edu.cn"
127.0.0.1:6379> del web
(integer) 1
127.0.0.1:6379> get web
(nil)
```



## 4、Redis 如何操作字符串

更多命令请参考：<https://redis.io/commands>

| 命令        | 描述                                                        |
| ----------- | ----------------------------------------------------------- |
| SET         | 设置指定 key 的值                                           |
| GET         | 获取指定 key 的值                                           |
| GETRANGE    | 返回 key 中字符串值的子字符                                 |
| GETSET      | 将给定 key 的值设为 value ，并返回 key 的旧值 ( old value ) |
| GETBIT      | 对 key 所储存的字符串值，获取指定偏移量上的位 ( bit )       |
| MGET        | 获取所有(一个或多个)给定 key 的值                           |
| SETBIT      | 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)    |
| SETEX       | 设置 key 的值为 value 同时将过期时间设为 seconds            |
| SETNX       | 只有在 key 不存在时设置 key 的值                            |
| SETRANGE    | 从偏移量 offset 开始用 value 覆写给定 key 所储存的字符串值  |
| STRLEN      | 返回 key 所储存的字符串值的长度                             |
| MSET        | 同时设置一个或多个 key-value 对                             |
| MSETNX      | 同时设置一个或多个 key-value 对                             |
| PSETEX      | 以毫秒为单位设置 key 的生存时间                             |
| INCR        | 将 key 中储存的数字值增一                                   |
| INCRBY      | 将 key 所储存的值加上给定的增量值 ( increment )             |
| INCRBYFLOAT | 将 key 所储存的值加上给定的浮点增量值 ( increment )         |
| DECR        | 将 key 中储存的数字值减一                                   |
| DECRBY      | 将 key 所储存的值减去给定的减量值 ( decrement )             |
| APPEND      | 将 value 追加到 key 原来的值的末尾                          |

例子

```shell
127.0.0.1:6379> setex channelname 60 面向加薪学习
OK

127.0.0.1:6379> ttl channelname
(integer) 40

超过60秒后
127.0.0.1:6379> get channelname
(nil)

127.0.0.1:6379> setnx author huanxi
(integer) 1

第2次执行失败
127.0.0.1:6379> setnx author huanxi
(integer) 0

127.0.0.1:6379> incr spend
(integer) 1
127.0.0.1:6379> incr spend
(integer) 2
127.0.0.1:6379> incr spend
(integer) 3

127.0.0.1:6379> get spend
"3"

127.0.0.1:6379> getset language java
(nil)
127.0.0.1:6379> getset language go
"java"
127.0.0.1:6379> getset language rust
"go"
```





## 5、哈希操作

hash 是一个 string 类型的 field 和 value 的映射表。特别适合用于存储对象。

更多命令请参考：<https://redis.io/commands>



### Redis hash 命令

下表列出了 redis hash 命令

| HDEL         | 删除一个或多个哈希表字段                                  |
| ------------ | --------------------------------------------------------- |
| HEXISTS      | 查看哈希表 key 中，指定的字段是否存在                     |
| HGET         | 获取存储在哈希表中指定字段的值                            |
| HGETALL      | 获取在哈希表中指定 key 的所有字段和值                     |
| HINCRBY      | 为哈希表 key 中的指定字段的整数值加上增量 increment       |
| HINCRBYFLOAT | 为哈希表 key 中的指定字段的浮点数值加上增量 increment     |
| HKEYS        | 获取所有哈希表中的字段                                    |
| HLEN         | 获取哈希表中字段的数量                                    |
| HMGET        | 获取所有给定字段的值                                      |
| HMSET        | 同时将多个 field-value (域-值)对设置到哈希表 key 中       |
| HSET         | 将哈希表 key 中的字段 field 的值设为 value                |
| HSETNX       | 只有在字段 field 不存在时，设置哈希表字段的值             |
| HVALS        | 获取哈希表中所有值                                        |
| HSCAN        | 迭代哈希表中的键值对                                      |
| HSTRLEN      | 返回哈希表 key 中， 与给定域 field 相关联的值的字符串长度 |



```
127.0.0.1:6379> hmset gostudy3 k1 Go语言极简一本通 k2 Go语言微服务架构核心22讲 k3 从0到Go语言微服务架构师
OK
127.0.0.1:6379> hlen gostudy3
(integer) 3
127.0.0.1:6379> hkeys gostudy3
1) "k1"
2) "k2"
3) "k3"
127.0.0.1:6379> hgetall gostudy3
1) "k1"
2) "Go\xe8\xaf\xad\xe8\xa8\x80\xe6\x9e\x81\xe7\xae\x80\xe4\xb8\x80\xe6\x9c\xac\xe9\x80\x9a"
3) "k2"
4) "Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe6\xa0\xb8\xe5\xbf\x8322\xe8\xae\xb2"
5) "k3"
6) "\xe4\xbb\x8e0\xe5\x88\xb0Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe5\xb8\x88"

127.0.0.1:6379> hset gostudy3 k4 java
(integer) 1

127.0.0.1:6379> hgetall gostudy3
1) "k1"
2) "Go\xe8\xaf\xad\xe8\xa8\x80\xe6\x9e\x81\xe7\xae\x80\xe4\xb8\x80\xe6\x9c\xac\xe9\x80\x9a"
3) "k2"
4) "Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe6\xa0\xb8\xe5\xbf\x8322\xe8\xae\xb2"
5) "k3"
6) "\xe4\xbb\x8e0\xe5\x88\xb0Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe5\xb8\x88"
7) "k4"
8) "java"

127.0.0.1:6379> hgetall gostudy3
1) "k1"
2) "Go\xe8\xaf\xad\xe8\xa8\x80\xe6\x9e\x81\xe7\xae\x80\xe4\xb8\x80\xe6\x9c\xac\xe9\x80\x9a"
3) "k2"
4) "Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe6\xa0\xb8\xe5\xbf\x8322\xe8\xae\xb2"
5) "k3"
6) "\xe4\xbb\x8e0\xe5\x88\xb0Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe5\xb8\x88"
```



## 6、列表操作

列表(List)是简单的字符串列表，按照插入顺序排序。 可以添加一个元素到列表的头部（左边）或者尾部（右边）。



### Redis 列表命令

下表列出了列表相关命令

| 命令       | 描述                                                     |
| ---------- | -------------------------------------------------------- |
| BLPOP      | 移出并获取列表的第一个元素                               |
| BRPOP      | 移出并获取列表的最后一个元素                             |
| BRPOPLPUSH | 从列表中弹出一个值，并将该值插入到另外一个列表中并返回它 |
| LINDEX     | 通过索引获取列表中的元素                                 |
| LINSERT    | 在列表的元素前或者后插入元素                             |
| LLEN       | 获取列表长度                                             |
| LPOP       | 移出并获取列表的第一个元素                               |
| LPUSH      | 将一个或多个值插入到列表头部                             |
| LPUSHX     | 将一个值插入到已存在的列表头部                           |
| LRANGE     | 获取列表指定范围内的元素                                 |
| LREM       | 移除列表元素                                             |
| LSET       | 通过索引设置列表元素的值                                 |
| LTRIM      | 对一个列表进行修剪(trim)                                 |
| RPOP       | 移除并获取列表最后一个元素                               |
| RPOPLPUSH  | 移除列表的最后一个元素，并将该元素添加到另一个列表并返回 |
| RPUSH      | 在列表中添加一个或多个值                                 |
| RPUSHX     | 为已存在的列表添加值                                     |



```shell
127.0.0.1:6379> lpush studylist Go语言极简一本通 Go语言微服务架构核心22讲 从0到Go语言微服务架构师 java rust
(integer) 5

127.0.0.1:6379> lrange studylist 0 10
1) "rust"
2) "java"
3) "\xe4\xbb\x8e0\xe5\x88\xb0Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe5\xb8\x88"
4) "Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe6\xa0\xb8\xe5\xbf\x8322\xe8\xae\xb2"
5) "Go\xe8\xaf\xad\xe8\xa8\x80\xe6\x9e\x81\xe7\xae\x80\xe4\xb8\x80\xe6\x9c\xac\xe9\x80\x9a"

127.0.0.1:6379> lindex studylist 3
"Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe6\xa0\xb8\xe5\xbf\x8322\xe8\xae\xb2"

127.0.0.1:6379> lpop studylist 1
1) "rust"

127.0.0.1:6379> lrem studylist 1 java
(integer) 1

linsert studylist before Go语言极简一本通 golang
(integer) 4

127.0.0.1:6379> lrange studylist 0 10
1) "\xe4\xbb\x8e0\xe5\x88\xb0Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe5\xb8\x88"
2) "Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe6\xa0\xb8\xe5\xbf\x8322\xe8\xae\xb2"
3) "golang"
4) "Go\xe8\xaf\xad\xe8\xa8\x80\xe6\x9e\x81\xe7\xae\x80\xe4\xb8\x80\xe6\x9c\xac\xe9\x80\x9a"

127.0.0.1:6379> lset studylist 3 go语言
OK
127.0.0.1:6379> lrange studylist 0 10
1) "\xe4\xbb\x8e0\xe5\x88\xb0Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe5\xb8\x88"
2) "Go\xe8\xaf\xad\xe8\xa8\x80\xe5\xbe\xae\xe6\x9c\x8d\xe5\x8a\xa1\xe6\x9e\xb6\xe6\x9e\x84\xe6\xa0\xb8\xe5\xbf\x8322\xe8\xae\xb2"
3) "golang"
4) "go\xe8\xaf\xad\xe8\xa8\x80"
```



## 7、集合(Set)操作

1. set 是 string 类型的无序集合。
2. set 是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。
3. 集合成员是唯一的，不能出现重复的数据。

下表列出了 Redis 集合相关命令

| 命令        | 描述                                                |
| ----------- | --------------------------------------------------- |
| SADD        | 向集合添加一个或多个成员                            |
| SCARD       | 获取集合的成员数                                    |
| SDIFF       | 返回给定所有集合的差集                              |
| SDIFFSTORE  | 返回给定所有集合的差集并存储在 destination 中       |
| SINTER      | 返回给定所有集合的交集                              |
| SISMEMBER   | 判断 member 元素是否是集合 key 的成员               |
| SMEMBERS    | 返回集合中的所有成员                                |
| SMOVE       | 将 member 元素从 source 集合移动到 destination 集合 |
| SPOP        | 移除并返回集合中的一个随机元素                      |
| SRANDMEMBER | 返回集合中一个或多个随机数                          |
| SREM        | 移除集合中一个或多个成员                            |
| SUNION      | 返回所有给定集合的并集                              |
| SUNIONSTORE | 所有给定集合的并集存储在 destination 集合中         |
| SSCAN       | 迭代集合中的元素                                    |

通过 **SADD** 命令向名为 **skill** 的集合插入的三个元素。java 添加 2 次，但是集合中仍然只有一个。

## 8、有序集合（sorted set）操作



1. sorted set 和 set 一样也是 string 类型元素的集合，元素不能重复。
2. sorted set 的每个元素都会有一个 double 类型的分数(score)，元素是唯一的,但分数 (score) 却可以重复。
3. sorted set 通过分数(score) 给集合中的元素进行从小到大的排序
4. sorted set 是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)

### Redis 有序集合命令

下表列出了 Redis 有序集合的基本命令

| 命令             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| ZADD             | 向有序集合添加一个或多个成员，或者更新已存在成员的分数       |
| ZCARD            | 获取有序集合的成员数                                         |
| ZCOUNT           | 计算在有序集合中指定区间分数的成员数                         |
| ZINCRBY          | 有序集合中对指定成员的分数加上增量 increment                 |
| ZINTERSTORE      | 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中 |
| ZLEXCOUNT        | 在有序集合中计算指定字典区间内成员数量                       |
| ZRANGE           | 通过索引区间返回有序集合成指定区间内的成员                   |
| ZRANGEBYLEX      | 通过字典区间返回有序集合的成员                               |
| ZRANGEBYSCORE    | 通过分数返回有序集合指定区间内的成员                         |
| ZRANK            | 返回有序集合中指定成员的索引                                 |
| ZREM             | 移除有序集合中的一个或多个成员                               |
| ZREMRANGEBYLEX   | 移除有序集合中给定的字典区间的所有成员                       |
| ZREMRANGEBYRANK  | 移除有序集合中给定的排名区间的所有成员                       |
| ZREMRANGEBYSCORE | 移除有序集合中给定的分数区间的所有成员                       |
| ZREVRANGE        | 返回有序集中指定区间内的成员，通过索引，分数从高到底         |
| ZREVRANGEBYSCORE | 返回有序集中指定分数区间内的成员，分数从高到低排序           |
| ZREVRANK         | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| ZSCORE           | 返回有序集中，成员的分数值                                   |
| ZUNIONSTORE      | 计算一个或多个有序集的并集，并存储在新的 key 中              |
| ZSCAN            | 迭代有序集合中的元素（包括元素成员和元素分值）               |



```shell
127.0.0.1:6379> zadd zskill 1 golang 2 rust 3 java 4 typescript 5 flutter
(integer) 5

127.0.0.1:6379> zcount zskill 1 3
(integer) 3

127.0.0.1:6379> zrange zskill 1 3 withscores
1) "rust"
2) "2"
3) "java"
4) "3"
5) "typescript"
6) "4"

127.0.0.1:6379> zrevrange zskill 1 3 withscores
1) "typescript"
2) "4"
3) "java"
4) "3"
5) "rust"
6) "2"
```





## 9、事务

简单说，事务就是一次执行多个命令，并且要么都成功，要么都失败。

### 事务的特点

1. 原子性：要么全部执行，要么全部不执行。
2. 事务中的命令是按顺序执行的，在事务执行的过程中，不会被其他客户端发来的命令中断。



### Redis 开启事务

multi（开启事务）

命令入队

exec（执行事务）

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set web www.go-edu.cn
QUEUED
127.0.0.1:6379(TX)> get web
QUEUED
127.0.0.1:6379(TX)> sadd lesson golang redis web flutter
QUEUED
127.0.0.1:6379(TX)> smembers lesson
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
2) "www.go-edu.cn"
3) (integer) 4
4) 1) "golang"
   2) "flutter"
   3) "redis"
   4) "web"
```



### Redis 事务命令

| 命令    | 描述                                 |
| ------- | ------------------------------------ |
| MULTI   | 标记一个事务块的开始                 |
| EXEC    | 执行所有事务块内的命令               |
| WATCH   | 监视一个(或多个) key                 |
| UNWATCH | 取消 WATCH 命令对所有 key 的监视     |
| DISCARD | 取消事务，放弃执行事务块内的所有命令 |



## 10、发布订阅

发布订阅(pub/sub)是一种常见的消息传递模式。比如我们订阅的微信公众号，只要你订阅的公众号，有新文章发布，都会推送给你。

![redis 发布订阅](redis教程.assets/e6c9d24ely1h1p7e20o4yj21mw0pqgnl.jpg)

### Redis 发布订阅命令

下表列出了 redis 发布订阅相关的命令

| 命令         | 描述               |
| ------------ | ------------------ |
| PSUBSCRIBE   | 订阅一个或多个消息 |
| PUBSUB       | 查看订阅与发布状态 |
| PUBLISH      | 发送信息           |
| PUNSUBSCRIBE | 退订多个消息       |
| SUBSCRIBE    | 订阅一个消息       |
| UNSUBSCRIBE  | 退订消息           |

### 发布订阅消息案例

```
127.0.0.1:6379> subscribe huanxi
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "huanxi"
3) (integer) 1
```

发布消息（重新打开一个 redis-cli）

```
127.0.0.1:6379> publish huanxi "www.go-edu.cn"
(integer) 1
```

订阅方接收消息

```
1) "message"
2) "huanxi"
3) "www.go-edu.cn"
```



## 11、redis 管理命令

| 命令             | 描述                                             |
| ---------------- | ------------------------------------------------ |
| INFO             | 查看 Redis 服务器的各种信息                      |
| SAVE             | 异步保存数据到硬盘                               |
| BGSAVE           | 在后台异步保存当前数据库的数据到磁盘             |
| TIME             | 返回当前服务器时间                               |
| DBSIZE           | 返回当前数据库的 key 的数量                      |
| BGREWRITEAOF     | 异步执行一个 AOF（AppendOnly File） 文件重写操作 |
| CLIENT           | 客户端连接                                       |
| CLIENT LIST      | 获取客户端连接列表                               |
| CLIENT GETNAME   | 获取客户端的名称                                 |
| CLIENT PAUSE     | 在指定时间内暂停运行来自客户端的命令             |
| CLIENT SETNAME   | 设置当前连接的名称                               |
| CLUSTER SLOTS    | 获取集群节点的映射数组                           |
| COMMAND          | Redis 命令                                       |
| COMMAND COUNT    | 获取 Redis 命令总数                              |
| COMMAND GETKEYS  | 获取给定命令的所有键                             |
| COMMAND INFO     | 获取指定 Redis 命令描述信息                      |
| CONFIG GET       | 获取指定配置参数的值                             |
| CONFIG REWRITE   | 修改 redis.conf 配置文件                         |
| CONFIG SET       | 修改 redis 配置参数，无需重启                    |
| CONFIG RESETSTAT | 重置 INFO 命令中的某些统计数据                   |
| DEBUG OBJECT     | 获取 key 的调试信息                              |
| DEBUG SEGFAULT   | 让 Redis 服务崩溃                                |
| FLUSHALL         | 删除所有数据库的所有 key                         |
| FLUSHDB          | 删除当前数据库的所有 key                         |
| LASTSAVE         | 返回最近一次 Redis 成功将数据保存到磁盘上的时间  |
| MONITOR          | 实时打印出 Redis 服务器接收到的命令，调试用      |
| ROLE             | 返回主从实例所属的角色                           |
| SHUTDOWN         | 异步保存数据到硬盘，并关闭服务器                 |
| SLAVEOF          | 将当前服务器转变从属服务器(slave server)         |
| SLOWLOG          | 管理 redis 的慢日志                              |
| SYNC             | 用于复制功能 ( replication ) 的内部命令          |

### 查看备份文件目录

```
127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/data"
```

### 数据备份

**SAVE** 命令：创建当前数据库的备份。将在 Redis 安装目录中创建 **dump.rdb** 文件。

```
127.0.0.1:6379> save
OK
```

**SAVE** 会在前台执行，如果数据量巨大，可能会堵塞 Redis 服务，使用 **BGSAVE** 命令用于后台运行备份数据库。

```
127.0.0.1:6379> bgsave
Background saving started
```

### 恢复数据

只需将备份文件 ( **dump.rdb** ) 移动到 Redis 目录（CONFIG GET dir 命令获得的目录）并启动服务即可。



## 12、Redis 分区

分区是分割数据到多个 Redis 实例中去，分区后，每个 Redis 实例只保存 key 的一个子集（一部分）。

### 分区的好处

1. 通过利用多台计算机内存，可以存储更多的数据。
2. 通过多核和多台计算机，提供更高的性能。
3. 通过多台计算机和网络适配器，提供更强的传输能力。

### 分区的劣势

1. 不支持多个 Key 的操作。
2. 不支持多个 key 的事务操作。
3. 分区后，处理数据变得复杂（要处理不同机器上的 rdb/aof 文件）。

### 分区类型

1. 范围分区，是最简单的分区方式，比如从 id 1 到 100 的订单保存到分区 1,101 到 200 的订单保存到分区 2。不足是每增加区间范围都要维护一个映射表，要记得，哪个分区和哪个范围有联系。

2. 哈希分区，比范围分区更好的方法，可以对任何 key 都适用，key 不需要是 object_name:这样的形式。

   >1. Hash(key),用 hash 函数对 key 进行计算出一个整数。
   >2. 对这个整数取模操作得到一个数值。
   >3. 将上一步的结果映射到对应的实例中。

   

   比如我们分区是 16 个，我们算出来的 hash 数值是 83084917

   ```
   83084917 % 16 = 5
   ```

   那么我们就应该把 key 存储到 实例 5 中。



## 13、AOF 和 RDB



redis 的两种持久化方式:

- AOF
- RDB

### RDB

在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。RDB 持久化主要是通过 SAVE 和 BGSAVE 两个命令对 Redis 数据库中当前的数据做 snapshot 并生成 rdb 文件来实现的。其中 SAVE 是阻塞的，BGSAVE 是非阻塞的，通过 fork 了一个子进程来完成的。在 Redis 启动的时候会检测 rdb 文件，然后载入 rdb 文件中未过期的数据到服务器中。

```shell

save 900 1    //Redis 服务器在 900 秒之内，对数据库进行了至少一次修改
save 300 10   //Redis 服务器在 300 秒之内，对数据库进行了至少 10 次修改
save 60 10000 //Redis 服务器在 60 秒之内，对数据库进行了至少 10000 次修改
```



#### RDB 优点

1. 它是非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。 采用子线程创建 RDB 文件，不会对 redis 服务器性能造成大的影响。
2. RDB 非常适用于灾难恢复（disaster recovery）：它只有一个文件，并且内容都非常紧凑，可以（在加密后）将它传送到别的数据中心。RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。
3. Redis 父进程在保存 RDB 文件时,会 fork  出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。

#### RDB 缺点

1. redis 文件在某个时间点，生成之后产生了新数据，还未到达另一次生成 RDB 文件的条件，如果这个期间 redis 服务器崩溃，那么会导致数据丢失，如果数据一致性要求不高，可以用生成 RDB 的方式。
2. 快照持久化方法通过调用 fork()方法创建子线程。当 redis 内存的数据量比较大时，创建子线程和生成 RDB 文件会占用大量的系统资源和处理时间，对 redis 处理正常的客户端请求造成较大影响。



### AOF

AOF 是 redis 对将所有的写命令保存到一个 aof 文件中，根据这些写命令，实现数据的持久化和数据恢复。

AOF 方式是一条一条的写命令，随着服务器运行的时间越来越长，AOF 文件会越来越大，AOF 重写就是为了解决这个问题。AOF（Append Only File）持久化是通过将存储每次执行的客户端命令，然后由一个伪客户端来执行这些命令将数据写入到服务器中的方式实现的。一共分为命令追加（append）、文件写入、文件同步（sync）三个步骤。



#### AOF 优点

1. AOF 的默认策略为每秒钟同步一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（  同步会在后台线程执行，所以主线程可以继续努力地处理命令请求）。
2. AOF 文件是一个只进行追加操作的日志文件（append only log）， 因此对 AOF 文件的写入不需要进行  seek ， 即使日志因为某些原因而包含了未写入完整的命令（比如写入时磁盘已满，写入中途停机，等等）， redis-check-aof  工具也可以修复这种问题。
3. Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
4. AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了  FLUSHALL  命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的  FLUSHALL  命令， 并重启 Redis ， 就可以将数据集恢复到  FLUSHALL  执行之前的状态。



#### AOF 的缺点

1. 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
2. 根据同步策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒同步的性能依然非常高， 而关闭同步可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。

## 15、哨兵



### 哨兵



之前我们主从架构中，如果主节点挂了，我们要人为干预。现在 Redis 引入了哨兵这个机制，自动化故障恢复。

下图，右面是数据节点，左面是哨兵，它是分布式架构模式，主要就是监控数据节点的主从服务器，自动切换故障节点。

![1702185179583](redis教程.assets/1702185179583.png)

**概念**

监控(Monitor)：哨兵会不断检查主从节点运作正常。

通知（Notification）：如果有问题，那么会通过 api 发送消息出去。

故障自动迁移(Automatic failover)：当一个主服务器故障，哨兵会自动发生故障迁移。

配置提供者（Configuration provider）客户端不需要直接链接到主从节点，可以直接连接到哨兵，因为哨兵提供了一个服务发现的功能，客户端连接到哨兵后，就可以获取到主从节点的信息，如果主节点发生故障，重新产生主节点，哨兵也会把新的信息同步给客户端。



#### 哨兵的分布式好处



可以再一个架构中运行多个哨兵进程。

1. 降低误报。（决策机制，大多数节点认为正确的事情，才是正确的）
2. 减低客户端的影响。（哨兵可以统一的对外提供服务，并不用关心数据节点内部细节）
3. 任意哨兵都可以对外提供服务。
4. 哨兵模式基于主从模式，所有的优点，哨兵都有。
5. 主从自动切换，系统健壮性，高可用性。
6. 哨兵可以一直检查主从服务器是否运行正常。一旦监控到某个 Redis 服务器有问题，哨兵就通过 API 向管理员发送通知。



#### 哨兵的不足



1. 哨兵的配置是允许丢失有限的写入。这点要评估一下是否符合你的系统要求。
2. 主从切换需要时间，会丢失数据。（集群中，分片机制解决）
3. 主节点写入的压力也存在。存储能力受到单机的限制。（受单机限制）
4. 动态扩容困难，对于集群，容量达到上限时在线扩容会变得复杂。





### 哨兵工作原理



哨兵内部有 3 个定时任务

1. 每秒每个哨兵对其他哨兵和 Redis 节点执行 PING 命令。
2. 每 2 秒每个哨兵通过 Master 节点的 channel 交换消息。（Pub/Sub 机制）
3. 每 10 秒每个哨兵会对主节点（Master）和从节点（Slave）执行 Info 命令。



#### 如何判断下线如何判断下线



主观下线：某个单独的哨兵实例对服务器做出下线的判断，认为某个服务下线了（接收不到返回结果，网络超时等等）。依据配置

```shell
sentinel down-after-milliseconds mymaster 10000
```

然后让其他的哨兵去判断，如果也是无法返回结果，就进行后续流程，进行仲裁，我们配置的仲裁结果是 2，也就是说 3 个哨兵中有 2 个哨兵觉得不通，那么就不通了。

客观下线：多个哨兵实例对同一个服务器做出下线判断，并且通过命令交流以后，得出下线的判断，然后开启故障转移(failover)。

仲裁：就是少数服从多数。配置文件中 quorum 配置项，默认是哨兵个数的 1/2+1,我们的案例中是 3 个哨兵，所以 3/2+1=2



#### 故障迁移一致性



哨兵的分布式一致性算法使用 raft 共识算法。

1. 哨兵自动故障迁移使用 Raft 算法选出 leader，确保一个给定的周期（epoch）中，只有一个 leader 产生。
2. 在同一个周期中，不会有 2 个哨兵同时被选中为 leader，并且每个哨兵在同一个节点中只会对一个 Leader 进行投票。
3. 更高的配置节点总是由于较低的节点，因此每个哨兵都会主动使用更新的节点来代替自己的配置。



## 16、集群与分片





### 集群分片



![1702137384715](redis教程.assets/1702137384715.png)

每个节点都可以保存数据，数据是保存在主节点的，每个从节点只是主节点的一个复制，容灾和复制的作用。

为了集群的高可用，所以在每一个主节点后都配置了一个从节点，这样就当某个主节点挂了，从节点可用顶替主节点，继续提供服务，不至于整个集群挂了。

M1，M2，M3 — 主节点 1，主节点 2，主节点 3

S1，S2，S3 — 从节点 1，从节点 2，从节点 3

集群要是多个节点，官方建议，至少 3 主 3 从，主节点会被分配插槽来存储数据，每个 key 都会进行 hash 计算，找到一个插槽的索引进行存储，一共是 16384 个插槽。

Redis-cluster 集群通过 Gossip 协议进行信息广播。



优点

```
1. 可扩展性（数据按插槽分配在多个节点上的）
2. 高可用性（部分节点不可用，可用通过Gossip协议，然后投票，让从节点变成主节点顶上去，保证集群的可用性）
3. 自动故障转义
4. 去中性化
```

缺点

```
1. 异步复制数据，无法保证数据实时强一致性（保证数据最终一致性）
2. 集群搭建有成本。
```







### 数据如何分区



redis-cluster 集群采用分布式存储。

好处：

1. 可用性高（一个节点挂了，其他节点的数据还可以用）
2. 容易维护（一个节点挂了，只需要处理该节点就可以）
3. 均衡 I/O（不同的请求，映射到不同的节点访问，改善整个系统性能）
4. 性能高（只关心自己关心的节点，提高搜索速度）



#### 分区算法



1. 范围分区 ：同一个范围的范围查询不需要跨节点，提升查询速度。比如 MySQL,Oracle.

2. 节点取余分区：hash(key) %N。实现简单，但是当扩容或缩容的时候，需要迁移的数据量大。

3. 一致性哈希分区

   ![1702180107663](redis教程.assets/1702180107663.png)

   **加入和删除节点只影响哈希环中相邻的节点，对其他节点无影响。但是当使用少量节点的时候，节点变化将大范围影响哈希环中数据映射，因此这种方式不适合少量数据节点的分布式方案。**

4. 虚拟槽分区

   每个节点均匀的分配了槽，减少了扩容和缩容节点影响的范围。但是需要存储节点和槽的对应信息。

   ![1702180304669](redis教程.assets/1702180304669.png)





### 集群环境搭建





docker-compose.yml

```shell
version: "3.0"
services:
  redis-1:
    image: redis:6.2
    container_name: redis-1
    ports:
      - "6379:6379"
      - "16379:16379" # Cluster bus port The default is redis Port plus 1000, Each node should be opened
    volumes:
      - $PWD/redis-1/redis.conf:/redis.conf
      - $PWD/redis-1/data:/data
    command: redis-server /redis.conf
  redis-2:
    image: redis:6.2
    container_name: redis-2
    ports:
      - "6380:6380"
      - "16380:16380" # Cluster bus port The default is redis Port plus 1000, Each node should be opened
    volumes:
      - $PWD/redis-2/redis.conf:/redis.conf
      - $PWD/redis-2/data:/data
    command: redis-server /redis.conf
  redis-3:
    image: redis:6.2
    container_name: redis-3
    ports:
      - "6381:6381"
      - "16381:16381" # Cluster bus port The default is redis Port plus 1000, Each node should be opened
    volumes:
      - $PWD/redis-3/redis.conf:/redis.conf
      - $PWD/redis-3/data:/data
    command: redis-server /redis.conf
  redis-4:
    image: redis:6.2
    container_name: redis-4
    ports:
      - "6382:6382"
      - "16382:16382" # Cluster bus port The default is redis Port plus 1000, Each node should be opened
    volumes:
      - $PWD/redis-4/redis.conf:/redis.conf
      - $PWD/redis-4/data:/data
    command: redis-server /redis.conf
  redis-5:
    image: redis:6.2
    container_name: redis-5
    ports:
      - "6383:6383"
      - "16383:16383" # Cluster bus port The default is redis Port plus 1000, Each node should be opened
    volumes:
      - $PWD/redis-5/redis.conf:/redis.conf
      - $PWD/redis-5/data:/data
    command: redis-server /redis.conf
  redis-6:
    image: redis:6.2
    container_name: redis-6
    ports:
      - "6384:6384"
      - "16384:16384"
    volumes:
      - $PWD/redis-6/redis.conf:/redis.conf
      - $PWD/redis-6/data:/data
    command: redis-server /redis.conf

```



Redis.conf

```shell
bind 0.0.0.0
port 端口
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
cluster-announce-ip 你自己的ip
```

配置 Redis-cluster 集群

```shell
redis-cli --cluster create --cluster-replicas 1 192.168.0.100:6379 192.168.0.100:6380 192.168.0.100:6381 192.168.0.100:6382 192.168.0.100:6383 192.168.0.100:6384
```

如下图：

![1702180946375](redis教程.assets/1702180946375.png)

![1702180999759](redis教程.assets/1702180999759.png)

主节点：6379 槽：0-5460

主节点：6380 槽：5461-10922

主节点：6381 槽：10923-16383





### 单节点与集群性能测试



性能测试一定要参考硬件，否则就是**耍流氓**。为什么这么说呢，比如一个 8 核 16G 的服务器和 1 核 1G 的服务器分别做性能测试，一定是相同配置的服务器，跑不同的测试用例。

|      |                           |                                            |           |
| ---- | ------------------------- | ------------------------------------------ | --------- |
| 序号 | 选项                      | 描述                                       | 默认值    |
| 1    | **-h**                    | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**                    | 指定服务器端口                             | 6379      |
| 3    | **-s**                    | 指定服务器 socket                          |           |
| 4    | **-c**                    | 指定并发连接数                             | 50        |
| 5    | **-n**                    | 指定请求数                                 | 10000     |
| 6    | **-d**                    | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| 7    | **-k**                    | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**                    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**                    | 通过管道传输 请求                          | 1         |
| 10   | **-q**                    | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **–csv**                  | 以 CSV 格式输出                            |           |
| 12   | ***-l\*（L 的小写字母）** | 生成循环，永久执行测试                     |           |
| 13   | **-t**                    | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | ***-I\*（i 的大写字母）** | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

**单机测试**

启动一个单节点。

```shell
docker exec -it my-redis /bin/bash

cd /usr/local/bin  #进入bin目录

redis-benchmark -h 127.0.0.1 -p 6379 -t set,get -r 1000000 -n 100000 -c 1000

====== SET ======
  100000 requests completed in 1.45 seconds
  1000 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 3600 1 300 100 60 10000
  host configuration "appendonly": yes
  multi-thread: no

Summary:
  throughput summary: 69108.50 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
       10.145     3.712     9.999    13.903    18.255    30.447


====== GET ======
  100000 requests completed in 0.98 seconds
  1000 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 3600 1 300 100 60 10000
  host configuration "appendonly": yes
  multi-thread: no


Summary:
  throughput summary: 101626.02 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        4.947     2.400     4.903     6.031     6.503     8.399
```



**集群测试**

```shell
进入redis-1容器
docker exec -it redis-1 /bin/baah

创建集群
redis-cli --cluster create --cluster-replicas 1 192.168.0.101:6379 192.168.0.101:6380 192.168.0.101:6381 192.168.0.101:6382 192.168.0.101:6383 192.168.0.101:6384

开始测试
redis-benchmark --cluster -h 192.168.0.101 -p 6379 -t set,get -r 1000000 -n 1000000 -c 1000
------------------------------------------------------------------------

Cluster has 3 master nodes:

Master 0: a8e635a0b08c44af412bda2aa9a84a66e58e830d 192.168.0.101:6380
Master 1: 2106cc10e6b285d55e511194c9456bd066664564 192.168.0.101:6381
Master 2: bddf5e622812b6f10be30a21833eb4be91a63cda 192.168.0.101:6379

====== SET ======
  1000000 requests completed in 53.49 seconds
  1000 parallel clients
  3 bytes payload
  keep alive: 1
  cluster mode: yes (3 masters)
  node [0] configuration:
    save: 3600 1 300 100 60 10000
    appendonly: no
  node [1] configuration:
    save: 3600 1 300 100 60 10000
    appendonly: no
  node [2] configuration:
    save: 3600 1 300 100 60 10000
    appendonly: no
  multi-thread: yes
  threads: 3

  Summary:
  throughput summary: 18696.48 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
       51.964     4.728    50.143    70.079    80.703  1341.439
====== GET ======
  1000000 requests completed in 53.64 seconds
  1000 parallel clients
  3 bytes payload
  keep alive: 1
  cluster mode: yes (3 masters)
  node [0] configuration:
    save: 3600 1 300 100 60 10000
    appendonly: no
  node [1] configuration:
    save: 3600 1 300 100 60 10000
    appendonly: no
  node [2] configuration:
    save: 3600 1 300 100 60 10000
    appendonly: no
  multi-thread: yes
  threads: 3

  Summary:
  throughput summary: 18643.50 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
       51.083     8.928    49.215    71.615    83.583  1342.463
```

**单节点统计**

Summary:
throughput summary: 101626.02 requests per second

**集群统计**

Summary:
throughput summary: 18643.50 requests per second

单点要比集群性能更好，大家可以思考一下，为什么?





### Redis 集群原理



redis 没有采用一致性（哈希环），如果添加和删除 Node 的时候，只影响临近的 Node,但是 Node 比较少的时候，涉及到数据分担不均衡，并且添加和删除数据也是影响范围很大。节点越多（几千或上万），影响范围越小。

**采用了哈希槽**

1. 哈希算法,不是简单的 hash 算法，是 crc16 算法，是一种校验算法。
2. 槽位的空间分配。(类似 windows 分区，可用分配 C，D，E 盘，并且每个盘都可以自定义大小) 对于槽位的转移和分配，Redis 是人工配置的，不会自动进行。所以 Redis 集群的高可用是依赖于节点的主从复制与主从间的自动故障转移。槽位共 16384 个。集群中每个节点只负责一部分槽位，槽位的信息存储于每个节点中。

槽位计算公式

```shell
slot = CRC16(key) mod 16384
```

为什么只有 16384 个槽？

crc16()方法有 2 的 16 次方-1，也就是 65535，也就是 0-65535，共 65536 个槽。那为什么不是对 65536 取余，而是用 16384 取余。

1. 如果 65536 个槽，那么发送心跳包的 header 会是 8k，那么心跳包每秒都要发送，发送数据过大，太占带宽。
2. redis 集群不会超过 1000 个节点。 没有公司能达到这种数据量。如果节点都不会超过 1000 个，那么就没有必要用到 65536 个槽。
3. 压缩率高。（16384 个槽和 65536 个槽比，槽位少的，压缩率自然就高了）





### 添加主节点



添加一个节点，需要从其他节点的插槽拿过来分配。

1. 新建 redis-7 目录

2. 拷贝 redis.conf，并修改端口为 6385

3. 修改 docker-compose.yml 增加 redis-7 节点

4. docker-compose up

   ![1702182854184](redis教程.assets/1702182854184.png)

5. 进入 redis-1

   ```shell
   docker exec -it redis-1 /bin/bash
   ```

6. 查看集群信息

   ```shell
   redis-cli cluster info
   
   cluster_state:ok
   cluster_slots_assigned:16384
   cluster_slots_ok:16384
   cluster_slots_pfail:0
   cluster_slots_fail:0
   cluster_known_nodes:6
   cluster_size:3
   cluster_current_epoch:6
   cluster_my_epoch:1
   cluster_stats_messages_ping_sent:346
   cluster_stats_messages_pong_sent:373
   cluster_stats_messages_sent:719
   cluster_stats_messages_ping_received:373
   cluster_stats_messages_pong_received:346
   cluster_stats_messages_received:719
   ```

7. 查看集群节点信息

   ```shell
   redis-cli cluster nodes
   
   09d006ff1c193057e5b4969a3854dfa85929f382 192.168.0.101:6382@16382 slave 2106cc10e6b285d55e511194c9456bd066664564 0 1661952829000 3 connected
   bf2a9306c2ebc01405d84d1ec8d6d85ffdcdef83 192.168.0.101:6384@16384 slave a8e635a0b08c44af412bda2aa9a84a66e58e830d 0 1661952829000 2 connected
   d6033f945a5d8391fea832ef1ed98ec018062d35 192.168.0.101:6383@16383 slave bddf5e622812b6f10be30a21833eb4be91a63cda 0 1661952829000 1 connected
   a8e635a0b08c44af412bda2aa9a84a66e58e830d 192.168.0.101:6380@16380 master - 0 1661952829926 2 connected 5461-10922
   2106cc10e6b285d55e511194c9456bd066664564 192.168.0.101:6381@16381 master - 0 1661952830934 3 connected 10923-16383
   bddf5e622812b6f10be30a21833eb4be91a63cda 192.168.0.101:6379@16379 myself,master - 0 1661952827000 1 connected 0-5460
   ```

8. 添加主节点，语法如下

   ```shell
   redis-cli --cluster add-node new_host:new_port 最后节点的Host:最后节点的Port --cluster-master-id 最后节点的Id
   
   redis-cli --cluster add-node 192.168.0.101:6385 192.168.0.101:6381 --cluster-master-id 2106cc10e6b285d55e511194c9456bd066664564
   ```

   

   ![1702183006274](redis教程.assets/1702183006274.png)

   ![1702183063230](redis教程.assets/1702183063230.png)

节点添加完了，但是没有分片，也就是没有槽，那就要从其他节点找到一些槽，分配到这个节点上。

```shell
redis-cli --cluster reshard host:port --cluster-from node_id --cluster-to node_id --cluster-slots 需要分配的槽个数 --cluster-yes

redis-cli --cluster reshard 172.18.0.7:6379 --cluster-from a4b3e461d95d09eb1e991d8ac910b9456db64af6 --cluster-to ac7673c66c50b547beee9df3d0781d60573fb701 --cluster-slots 2000 --cluster-yes
```

1702183252630

再次查看节点信息,槽位是否分片成功？

```shell
redis-cli cluster nodes
```

![1702183289252](redis教程.assets/1702183289252.png)

可以看到 6385 节点上 0-1999 个槽。





### 添加从节点并组成主从结构



格式如下

```shell
redis-cli --cluster add-node new_host:new_port 给要添加的主节点的Host:给要添加的主节点的Port --cluster-slave --cluster-master-id 给要添加的主节点的Id

redis-cli --cluster add-node 192.168.0.101:6386 192.168.0.101:6385 --cluster-slave --cluster-master-id ac7673c66c50b547beee9df3d0781d60573fb701
```



![1702183896286](redis教程.assets/1702183896286.png)

如果是从节点，直接删掉，如果是主节点，要把槽放到其他节点上，再删除，保证数据安全。





### 删除从节点



1. 从 docker-compose.yml 节点中移除。
2. 移除对应的目录文件

```shell
redis-cli -a 123456 --cluster del-node 要删除节点的Host 要删除节点的Port 要删除节点的ID
```





### 删除主节点



直接删除主节点，要有数据丢失的风险。

1. 移除 docker-compose.yml 节点。

2. 重新分片,把节点的槽分配到其他节点上，否则会丢数据。

   ```shell
   redis-cli --cluster reshard host:port --cluster-from node_id --cluster-to node_id --cluster-slots 需要分配的槽个数 --cluster-yes
   ```

3. 执行删除从节点的命令

   ```shell
   redis-cli --cluster del-node 要删除节点的Host 要删除节点的Port 要删除节点的ID
   ```

   



### 自动故障转移



找到 1 主(redis-3)和 1 从（redis-4），我们把 redis-3 停掉,然后看看 redis-4 是否可以变成主节点

```shell
docker stop redis-3
```

查看 redis-4 日志

```shell
docker logs redis-4
-------------------------------------------------
1:C 01 Sep 2022 01:19:25.330 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 01 Sep 2022 01:19:25.330 # Redis version=6.2.7, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 01 Sep 2022 01:19:25.330 # Configuration loaded
1:M 01 Sep 2022 01:19:25.331 * monotonic clock: POSIX clock_gettime
1:M 01 Sep 2022 01:19:25.342 * Node configuration loaded, I'm 09d006ff1c193057e5b4969a3854dfa85929f382
1:M 01 Sep 2022 01:19:25.342 # A key '__redis__compare_helper' was added to Lua globals which is not on the globals allow list nor listed on the deny list.
1:M 01 Sep 2022 01:19:25.342 * Running mode=cluster, port=6382.
1:M 01 Sep 2022 01:19:25.342 # Server initialized
1:M 01 Sep 2022 01:19:25.352 * Loading RDB produced by version 6.2.7
1:M 01 Sep 2022 01:19:25.352 * RDB age 10 seconds
1:M 01 Sep 2022 01:19:25.352 * RDB memory usage when created 31.16 Mb
1:M 01 Sep 2022 01:19:26.177 # Done loading RDB, keys loaded: 284061, keys expired: 0.
1:M 01 Sep 2022 01:19:26.177 * DB loaded from disk: 0.833 seconds
1:M 01 Sep 2022 01:19:26.177 * Before turning into a replica, using my own master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
1:M 01 Sep 2022 01:19:26.177 * Ready to accept connections
1:S 01 Sep 2022 01:19:26.178 * Discarding previously cached master state.
1:S 01 Sep 2022 01:19:26.178 * Before turning into a replica, using my own master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
1:S 01 Sep 2022 01:19:26.178 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 01:19:26.178 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 01:19:26.178 # Cluster state changed: ok
1:S 01 Sep 2022 01:19:26.179 * Non blocking connect for SYNC fired the event.
1:S 01 Sep 2022 01:19:26.184 * Master replied to PING, replication can continue...
1:S 01 Sep 2022 01:19:26.187 * Trying a partial resynchronization (request 3fd4e6de6eef88188b18361afbe8ed6a36b28437:8555).
1:S 01 Sep 2022 01:19:26.191 * Full resync from master: aa669c5247747f4721ab0f45e1ce8d76101f0748:0
1:S 01 Sep 2022 01:19:26.191 * Discarding previously cached master state.
1:S 01 Sep 2022 01:19:29.393 * MASTER <-> REPLICA sync: receiving 7953892 bytes from master to disk
1:S 01 Sep 2022 01:19:31.334 * MASTER <-> REPLICA sync: Flushing old data
1:S 01 Sep 2022 01:19:31.386 * MASTER <-> REPLICA sync: Loading DB in memory
1:S 01 Sep 2022 01:19:31.395 * Loading RDB produced by version 6.2.7
1:S 01 Sep 2022 01:19:31.395 * RDB age 5 seconds
1:S 01 Sep 2022 01:19:31.395 * RDB memory usage when created 32.26 Mb
1:S 01 Sep 2022 01:19:31.716 # Done loading RDB, keys loaded: 284061, keys expired: 0.
1:S 01 Sep 2022 01:19:31.716 * MASTER <-> REPLICA sync: Finished with success
1:S 01 Sep 2022 03:05:31.415 * 1 changes in 3600 seconds. Saving...
1:S 01 Sep 2022 03:05:31.417 * Background saving started by pid 23
23:C 01 Sep 2022 03:05:33.797 * DB saved on disk
23:C 01 Sep 2022 03:05:33.797 * RDB: 0 MB of memory used by copy-on-write
1:S 01 Sep 2022 03:05:33.827 * Background saving terminated with success
1:S 01 Sep 2022 03:09:28.056 # Connection with master lost.
1:S 01 Sep 2022 03:09:28.056 * Caching the disconnected master state.
1:S 01 Sep 2022 03:09:28.056 * Reconnecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:28.056 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:28.057 * Non blocking connect for SYNC fired the event.
1:S 01 Sep 2022 03:09:28.058 # Error reply to PING from master: '-Reading from master: Operation now in progress'
1:S 01 Sep 2022 03:09:29.004 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:29.005 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:29.005 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:30.015 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:30.015 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:30.016 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:31.023 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:31.023 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:31.024 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:32.032 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:32.032 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:32.032 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:33.043 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:33.043 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:33.044 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:34.052 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:34.052 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:34.053 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:35.063 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:35.063 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:35.064 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:36.073 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:36.073 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:36.074 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:37.082 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:37.082 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:37.083 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:38.092 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:38.092 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:38.093 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:39.106 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:39.106 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:39.108 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:40.119 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:40.119 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:40.120 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:41.135 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:41.135 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:41.136 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:42.152 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:42.152 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:42.153 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:43.164 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:43.165 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:43.166 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:44.176 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:44.176 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:44.177 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:45.203 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:45.203 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:45.204 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:46.212 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:46.212 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:46.212 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:47.024 * FAIL message received from bf2a9306c2ebc01405d84d1ec8d6d85ffdcdef83 about 2106cc10e6b285d55e511194c9456bd066664564
1:S 01 Sep 2022 03:09:47.024 # Cluster state changed: fail
1:S 01 Sep 2022 03:09:47.125 # Start of election delayed for 961 milliseconds (rank #0, offset 9292).
1:S 01 Sep 2022 03:09:47.226 * Connecting to MASTER 192.168.0.101:6381
1:S 01 Sep 2022 03:09:47.226 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:09:47.226 # Error condition on socket for SYNC: Connection refused
1:S 01 Sep 2022 03:09:48.133 # Starting a failover election for epoch 8.
1:S 01 Sep 2022 03:09:48.147 # Failover election won: I'm the new master.
1:S 01 Sep 2022 03:09:48.147 # configEpoch set to 8 after successful failover
1:M 01 Sep 2022 03:09:48.147 * Discarding previously cached master state.
1:M 01 Sep 2022 03:09:48.147 # Setting secondary replication ID to aa669c5247747f4721ab0f45e1ce8d76101f0748, valid up to offset: 9293. New replication ID is 5eca7aa6bca9534e75e85bff2aca7211113eb1fb
1:M 01 Sep 2022 03:09:48.148 # Cluster state changed: ok
```

上述表示 Cluster state changed: ok 集群已经重新搭建完成。

再次查看集群状态，6381 是 fail

![1702184579323](redis教程.assets/1702184579323.png)

现在我们重新启动 6381

```shell
docker logs redis-3
------------------------------------------------------------------------------
1:C 01 Sep 2022 01:19:25.066 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 01 Sep 2022 01:19:25.066 # Redis version=6.2.7, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 01 Sep 2022 01:19:25.066 # Configuration loaded
1:M 01 Sep 2022 01:19:25.067 * monotonic clock: POSIX clock_gettime
1:M 01 Sep 2022 01:19:25.071 * Node configuration loaded, I'm 2106cc10e6b285d55e511194c9456bd066664564
1:M 01 Sep 2022 01:19:25.071 # A key '__redis__compare_helper' was added to Lua globals which is not on the globals allow list nor listed on the deny list.
1:M 01 Sep 2022 01:19:25.071 * Running mode=cluster, port=6381.
1:M 01 Sep 2022 01:19:25.071 # Server initialized
1:M 01 Sep 2022 01:19:25.075 * Loading RDB produced by version 6.2.7
1:M 01 Sep 2022 01:19:25.075 * RDB age 10 seconds
1:M 01 Sep 2022 01:19:25.075 * RDB memory usage when created 31.14 Mb
1:M 01 Sep 2022 01:19:25.566 # Done loading RDB, keys loaded: 284061, keys expired: 0.
1:M 01 Sep 2022 01:19:25.566 * DB loaded from disk: 0.495 seconds
1:M 01 Sep 2022 01:19:25.567 * Ready to accept connections
1:M 01 Sep 2022 01:19:26.189 * Replica 192.168.16.1:6382 asks for synchronization
1:M 01 Sep 2022 01:19:26.189 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for '3fd4e6de6eef88188b18361afbe8ed6a36b28437', my replication IDs are '16089dc0096b9bb777ed1fa6fe1040e19f934812' and '0000000000000000000000000000000000000000')
1:M 01 Sep 2022 01:19:26.189 * Replication backlog created, my new replication IDs are 'aa669c5247747f4721ab0f45e1ce8d76101f0748' and '0000000000000000000000000000000000000000'
1:M 01 Sep 2022 01:19:26.189 * Starting BGSAVE for SYNC with target: disk
1:M 01 Sep 2022 01:19:26.190 * Background saving started by pid 23
1:M 01 Sep 2022 01:19:27.580 # Cluster state changed: ok
23:C 01 Sep 2022 01:19:29.300 * DB saved on disk
23:C 01 Sep 2022 01:19:29.301 * RDB: 0 MB of memory used by copy-on-write
1:M 01 Sep 2022 01:19:29.389 * Background saving terminated with success
1:M 01 Sep 2022 01:19:30.089 * Synchronization with replica 192.168.16.1:6382 succeeded
1:M 01 Sep 2022 03:05:31.415 * 1 changes in 3600 seconds. Saving...
1:M 01 Sep 2022 03:05:31.416 * Background saving started by pid 24
24:C 01 Sep 2022 03:05:33.804 * DB saved on disk
24:C 01 Sep 2022 03:05:33.804 * RDB: 0 MB of memory used by copy-on-write
1:M 01 Sep 2022 03:05:33.827 * Background saving terminated with success
1:signal-handler (1662001765) Received SIGTERM scheduling shutdown...
1:M 01 Sep 2022 03:09:25.990 # User requested shutdown...
1:M 01 Sep 2022 03:09:25.990 * Saving the final RDB snapshot before exiting.
1:M 01 Sep 2022 03:09:28.046 * DB saved on disk
1:M 01 Sep 2022 03:09:28.046 # Redis is now ready to exit, bye bye...
1:C 01 Sep 2022 03:18:18.736 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 01 Sep 2022 03:18:18.736 # Redis version=6.2.7, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 01 Sep 2022 03:18:18.736 # Configuration loaded
1:M 01 Sep 2022 03:18:18.736 * monotonic clock: POSIX clock_gettime
1:M 01 Sep 2022 03:18:18.740 * Node configuration loaded, I'm 2106cc10e6b285d55e511194c9456bd066664564
1:M 01 Sep 2022 03:18:18.740 # A key '__redis__compare_helper' was added to Lua globals which is not on the globals allow list nor listed on the deny list.
1:M 01 Sep 2022 03:18:18.740 * Running mode=cluster, port=6381.
1:M 01 Sep 2022 03:18:18.740 # Server initialized
1:M 01 Sep 2022 03:18:18.745 * Loading RDB produced by version 6.2.7
1:M 01 Sep 2022 03:18:18.745 * RDB age 533 seconds
1:M 01 Sep 2022 03:18:18.745 * RDB memory usage when created 32.28 Mb
1:M 01 Sep 2022 03:18:19.181 # Done loading RDB, keys loaded: 284065, keys expired: 0.
1:M 01 Sep 2022 03:18:19.181 * DB loaded from disk: 0.440 seconds
1:M 01 Sep 2022 03:18:19.181 * Ready to accept connections
1:M 01 Sep 2022 03:18:19.183 # Configuration change detected. Reconfiguring myself as a replica of 09d006ff1c193057e5b4969a3854dfa85929f382
1:S 01 Sep 2022 03:18:19.183 * Before turning into a replica, using my own master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
1:S 01 Sep 2022 03:18:19.183 * Connecting to MASTER 192.168.0.101:6382
1:S 01 Sep 2022 03:18:19.184 * MASTER <-> REPLICA sync started
1:S 01 Sep 2022 03:18:19.185 # Cluster state changed: ok
1:S 01 Sep 2022 03:18:19.190 * Non blocking connect for SYNC fired the event.
1:S 01 Sep 2022 03:18:19.203 * Master replied to PING, replication can continue...
1:S 01 Sep 2022 03:18:19.205 * Trying a partial resynchronization (request 23b6898179e974baea9642acefdef8aecbc4f201:1).
1:S 01 Sep 2022 03:18:19.207 * Full resync from master: 5eca7aa6bca9534e75e85bff2aca7211113eb1fb:9292
1:S 01 Sep 2022 03:18:19.207 * Discarding previously cached master state.
1:S 01 Sep 2022 03:18:21.477 * MASTER <-> REPLICA sync: receiving 7953931 bytes from master to disk
1:S 01 Sep 2022 03:18:22.442 * MASTER <-> REPLICA sync: Flushing old data
1:S 01 Sep 2022 03:18:22.493 * MASTER <-> REPLICA sync: Loading DB in memory
1:S 01 Sep 2022 03:18:22.503 * Loading RDB produced by version 6.2.7
1:S 01 Sep 2022 03:18:22.503 * RDB age 3 seconds
1:S 01 Sep 2022 03:18:22.503 * RDB memory usage when created 32.30 Mb
1:S 01 Sep 2022 03:18:22.754 # Done loading RDB, keys loaded: 284065, keys expired: 0.
1:S 01 Sep 2022 03:18:22.754 * MASTER <-> REPLICA sync: Finished with success
```

再次查看集群状态，6831 已变成从节点

![1702184628100](redis教程.assets/1702184628100.png)

小伙伴也可以自己动手试试查询一下数据。





### 手动故障转移



之前我们对集群中的主从节点切换，操作步骤多，容易出错，下面看一下方便的做法.

在从节点上执行

```shell
redis-cli -c -h x.x.x.x -p 端口 cluster failover
```

这样当前的从节点就会升级为主节点，原来主节点就会降级为从节点。非常方便。小伙伴们可以自行查看对应的日志。