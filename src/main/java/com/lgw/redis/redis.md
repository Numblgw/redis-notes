# Redis

## NoSql数据库的的四大分类

#### K/V 键值型

#### 文档型数据库（bson）

#### 列存储数据库
#### 图关系数据库
    
## NoSql数据库中的CAP原理

1) Consistency 强一致性

2) Availability 可用性

3) Partition tolerance 分区容错性
    
CAP理论的核心是：一个分布式系统无法同时很好的满足强一致性、可用性、分区容错性这三个需求，最多只能较好的同时满足两个。

因此根据CAP原理将NoSql数据库分成了三大类：

1) CA —— 单点集群，满足一致性、可用性的系统，通常在可扩展性上不太强大。

2) CP —— 满足一致性、分区容忍性的系统，通常性能不高。

3) AP —— 满足可用性、分区容忍性的系统，通常对一致性要求低一些。

分区容错性必须要实现，所以只能从一致性和可用性之间进行选择。
      
## BASE原理

BASE就是为了解决关系型数据库强一致性引起的可用性降低的问题而提出的方案。

它的思想是通过让系统放松某一时刻数据一致性的要求来换取系统整体的伸缩性和性能上的改观。

#### 基本可用

#### 软状态

#### 最终一致
    
## Redis的概念和三个特性

Redis是一个高性能的（key/value）分布式内存数据库，基于内存存储并支持持久化的NoSql数据库。

1) Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载使用。

2) Redis不仅仅支持key/value类型的数据，还支持list、set、hash、zset等。

3) Redis支持数据备份，即master-slave模式数据备份（主从复制）。
    
## Redis 安装和使用

### 安装

1) 使用 wget + 网址 命令下载 redis 压缩包，使用 tar -zxvf 文件名 命令解压文件。

2) 进入 解压后的 redis 文件目录内，执行 make 命令，需要有 gcc 环境。

3) 重复执行 make 命令会提示失败，这时要使用 make distclean 清除之前的安装记录。   
    
4) 使用 make install 命令安装 redis。

### 基本使用

1) 进入 /usr/local/bin 目录下 使用 redis-server xxxx/redis.conf 命令。

2) 使用 redis-cli -p 【端口号】命令，进入到 redis，默认端口号是 6379。

3) 使用 shutdown 然后 exit 关闭 redis。

### redis 启动后相关基础知识

1) redis-benchmark 测试 redis 的一些性能。
    
2) 单进程模型来处理客户端请求。对读写等时间的响应是通过对epoll函数的包装来做到的。redis 的实际处理速度完全依靠主进程的执行效率。
Epoll 是 Linux 内核为处理大批量的文件描述符而做了改进的 epoll，是 Linux 下多路复用 IO 接口 select/poll 的增强版本，它能显著提高
程序在大量并发连接中只有少量活跃的情况下的 cpu 利用率。

3) 默认16个数据库（在 redis.conf 中可以找到相关配置） 数据库的编号是从 0 开始的，可以使用 select 8 这种方式切换到 8 号数据库。

4) 几种常见的数据类型，String、List、Set、Hash、ZSet。

### redis 配置文件 redis.conf

#### 单位定义

大小写不敏感

1k = 1000 bytes 

1kb = 1025 bytes
    
#### includes 可以引入其他配置文件到 redis.conf 中

#### general 通用配置

1) daemonize no

    redis默认不是以守护进程的方式启动的，可以将该选项设置为 yes 使 redis 以守护进程启动。
    
    启用守护进程后，Redis会把pid写到一个pidfile中，在/var/run/redis.pid

2) port 6379

    redis 的端口号，默认为 6379
    
3) tcp-backlog 511     

    设置 tcp 的 backlog，backlog 其实是一个连接队列。
    
    队列总和 = 未完成三次握手队列 + 已完成三次握手队列
    
    在高并发环境下你需要一个高 backlog 值来避免慢客户端连接的问题。
    
4) Examples

    配置 ip 、端口等信息。
    
5) timeout 0

    连接超时时间，设置为 0 则为永不超时。
    
6) tcp-keepalive 0

    tcp 连接检测，如果设置为 0 则不会进行检测，建议设置为 60

7) loglevel notice

    日志级别，初始有 4 中。
    
    1) debug
    
    2) verbose
    
    3) notice
    
    4) warning
    
8) logfile ""

    日志文件的名字，如果不写则会将日志打印到控制台上。
    
#### 缓存过期配置

maxmemory-policy noeviction

1) volatile-lru

    使用 lru 算法移除 key，只对设置了过期时间的键。
    
2) allkeys-lru 

    使用 lru 算法移除 key，对所有 key 有效。
    
3) volatile-random

    在过期集合中移除随机 key，只针对设置了过期时间的键。
    
4) allkeys-random

    在过期集合中移除随机 key，对所有 key 有效。
    
5) bolatile-ttl

    移除那些 ttl（time to live） 值最小的 key，即移除即将过期的 key。
    
6) noeviction

    不进行移除操作，写入失败则返回错误信息。    
    
### 持久化 rdb aof

#### rdb - redis database

- 定义

    在指定的时间间隔内将内存中的数据集快照写入磁盘，也就时 snapshot 快照。它恢复时是将快照文件直接写入内存。
    
    Redis 会单独创建一个子进程来进行持久化，会先将数据写到一个临时文件中，待持久化过程都结束了，再用这个文件替换上次持久化好的文件。
    
    整个过程中主进程不进行任何 IO 操作确保了极高的性能。
    
    如果需要大规模的数据恢复并且对数据的完整性要求不高时，rdb 的方式 要比 aof 的方式更加高效，但 rdb 的方式可能会造成最后一次持久化的数据丢失的情况。

- fork

    fork 是创建一个和当前进程一样的进程，新进程的所有数据（程序计数器，局部变量等）都跟原进程一致，作为原进程的子进程。
    
    这时候要考虑原进程的数据量的问题，如果原进程中数据量过大，fork 时会占用过多的内存。
    
- 生成 dump.rdb 备份文件

    在配置文件中配置 snapshotting，配置 rdb 持久化相关的参数，主要有一下配置：
    
    - save 【seconds】 【changes】 
    
        设置 xx 秒内 修改了 xx 次，则进行 rdb 操作，默认有三种触发 rdb 的条件：
    
        1) 15分钟内 修改 1 次
        
        2) 5分钟内 修改 10 次         这三种情况都会触发 rdb 持久化
        
        3) 1分钟内 修改 10000 次
        
        - save "" 禁用 rdb 备份
        
        - 不能将 dump.rdb 只在本机内备份，防止本机物理故障。
    
        在执行 shutdown 时会立刻生成新的 dump.rdb 文件。
        
        示例中先使用 flushall 清空内存再关机是无法从本机的 dump.rdb 文件中恢复数据的，因为 flushall 之后会生成新的 dump.rdb 文件，而这时内存中是空的导致新生成的文件中也没有数据。

    - Stop-writes-on-bgsave-error yes
    
        在后台保存出错时停止写入内存，默认是 yes，如果不在乎数据一致性则配置成 no。
    
    - rdbcompression yes
    
        是否启用 LZF 压缩算法，默认为启动。
        
    - rdbchecksum yes
    
        在存储快照之后会使用 crc64 算法进行数据校验，大约会增加 10% 的性能消耗。
        
    - dbfilename
    
        备份文件的名称，默认为 dump.rdb
        
    - dir
    
        备份文件的目录，使用 config get dir 可以获得日志、备份等文件的路径。
    
    - save 命令 和 bgsave 命令
    
        save 命令，只进行保存，阻塞其他操作。
        
        bgsave 命令，在后台异步的进行保存操作，同时还可以处理客户端请求，可以通过 lastsave 获得最后一次保存时间。
    
    - 使用 flushall 也会生成 dump.rdb 文件，但里面是空的。

- 缺点 
    
    - 启动子进程会复制数据，使数据量变为两倍，而且数据量大的时候备份会非常消耗时间。
    
    - 最后一次修改的输入容易丢失，也就是在还没满足备份条件的时候如果 redis 宕掉了，那么最后的一点修改没有备份也就无法恢复。
   
#### aof - append only file

- 定义

    - 以日志形式记录每一个写操作，将 redis 执行过的所有写指令指令记录下来。
    
    - 只需追加文件，不允许改写文件，redis 启动之初会读取该文件重新构建数据。
    
    - redis 重启的话就根据日志文件的内容从前到后执行一次以完成数据恢复工作。
    
- 配置文件

    - appendonly no
    
        aof 默认是关闭的，因为 rdb 已经可以满足大多数的需求。
        
    - appendfilename appendonly.aof
    
        append 日志的文件名，默认为 appendonly.aof
    
    - appendfsync 
    
        aof 配置策略
        
        1) always 
        
        同步持久化，每次数据变更会立即记录到磁盘，性能较差但完整性好。
        
        2) Everysec 
        
        默认配置。异步操作，没秒记录，如果一秒内宕机则会有数据丢失。
    
        3) No 不进行持久化。
        
- 案例

    aof 和 rdb 两种备份策略可以同时存在。当 aof 和 rdb 都存在时，会先使用 aof 启动。

    当由于网络中断等原因导致 aof 文件中 有错误的记录时，redis 会启动失败。
    
    这时可以使用，redis-check-aof --fix 文件名，来恢复有错误记录的 aof 文件。
    
- Rewrite

    - 定义
    
        aof 会采用文件追加的方式，文件会越来越大，为了避免这种情况采用重写机制。
            
        当文件大小超过了设置的阈值时，将会进行文件压缩只保留可以恢复数据的最小指令集，可以使用命令 bgrewriteaof

    - 重写原理
    
        aof 文件持续增长而过大时，会 fork 出一条新进程来将文件重写（也是先写临时文件最后在rename）。
        
        遍历新进程中的数据，没条记录有一个set语句。重写 aof 文件的操作并没有读取旧的 aof 文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的 aof 文件，类似于快照。

    - 触发机制
    
        Redis 会记录上一次重写时的 aof 文件大小。默认配置是当 aof 文件大小是上一次 rewrite 后大小的一倍并且文件大小大于 64M 时触发。
        
- 缺点

    相同的数据集而言，aof 的日志文件远大于 rdb 文件。数据恢复速度慢。
    
    
























