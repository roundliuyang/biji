# Zookeeper 极简入门



## 1. 概述

ZooKeeper 是一个开放源码的分布式协调服务，它是集群的管理者，监视着集群中各个节点的状态根据节点提交的反馈进行下一步合理操作。最终，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

分布式应用程序可以基于 Zookeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

Zookeeper 具有如下特性：

**① 顺序一致性(有序性)**

从同一个客户端发起的事务请求，最终将会严格地按照其发起顺序被应用到 Zookeeper 中去。

有序性是 Zookeeper 中非常重要的一个特性。

- 所有的更新都是全局有序的，每个更新都有一个唯一的时间戳，这个时间戳称为 zxid(Zookeeper Transaction Id)。
- 而读请求只会相对于更新有序，也就是读请求的返回结果中会带有这个 Zookeeper 最新的 zxid 。



**② 原子性**

所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，即整个集群要么都成功应用了某个事务，要么都没有应用。



**③ 单一视图**

无论客户端连接的是哪个 Zookeeper 服务器，其看到的服务端数据模型都是一致的。



**④ 可靠性**

一旦服务端成功地应用了一个事务，并完成对客户端的响应，那么该事务所引起的服务端状态变更将会一直被保留，除非有另一个事务对其进行了变更。



**⑤ 实时性**

Zookeeper 保证在一定的时间段内，客户端最终一定能够从服务端上读取到最新的数据状态。



## 2. 单机部署

> 操作系统：macOS 10.14
>
> 其它系统，基本一致的。

注意，需要安装 JDK 1.6+ 版本，因为 Zookeeper 是使用 Java 语言开发的。



### 2.1 下载

打开 [Zookeeper 下载页面](https://archive.apache.org/dist/zookeeper/)，选择想要的 Zookeeper 版本。这里，我们选择 [stable](https://archive.apache.org/dist/zookeeper/stable/) 稳定版本。

```shell
# 创建目录
$ mkdir -p /Users/yunai/Zookeeper
$ cd /Users/yunai/Zookeeper

# 下载
$ wget https://archive.apache.org/dist/zookeeper/stable/apache-zookeeper-3.5.6-bin.tar.gz

# 解压
$ tar -zxvf apache-zookeeper-3.5.6-bin.tar.gz
$ cd apache-zookeeper-3.5.6-bin

# 查看目录
$ ls- ls
24 -rw-r--r--   1 yunai  staff  11358 Oct  5 19:27 LICENSE.txt
 8 -rw-r--r--   1 yunai  staff    432 Oct  9 04:14 NOTICE.txt
 8 -rw-r--r--   1 yunai  staff   1560 Oct  9 04:14 README.md
 8 -rw-r--r--   1 yunai  staff   1347 Oct  5 19:27 README_packaging.txt
 0 drwxr-xr-x  13 yunai  staff    416 Oct  9 04:14 bin # 执行脚本
 0 drwxr-xr-x   5 yunai  staff    160 Dec  5 13:13 conf # 配置文件
 0 drwxr-xr-x  21 yunai  staff    672 Oct  9 04:15 docs # 文档
 0 drwxr-xr-x  41 yunai  staff   1312 Dec  5 13:11 lib # Zookeeper jar 包
```



### 2.2 配置文件

Zookeeper 提供了 `conf/zoo_sample.cfg` 配置文件，作为示例。这里，我们使用 `cp` 命令，复制出一个 `conf/zoo.cfg` 配置文件，然后在上面进行修改。默认配置如下：

```shell
# The number of milliseconds of each tick
# Client-Server 通信心跳时间
# Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime 以毫秒为单位。
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
# Leader-Follower 初始通信时限
# 集群中的 follower 服务器(F)与 leader 服务器(L)之间初始连接时能容忍的最多心跳数（tickTime 的数量）。
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
# Leader-Follower 同步通信时限
# 集群中的 follower 服务器与 leader 服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
# 数据文件目录
# Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
# 注意，生产环境下不要放在 /tmp 目录下，这里仅仅是示例。
dataDir=/tmp/zookeeper
# the port at which the clients will connect
# 客户端连接端口
# 客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

这里，我们创建一个 `data` 目录，将配置文件修改为 `dataDir=/Users/yunai/Zookeeper/apache-zookeeper-3.5.6-bin/data` 。



### 2.3 运行 Zookeeper Server

执行 `bin/zkServer.sh start` 命令，启动 Zookeeper Server 服务。此时，控制台会输出如下日志，表示启动成功。

```shell
# 默认情况下，Zookeeper 开启 JMX
ZooKeeper JMX enabled by default
# 使用 conf/zoo.cfg 配置文件
Using config: /Users/yunai/Zookeeper/apache-zookeeper-3.5.6-bin/bin/../conf/zoo.cfg
# 启动 Zookeeper Server 成功（实际不一定成功）
Starting zookeeper ... STARTED

```

注意，Zookeeper 3.5 版本开始，默认会在 8080 端口，启动一个 Zookeeper **AdminServer**。如果胖友 8080 端口已经被其它服务占用，会导致 Zookeeper Server 启动失败。此时，我们有三种解决方案：

- 方式一，可以修改 `conf/zoo.cfg` 配置文件的 `admin.serverPort` 配置项，从而修改 Zookeeper AdminServer 的端口。
- 方式二，可以修改 `conf/zoo.cfg` 配置文件的 `admin.enableServer=false` 配置项，从而关闭 Zookeeper AdminServer 的启动。
- 方式三，关闭占用 8080 端口的服务，嘿嘿嘿。





### 2.4 测试连接

连接到 Zookeeper Server 上，看看是否真的启动成功。操作命令如下：

```shell
# 连接 Zookeeper Server
$ bin/zkCli.sh
Welcome to ZooKeeper!
JLine support is enabled

# 在 Zookeeper 命令行中，执行 ls / 命令，输出根目录
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]
```

- 执行成功，说明 Zookeeper Server 已经启动完成了。

如果胖友发现 Zookeeper Server 并未启动成功，可以查看 `logs/` 目录下的日志文件，是否存在错误日志。



## 3. 集群部署

> 艿艿：这里先偷懒，直接引用 [《Zookeeper 安装及配置（Mac）》](https://www.jianshu.com/p/0ba61bf7149f) 文章，略作修改。
>
> 生产环境下，Zookeeper 一定要搭建集群噢，并且节点个数是奇数个。

集群模式有两种形式：

- 1）使用多台机器，在每台机器上运行一个ZooKeeper Server进程；
- 2）使用一台机器，在该台机器上运行多个ZooKeeper Server进程。

在生产环境中，一般使用第一种形式，在练习环境中，一般使用第二种形式。



### 3.1 配置文件

集群模式下，需要配置一些参数，以下是常见的一些参数。

- 1、data目录

  > 用于存放进程运行数据。

- 2、data目录下的 myid 文件

  > 用于存储一个数值，用来作为该ZooKeeper Server进程的标识。

- 3、监听 Client 端请求的端口号

- 4、监听同 ZooKeeper 集群内其他 Server 进程通信请求的端口号

- 5、监听 ZooKeeper 集群内“leader”选举请求的端口号

  > 该端口号用来监听 ZooKeeper 集群内“leader”选举的请求。注意这个是 ZooKeeper 集群内“leader”的选举，跟分布式应用程序无关。

**参数配置注意事项：**

- 1）同一个 ZooKeeper 集群内，不同 ZooKeeper Server 进程的标识需要不一样，即 myid 文件内的值需要不一样
- 2）采用上述第 2 种形式构建 ZooKeeper 集群，需要注意“目录，端口号”等资源的不可共享性，如果共享会导致 ZooKeeper Server 进程不能正常运行，比如“data 目录，几个监听端口号”都不能被共享



| myid | Data目录 | Client | Server | Leader | 配置文件 |
| :--- | :------- | :----- | :----- | :----- | :------- |
| 1    | /z1/data | 2181   | 2222   | 2223   | z1.cfg   |
| 2    | /z2/data | 2182   | 3333   | 3334   | z2.cfg   |
| 3    | /z3/data | 2183   | 4444   | 4445   | z3.cfg   |

配置如下：

```shell
# zx.cfg
tickTime=2000
initLimit=10
syncLimit=2
dataDir=/usr/myenv/zookeeper-3.4.8/zx/data
clientPort=218x
# server.x 中的“x”表示 ZooKeeper Server 进程的标识
server.1=127.0.0.1:2222:2225
server.2=127.0.0.1:3333:3335
server.3=127.0.0.1:4444:4445
```

**注：**

- initLimit：Zookeeper 集群中的包含多台 Server ，其中一台为leader，集群中其余的 server 为 follower。initLimit 参数配置初始化连接时, follower 和 leader 之间的最长心跳时间。此时该参数设置为 5，说明时间限制为 5 倍 tickTime, 即`5*2000=10000ms=10s` 。
- syncLimit：该参数配置 leader 和 follower 之间发送消息，请求和应答的最大时间长度。此时该参数设置为 2，说明时间限制为 2 倍tickTime，即 4000ms 。

### 3.2 运行 ZooKeeper Server

分别执行

```shell
bin/zkServer.sh start deploy/z1/z1.cfg，
bin/zkServer.sh start deploy/z2/z2.cfg
bin/zkServer.sh start deploy/z3/z3.cfg
```

运行上述配置的 3 个 ZooKeeper Server 进程。



### 3.3 运行 ZooKeeper 命令行客户端

执行命令

```shell
建立 ZooKeeper Client 端到 ZooKeeper 集群的连接会话。
$ bin/zkCli.sh -server 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
```



## 4. UI 控制台

参考 [《Zookeeper 的 web 管理系统》](https://blog.csdn.net/u013673976/article/details/47306891) 。



## 5. 简单示例

一般情况下，我们并不会直接使用 Zookeeper 提供的 Java 客户端，而是使用 [Apache Curator](https://curator.apache.org/) ，为了简化 Zookeeper 客户端调用而生，利用它可以更好的使用Zookeeper。

具体的示例，可以看看 [curator-examples](https://github.com/apache/curator/tree/master/curator-examples) 项目，提供了 8 个示例，基本能够满足我们日常开发的需要。













