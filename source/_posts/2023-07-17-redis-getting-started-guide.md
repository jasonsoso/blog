---
title: Redis指南：安装、使用和关键点复习
date: 2023/7/17 22:00:00
tags:
    - CentOS
    - 软件
    - Redis
    - 缓存数据库
categories: 备忘录
---

![](http://tech.jasonsoso.com/images/2023/redis.png)

## Redis介绍

1. Redis概述

- Redis（Remote Dictionary Server）是一个开源的键值对存储系统，以内存为主要存储介质，也可以通过持久化将数据保存到磁盘上。
它被设计为一个高性能、低延迟的数据存储解决方案，广泛应用于缓存、会话管理、实时分析、消息传递和排行榜等场景。

- Redis支持多种数据结构，包括字符串、哈希（Hash）、列表（List）、集合（Set）和有序集合（Sorted Set）。
这些数据结构赋予了Redis更灵活的功能，例如存储各种类型的数据、处理计数器、实现发布订阅模式以及进行地理位置查询等。

- 由于Redis将数据存储在内存中，使得其读写速度非常快，通常可以达到每秒几十万次的操作。
此外，Redis还提供了持久化选项，可以将内存中的数据定期快照保存到磁盘上，或者使用AOF（Append-Only File）日志记录每个写操作，从而确保数据的可靠性和持久性。

- 总之，Redis作为一个高性能、多功能的键值存储系统，在各种场景下都展现出了卓越的表现，并受到广泛使用和青睐。



2. Redis的用途和优势（常见应用场景）

- **缓存**：作为内存数据库，Redis被广泛用作缓存层，将经常访问的数据存储在内存中，以提高读取速度和减轻后端数据库负载。
其高性能和低延迟的特点使得它成为理想的缓存解决方案。

- **实时数据处理**：Redis支持发布与订阅模式，使得实时消息传递和事件通知变得简单。
通过发布/订阅功能，可以轻松地构建实时聊天应用、实时推送系统和消息队列等。

- **排行榜和计数器**：Redis的有序集合和自增功能使之成为构建排行榜和计数器的理想工具。
开发人员可以使用有序集合来存储和更新用户的得分信息，并根据分数进行排名和查询。

- **分布式锁**：Redis的原子操作和过期时间设置使得它能够实现分布式环境下的锁机制。
通过Redis的命令和特性，可以实现互斥访问控制和并发控制，确保共享资源的安全性。

- **地理位置查询**：Redis提供了地理位置索引的功能，可以方便地进行附近位置搜索、地理围栏判断和地理数据存储。
这使得Redis在位置服务和地理信息系统等领域具备优势。

- **高性能和可扩展性**：Redis被设计为一个高性能的存储引擎，能够处理大量的读写操作和高并发访问。
它的简单架构和水平扩展能力使得它能够应对大规模数据和流量的挑战。

- **多种数据结构支持**：Redis提供了多种数据结构，如字符串、哈希、列表、集合和有序集合等，使得开发人员可以灵活地应对不同的数据存储需求。

- **共享Session**

- **消息队列系统**


3. 相关下载链接或者命令参考

- 官网地址：https://redis.io/
- 下载地址：https://redis.io/download/
- 安装说明：https://redis.io/docs/getting-started/installation/install-redis-from-source/
- Redis 命令参考：http://redisdoc.com/
- 阿里官方 Redis 开发规范: https://developer.aliyun.com/article/1009125


## 安装和配置

1. 下载，安装并编译Redis

```bash
cd /usr/local/
mkdir redis
cd redis/
# 下载
wget https://download.redis.io/redis-stable.tar.gz
# 解压
tar -xzvf redis-stable.tar.gz
ll
cd redis-stable/
#安装
make install
```




2. 配置Redis
```bash
#拷贝默认redis.conf当做配置
cp redis.conf /etc/redis/redis.conf

```


```bash
#配置文件redis.conf的常见配置

#确保守护进程开启,也就是在后台可以运行.
daemonize yes

#设置密码
requirepass 123456 #授权密码 AUTH 12345

#绑定的主机地址
bind 127.0.0.1 -::1
##监听端口
port 6379

#pid 文件和log文件的保存地址
pidfile /var/run/redis_6379.pid
logfile ""

#指定本地持久化文件的文件名，默认是dump.rdb
dbfilename dump.rdb

```


3. 启动和停止Redis
```bash
#进行服务启动
redis-server
#或者
redis-server /etc/redis/redis.conf

#客户端连接
redis-cli
#或者
redis-cli -h 127.0.0.1 -p 6379
```

![](http://tech.jasonsoso.com/images/2023/redis-server.png)
![](http://tech.jasonsoso.com/images/2023/redis-cli.png)

## Redis基础

1. 数据结构
- 字符串（String）：最基本的数据结构，可以存储字符串、整数、浮点数或者对象缓存存储。它们可以执行诸如获取、设置、增加和减少值等操作。

- 列表（List）：按照插入顺序存储一组有序的元素。可以在列表的两端进行插入和删除操作，还提供了根据索引访问和修剪列表的功能。在 Redis 中可以把 list 用作栈、队列、阻塞队列。

- 哈希（Hash）：存储键值对的无序散列表。适合存储对象或记录，并且可以直接访问和修改特定字段的值。

- 集合（Set）：无序且唯一的元素集合。可以进行添加、删除、求交集、并集和差集等操作。

- 有序集合（Sorted Set）：类似于集合，但每个元素都关联有一个分数（score），用于排序元素。可以按照分数范围或成员值范围进行检索。

- 地理空间（Geospatial）：用于存储地理空间信息的数据结构。支持存储坐标点，并提供距离计算和位置查询功能。

- 位图（bitmap）：bitmap就是通过最小的单位bit来进行0或者1的设置，表示某个元素对应的值或者状态。一个bit的值，或者是0，或者是1；也就是说一个bit能存储的最多信息是2。
bitmap 常用于统计用户信息比如活跃粉丝和不活跃粉丝、登录和未登录、是否打卡等。


2. Redis命令和操作

- Redis 命令参考：http://redisdoc.com/



##  Redis高级特性

1. 发布与订阅模式

2. Lua脚本执行

3. 事务管理

4. 过期时间和缓存策略

5. 分布式锁和并发控制


## Redis高可用性解决方案

Redis的高可用性解决方案主要包括以下几种：

- 主从复制（Master-Slave Replication）：
通过配置Redis实例为主节点（master）和从节点（slave），
将主节点的数据复制到从节点。当主节点发生故障时，可以将其中一个从节点提升为新的主节点，继续提供服务。
主从复制提供了基本的故障恢复和容错能力。

- 哨兵模式（Sentinel）：
哨兵是Redis的监控和自动故障转移解决方案，用于监控Redis实例的状态并自动进行主节点切换。
哨兵集群由多个哨兵节点组成，它们会相互通信以监测Redis实例，并在主节点失效时选举新的主节点。
哨兵模式提供了更高级的故障检测和自动切换功能。

- 集群模式（Cluster）：
Redis集群模式将数据分片存储在多个节点上，每个节点负责管理其中的一部分数据。
集群模式通过分布式方式提供了横向扩展和负载均衡能力，使得整个集群能够处理更大规模的数据和请求量。
集群模式还具备自动故障转移和重新平衡的功能。

- Redis Sentinel与Redis Cluster结合：
为了进一步提高Redis的高可用性，可以将哨兵模式与集群模式结合使用。
在这种配置下，哨兵用于监控和自动故障转移，而Redis集群用于实现数据分片和负载均衡。
这种组合方案结合了两种解决方案的优点，提供了更高级别的高可用性和扩展性。

需要注意的是，无论采用哪种高可用性解决方案，都需要进行适当的配置和管理，以确保系统的可靠性和稳定性。此外，还应考虑数据备份、监控和容灾等其他方面的措施来增强整个系统的可用性。


## 使用Redis实战示例

1. 缓存

    热点数据缓存（例如报表、明星出轨），对象缓存、全页缓存、可以提升热点数据的访问数据。


2. 全局ID

    int类型，incrby，利用原子性

    incrby userid 1000

    分库分表的场景，一次性拿一段


3. 分布式锁的实现

   String 类型setnx方法，只有不存在时才能添加成功，返回true
    ```bash
    public static boolean getLock(String key) {
        Long flag = jedis.setnx(key, "1");
        if (flag == 1) {
            jedis.expire(key, 10);
        }
        return flag == 1;
    }
    
    public static void releaseLock(String key) {
        jedis.del(key);
    }
    ```


4. 计数器

    int类型，incr方法

    例如：文章的阅读量、微博点赞数、允许一定的延迟，先写入Redis再定时同步到数据库


5. 限流

    int类型，incr方法

    以访问者的ip和其他信息作为key，访问一次增加一次计数，超过次数则返回false


6. 商城的购物车

   String 或hash。所有String可以做的hash都可以做

- key：用户id；field：商品id或者skuid；value：商品数量。
- +1：hincr。-1：hdecr。删除：hdel。全选：hgetall。商品数：hlen。


7. 用户消息时间线timeline

   list，双向链表，直接作为timeline就好了。插入有序


8. 消息队列

List提供了两个阻塞的弹出操作：blpop/brpop，可以设置超时时间

- blpop：blpop key1 timeout 移除并获取列表的第一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

- brpop：brpop key1 timeout 移除并获取列表的最后一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

   上面的操作。其实就是java的阻塞队列。学习的东西越多。学习成本越低

- 队列：先进先除：rpush blpop，左头右尾，右边进入队列，左边出队列

- 栈：先进后出：rpush brpop


9. 抽奖

   自带一个随机获得值
   ```bash
   spop myset
   ```


10. 点赞、签到、打卡

假如：微博ID是t1001，用户ID是u3001
   
用 like:t1001 来维护 t1001 这条微博的所有点赞用户

- 点赞了这条微博：sadd like:t1001 u3001
- 取消点赞：srem like:t1001 u3001
- 是否点赞：sismember like:t1001 u3001
- 点赞的所有用户：smembers like:t1001
- 点赞数：scard like:t1001


11. 用户关注、推荐模型

follow 关注 fans 粉丝
      
相互关注：

- sadd 1:follow 2

- sadd 2:fans 1

- sadd 1:fans 2

- sadd 2:follow 1

我关注的人也关注了他(取交集)：

- sinter 1:follow 2:fans

可能认识的人：

- 用户1可能认识的人(差集)：sdiff 2:follow 1:follow

- 用户2可能认识的人：sdiff 1:follow 2:follow


12. 排行榜

id 为6001 的新闻点击数加1：
```bash
zincrby hotNews:20190926 1 n6001
```

获取今天点击最多的15条：
```bash
zrevrange hotNews:20190926 0 15 withscores
```


##  关键点复习题目

1. Redis为什么那么快？
- 纯内存操作
- 单线程模型，避免频繁的上下文切换
- 合理、简单、高效的数据结构
- 非阻塞I/O：Redis使用异步I/O模型，通过事件驱动机制来处理网络请求。


2. Redis的数据过期策略？

Redis中的数据过期采用了惰性删除和定期删除两种策略。

- 惰性删除（Lazy Expiration）：
当客户端访问一个键时，Redis会检查该键是否已过期。
如果过期，Redis会立即删除它，并返回空值。这种策略保证了过期键不会返回给客户端，但也意味着过期键可能会在一段时间内仍然存在于内存中。
惰性删除是实现过期键删除的主要方式。

- 定期删除（Eviction）：
为了清理那些被错过的或未及时访问的过期键，Redis使用了定期删除策略。
Redis会以一定的频率（默认每秒钟10次）随机抽样一部分键，并检查它们是否过期。如果发现过期键，则立即删除。
定期删除操作是后台线程执行的，可以通过`hz`配置项来调整执行频率。

   需要注意的是，即使启用了定期删除，Redis并不保证对每个过期键都会立即进行删除操作。
   删除过期键是在访问键时进行的，或者通过定期删除操作进行的。
   因此，在某些情况下，过期键可能会在过期后仍然存在一段时间，直到被访问或执行定期删除操作。
   
   除了上述两种策略外，Redis还提供了持久化机制（如RDB快照和AOF日志）来确保过期键在Redis重启后仍然有效。
   这些持久化机制可以将过期时间一同保存下来，以便在系统重启后重新加载并正确处理过期键。
   
   综上所述，Redis的数据过期策略包括惰性删除和定期删除，通过这两种方式配合使用来实现数据的自动过期和清理。


3. Redis的持久化机制是什么？

Redis有两种主要的持久化策略：RDB（Redis Database）和AOF（Append-Only File）。

- RDB全量快照：
RDB是Redis默认的持久化方式。
它通过定时或手动触发生成一个快照(snapshot)来保存数据到磁盘上的二进制文件。
RDB是一个压缩、紧凑的文件，适合用于备份和恢复数据。
然而，使用RDB持久化意味着你将会失去最后一次快照之后的所有数据修改。

- AOF增量日志：
AOF持久化将每个写操作追加到一个只进行追加操作的日志文件中，记录了重建当前数据集所需的所有写命令。
当Redis重新启动时，它会重新执行这些命令来还原数据集。
相对于RDB，AOF提供了更好的数据可靠性，但文件大小会比RDB大，并且在恢复过程中可能需要较长的时间。

- Redis 还可以同时使用 AOF 持久化和 RDB 持久化。 
在这种情况下， 当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。



4. Redis与Mysql双写一致性方案？

- 先更新数据库，再删除缓存。
数据库的读操作的速度远快于写操作的，所以脏数据很难出现。
可以对异步延时删除策略，保证读请求完成以后，再进行删除操作。
- 解析MySQL的binlog实现，将数据库中的数据同步到Redis（Canal/Flink CDC等中间件）


5. Redis到底是单线程还是多线程？

-  其实，redis是单线程的。通常说的单线程，主要指redis对外提供的键值存储服务的主要流程是单线程，也就是网络I/O和数据读写是由单线程来完成的。
除此之外，redis的其他功能，比如持久化、异步删除、集群数据同步等是额外的线程来执行的。
- redis在6.0版本中支持多线程并不是指指令操作的多线程，而是针对网络I/O的多线程支持，也就是redis的命令操作仍然是线程安全的。
在Redis6.0里面，针对网络I/O的处理方式改成了多线程，通过多线程并行的方式提供了网络I/O的处理效率。但是对于客户端指令，还是使用单线程方式来执行的。


6. Redis相比memcached有哪些优势？

- memcached所有的值均是简单的字符串，redis作为其替代者，支持更为丰富的数据类型
- redis的速度比memcached快很多
- redis可以持久化其数据


7. Redis中最大内存（maxmemory）的设置是如何的？
- maxmemory : 默认为0 不限制
- maxmemory-policy：默认noeviction，具体策略解释：

   noeviction:不会继续服务写请求( del 请求可以继续服务)，读请求可以继续进行。这样可以保证不会丢失数据，但是会让线上的业务不能持续进行。这是默认的淘汰策略。

   volatile-lru: 尝试淘汰设置了过期时间的 key，最少使用的 key 优先被淘汰。 没有设置过期时间的 key不会被淘汰，这样可以保证需要持久化的数据不会突然丢失。

   volatile-ttl:跟上面几乎一样，不过淘汰的策略不是 LRU，而是比较 key 的剩余寿命时的值，ttl越小越优先被淘汰。

   volatile-random:跟上面几乎一样，不过淘汰的 key 是过期 key 集合中随机的 key。

   allkeys-lru:区别于 volatile-lru，这个策略要淘汰的 key 对象是全体的 key 集 合，而不只是过期的 key 集合。这意味着一些没有设置过期时间的 key 也会被淘汰。

   allkeys-random:跟上面几乎一样，不过淘汰的 key 是随机的 key。
