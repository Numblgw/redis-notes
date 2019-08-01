# Redis教程

## NoSql(Not Only Sql)简介

**是什么**

NoSql泛指非关系型数据库，这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展

**分类**

K-V键值对(Redis)  文档型数据库(MongDB) 列存储数据库 图关系数据库

## 分布式数据库CAP原理+BASE

**传统的关系型数据库的四个要素**

ACID即原子性 一致性 隔离性 持久性

**CAP是什么**

C(Consistency)  强一致性 

A(Availability) 高可用性 

P(Partition tolerance)分区容错性

**分布式数据库CAP特性不能三个都满足，只能满足其中的两条，其中P即分区容错性必须要实现。**

传统的Oracle Mysql等满足CA

AP+弱一致性 :大多数网站架构的选择

CP :Redis  Mongodb

**BASE是什么**

BASE是为了解决关系数据库强一致性引起的问题而引发的可用性降低而提出的解决方案。

BA(Basically Available)基本可用

S(Soft state)软状态

E(Eventually consistent)最终一致性

## NoSql数据模型BSON简介

可以把BSON简单的理解为与JSON基本相同

## Redis

### 简介

- 单线程
- K-V键值对
- 默认端口6379
- 共有16个数据库，且索引从零开始
- 命令参考大全:http://redisdoc.com

### 安装

1.常规安装

```shell
#下载
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
#解压
tar -zxvf redis-5.0.5.tar.gz
#make编译
		#不存在gcc时的解决方案
		yum install gcc-c++      make distclean
make
#安装
make install (默认安装目录在/opt/redis-5.0.5/src)
#运行redis,需要修改redis.conf,使得redis可以在后台运行
cd /usr/local/bin/          redis-server /redis/redis.conf
#命令行操作
cd /usr/local/bin/             redis-cli -p 6379
```

2.Dockr安装(推荐)

```shell
#搜索redis镜像
docker search redis
#拉取redis镜像
docker pull redis
#启动redis
docker run -p 6379:6379  -d redis  redis-server --appendonly yes
#连接到redis
docker exec -it [reids容器id] redis-cli
```

**2.安装后的杂项配置**

```shell
#该命令用于测试redis的性能如何
redis-benchmark
#Redis总共有16个数据库，默认从0库开始使用
select 0-15选择要使用的库
#查看某一个库中键的数量
DBSIZE
#FLUSHALL清除所有库中的键值对，FLUSHDB清除当前库中的键值对
FLUSHALL FLUSHDB
```

### 键与常用数据类型

- key

  ```shell
  #查看某一个库中所有的key
  keys *
  #判断当前库中，key是否存在
  EXISTS key
  #将记录从一个库移动到另一个库
  move key db
  #给key设置过期时间
  EXPIRE key seconds
  #查看key还有多少秒过期,-1表示永不过期，-2表示已经过期
  ttl key
  #查看key的类型
  type key
  ```

- String

  ```shell
  #设置键值/根据建获取值/删除键值/向值中追加/根据值得到值的长度
  set key value/get key/del key/append key append_value/strlen key
  #加1/减1/加指定的值/减指定的值，只有当值为数字时才能进行加减
  Incr [key]/Decr [key]/Incrby [key] 数字/Decrby [key] 数字
  #得到值从开始位置到结束位置的值(类似于java中String截取)/设定的范围内进行值的替换
  getrange [key] [开始的位置] [结束的位置]/setrange [key] [开始位置] [new_value]
  #set with expire,设值的同时指定存活时间/set if not exists,只有这个键不存在时才能设定成功
  setex [key] [存活时间] [value]/setnx [key] [value]
  #同时设值多个键值对/同时得到多个键值对/同时设值多个键值对设值前判断该键值对是否已经存在
  mset key1 vakue1 key2 value2 key3 value3/mget k1 k2 k3/msetnx 
  ```

- List

  - 它是一个字符串链表，left right都可以插入 删除
  - 如果键对应的值全部弹出(删除)，则值对应的键也会自动删除
  - 链表对头元素和尾元素操作效率很高，操作中间元素时性能较差

  ```shell
  #从左边插入(后进先出)/从右边插入(先进先出)/读取集合中从start到end位置的键值对
  #例如lpush list0 1 2 3 4 5 ,插入时先插入1,2插入到1的左边，读取出的顺序是5 4 3 2 1
  lpush [key] [value1 value2...]/rpush [key] [value1 value2 value3...]/lrange [key] [start] [end]
  #栈顶元素出栈/栈底元素出栈
  lpop [key]/rpop [key]
  #按照索引得到元素
  lindex [key] [index]
  #得到当前list的长度
  llen [key]
  #从key这个List中删除count个value
  lrem [key] [count] [value]
  #把key对应List从start截取到stop得到的值重新赋值给key
  ltrim [key] [start] [stop]
  #把source列表按照栈底出栈，插入到destination的栈顶
  rpoplpush [source] [destination]
  #改变key对应的List中下标为index的值
  lset [key] [index] [value]
  #在key这个键对应的List中的pivot(一项值)前或者后插入一个值
  linsert [key] [BEFORE|AFTER] [pivot] [value]
  ```

- **Hash(重要)**

  ```shell
  #向Redis中添加记录，键为String类型，值为k-v类型，值的键为child_key，值的值为child_value
  hset [parent_key] [child_key] [child_value]
  #根据键得到值，例如 hget user id 
  hget [parent_key] [child_key]
  #一个向parent_key对应的k-v键值对设值多个值(多个键值对)
  hmset [parent_key] [field1] [value1] [field2] [value2].....
  例如:    hmset custom id 1 name zhangsan sex man
  #一个从key对应的k-v键值对中一次根据多个key得到多个value
  hmget [key] field [field ...]
  #得到所有的某个键对应的所有键值对
  hgetall [key]
  #根据键删除对应键值对中的多个键值对
  hdel [key] [field] [field1]....
  #查看key对应的value中键值对的个数
  hlen [key]
  #判断parent_key对应的多个键值对中是否存在child_key
  hexists [parent_key] [child_key]
  #得到parent_key对应的所有键值对的key/得到parent_key对应的多个键值对的key对应的value
  hkeys [parent_key]/hvals [parent_key]
  #对key对应的键值对中的filed对应的值增加increment/对key对应的键值对中的filed对应的值增加increment(浮点数)
  hincrby [ke]y [field] [increment]/hincrbyfloat [key] [field] [increment]
  #当Redis中不存在该键值对时，向Redis中添加记录,值为k-v类型，值的键为child_key，值的值为child_value
  hset [parent_key] [child_key] [child_value]
  ```

- Set

  ```shell
  #添加一个键值对，值为set，当重复时自动去重/查看key对应set中的元素/
  sdd [key] [value1 value2...]/Smembers [key]
  #查看key对应的set中的元素个数
  scard [key]
  #删除key对应的set中的某个元素
  srem [key] [value]
  #从key对应的set中随机选取count个数(使用场景:100个用户选取10个中奖用户)
  srandmember [key] [count]
  #key对应的set随机出栈count个值，注意srandmember 命令指示产生随机数，不会导致出栈
  spop [key] [count]
  #值的移动，把member从source对应的set移动到destination对应的set
  smove [source] [destination] [member]
  #求差集即在第一个set里面不在第二个set里面的元素
  sdiff [key1] [key2]
  #求交集,元素既在key1对应的set里面可在key2对应的set里面
  sinter [key1] [key2]
  #求并集
  sunion [key1] [key2]
  ```

- Zset(Sorted set)

  ```shell
  #向其中添加zset类型的值，key为键值对的键，score member为排序的依据，value为值
  zadd key score member value1 score member value2 score member value3...
  #正序/逆序查看所有的记录,withscores表示查询结果中是否包含排序标准
  zrange key start stop [WITHSCORES]/zrevrange key start stop [WITHSCORES]
  #查看从最小score到最大score范围内的值，可以使用(表示不包含，limit表示每次从offset偏移量开始，读取count个
  zrangebyscore key [(]min_score [(]max_score [WITHSCORES] [LIMIT offset count]
  例如:
  	zrangebyscore z0 (60 (80
  #删除key对应的zset中的多个值
  zrem key member [member ...]
  #统计key对应的zset有几个元素/统计key对应的zset在min到max区间有几个元素
  zcard key/zcount key min max
  #在key对应的zset集合中得到member的下标/在key对应的zset集合中得到member的逆序下标
  zrank key member/zrevrank key member
  #在key对应的zset中得到member对应的score
  zscore key member
  ```

### 配置文件

- 配置文件放在/usr/redis-5.0.5/中，想要修改该配置文件时先复制出一份到某个路径下，避免造成文件损坏

- Units单位

  **Redis中1k与1kb的区别，另外Redis对大小写不敏感**

  ```shell
  # 1k => 1000 bytes
  # 1kb => 1024 bytes
  # 1m => 1000000 bytes
  # 1mb => 1024*1024 bytes
  # 1g => 1000000000 bytes
  # 1gb => 1024*1024*1024 bytes
  # units are case insensitive so 1GB 1Gb 1gB are all the same.
  ```

- INCLUDES用于包含Redis的其他的配置文件

  ```shell
  # Include one or more other config files here.  This is useful if you
  # have a standard template that goes to all Redis servers but also need
  # to customize a few per-server settings.  Include files can include
  # other files, so use this wisely.
  #
  # Notice option "include" won't be rewritten by command "CONFIG REWRITE"
  # from admin or Redis Sentinel. Since Redis always uses the last processed
  # line as value of a configuration directive, you'd better put includes
  # at the beginning of this file to avoid overwriting config change at runtime.
  #
  # If instead you are interested in using includes to override configuration
  # options, it is better to use include as the last line.
  #
  # include /path/to/local.conf
  # include /path/to/other.conf
  ```

- GENERAL指通用的标准配置

  ```shell
  #是否允许Redis在后台运行，默认为no，如果开启后台运行会在/var/run/redis.pid写入一个pid文件
  daemonize yes
  
  # Redis的日志级别，默认为notice
  # debug (a lot of information, useful for development/testing)
  # verbose (many rarely useful info, but not a mess like the debug level)
  # notice (moderately verbose, what you want in production probably)
  # warning (only very important / critical messages are logged)
  loglevel notice
  
  # 指定日志文件的名称，默认为"",日志文件将保存在/dev/null目录下
  logfile ""
  
  #默认系统日志关闭
  # syslog-enabled no
  
  # 默认系统日志以redis开头
  # syslog-ident redis
  
  # 系统日志设备使用LOCAL0-LOCAL7，默认为local0
  # syslog-facility local0
  
  #总数据库个数为16个，默认使用第0个，数据库下标0-15，可以使用select 0-15进行数据库的切换
  databases 16
  
  ```

- NETWORK

  ```shell
  #Redis绑定可以访问的ip地址，默认为本机
  bind 127.0.0.1
  
  #端口号，如果端口号为0，Redis不会监听TCP连
  port 6379
  
  # TCP listen() backlog.
  # In high requests-per-second environments you need an high backlog in order
  # to avoid slow clients connections issues. Note that the Linux kernel
  # will silently truncate it to the value of /proc/sys/net/core/somaxconn so
  # make sure to raise both the value of somaxconn and tcp_max_syn_backlog
  # in order to get the desired effect.
  tcp-backlog 511
  
  # 空闲多少秒以后关闭这个连接，如果为0则不会自动关闭(0 to disable)
  timeout 0
  
  #单位为秒，如果设置为0则不会进行keepalive检测，建议设置为60，每隔多少秒进行一次检测，检测该redis连接是否可用
  tcp-keepalive 300
  ```

- SECURITY

  连接安装在Linux上的Redis默认不需要输入密码

  ```shell
  #连接Redis以后输入下面命令可以获得默认的密码
  #设定Redis的密码，在设定密码后每次进行Redis的访问都需要输入密码
  #连接Redis进行访问，存在密码时输入密码进行验证密码的命令
  config get requirepass
  config set requirepass
  auth password
  
  #得到启动Redis的路径(一般日志会保存在Redis启动的目录下面)
  config get dirCLIENTS
  ```


- CLIENTS

  ```shell
  # 设置同一时间内最大连接数，默认为10000
  maxclients 10000
  ```

- MEMORY MANAGEMENT

  ```shell
  # 最大的内存用量
  maxmemory <bytes>
  
  #当达到Redis的最大可用内存时，Redis中缓存的过期策略，有一下8种选择:
  # volatile-lru -> Evict using approximated LRU among the keys with an expire set.(使用LRU算法移除key，只对设置了过期时间的key有用)
  # allkeys-lru -> Evict any key using approximated LRU.(使用LRU算法移除key)
  # volatile-lfu -> Evict using approximated LFU among the keys with an expire set.(使用LFU算法移除key，只对设置了过期时间的key有用)
  # allkeys-lfu -> Evict any key using approximated LFU.(使用LFU算法移除key)
  # volatile-random -> Remove a random key among the ones with an expire set.(在过期的集合中随机移除key，只对设置了过期时间的key有用)
  # allkeys-random -> Remove a random key, any key.(随机移除key)
  # volatile-ttl -> Remove the key with the nearest expire time (移除ttl最小的key即最近要过期的key)
  # noeviction -> Don't evict anything, just return an error on write operations.(永不过期，达到最大缓存时报错)
  
  #默认的缓存过期策略
  # maxmemory-policy noeviction
  
  
  #LRU，LFU和最小TTL算法不是精确的算法，而是近似算法（为了节省内存），因此您可以调整它以获得速度或精度。 默认情况下，Redis将检查五个键并选择最近使用的键，您可以使用以下配置指令更改样本大小。
  #默认值为5会产生足够好的结果。 10近似非常接近真实的LRU但成本更高的CPU。 3更快但不是很准确
  # maxmemory-samples 5
  ```

  

  LRU算法基本思想:LRU即Least recently used(最近最少使用)，根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高。

  ​	基本实现思路:

  1. 新数据插入到链表头部； 
  2. 每当缓存命中（即缓存数据被访问），则将数据移到链表头部； 
  3. 当链表满的时候，将链表尾部的数据丢弃。 

​    LFU算法基本思想:Least Frequently(过去最多使用)，根据数据的历史访问频率来淘汰数据，其核心思想是“如果数据过去被访问多次，那么将来被访问的频率也更高。

​	基本实现思路:

​	LFU的每个数据块都有一个引用计数，所有数据块按照引用计数排序，具有相同引用计数的数据块则按照时间排序。

1. 新加入数据插入到队列尾部（因为引用计数为1）；
2. 队列中的数据被访问后，引用计数增加，队列重新排序；
3. 当需要淘汰数据时，将已经排序的列表最后的数据块删除。



### 持久化

- fork是指复制一个与当前进程一样的进程。新进程的所有数据(变量/环境变量/程序计数器等)和数值都与原进程保持一致，但是新进程是作为原进程的子进程的一个全新的进程。
- RDB的dump.rdb文件和AOF的appendonly.aof文件可以同时存在，当同时存在时会优先加载appendonly.aof，因为appendonly.aof中的数据比dump.rdb中的数据更加准确

#### RDB(Redis DataBase)

**是什么**

- 在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshotting快照，它在恢复时将快照文件直接读到内存中
- dump.rdb可能因为网络等原因发生损坏，可以使用redis-check-rdb --fix file_name进行rdb文件的修复
- Redis会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束后，再用这个临时文件替换上次持久化好的文件。在持久化的整个过程中，主线程不进行任何IO操作，这就确保了极高的性能。如果需要进行大规模的数据的恢复，且对于数据恢复的完整性不是非常敏感，那么RDB方式要比AOF更高效，RDB的缺点是最后一次持久化后的数据可能丢弃。

**适用场景以及优缺点**

- 适用于大规模数据以及对数据完整性和一致性要求不高的备份

- 缺点:(1)果Redis意外down掉，最后一次的改的数据不能备份到dump.rbd文件中，

  ​	(2)fork时如果数据过大会造成数据的膨胀

**配置文件中的相关配置(SNAPSHOTTING)**

- shutdown命令和flushdb命令以及save/bgsave命令执行时会立即重新生成dump.rbd文件并替换原先保存的dump.rbd文件。
- 将dump.rdb文件移动到Redis的安装目录下在Redis启动时会自动从dump.rdb进行数据恢复。

```shell
#   使用这个格式的指令完成持久化策略的指定:save <seconds> <changes>

#   如果在second秒内key发生的更改(增删改)次数>=changes，会自动进行持久化操作,例如900秒内，发生1个key的更改，以及60秒内发生10000次的key的更改
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed

#   可以使用save "" 或者不设置任何save指令来取消RDB持久化策略
save ""

#redis中的默认配置如下:
save 900 1
save 300 10
save 60 10000

#默认设置为yes,为yes时表示当后台save操作发生error时，前台停止write操作
stop-writes-on-bgsave-error yes

# 对于存储到硬盘中的快照是否进行压缩，默认为true,Redis会采用lzf算法进行压缩处理，会造成不大的CPU的性能消耗，建议开启
rdbcompression yes

#在存储快照后，可以让Redis使用CRC64算法进行数据的校验，会造成CPU的性能消耗，建议开启
rdbchecksum yes


# 持久化时保存数据的文件名称在Redis中的配置
dbfilename dump.rdb

#dump.rdb文件的保存目录，即在哪个目录启动Redis就保存在哪个目录下，可以通过config get dir得到该目录信息
dir ./
```

#### AOF(Appebd Only File)

**是什么**

- 以**日志**的形式来记录每个**写**操作，将Redis执行过的每个写指令记录下来(读操作不记录)，只可以追加文件但是不可以改写文件，Redis启动时会读取该文件重新构建数据。其实就是Redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作
- appendonly.aof文件可能因为网络原因造成文件的损坏，可以使用redis-check-aof --fix fileName进行appendonly.aof文件的修复

**Rewrite操作**

- Rewrite操作是什么

  由于AOF采用文件追加方式，文件会越来越大，为了避免这种情况，新增加了重写机制，当AOF文件的大小超过所设的的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。

- Rewrite的原理

  AOF文件持续增长而过大时会fork出一条线程来将文件重新(也是写临时文件然后再rename)，遍历新进程的内存中数据，每条记录有一条set语句，冲写aof文件，并没有读取旧的aof文件而是将整个内存中的数据库内容用命令的方式重写一个新的aof文件。

- 触发机制

  Redis会记录上次重写时aof文件的大小，默认配置是当aof文件大小是上次rewrite后文件大小的一部并且文件大于64m时触发



**配置文件中的相关配置(APPEND ONLY MODE)**

```shell
# 是否开启AOF持久化功能，默认为false
appendonly yes


# AOF持久化时在磁盘上保存点文件的名称与类型
appendfilename "appendonly.aof"

#Aof持久化策略支持三种情况
#always 同步持久化，每次数据发生变更就立即记录到磁盘，性能较差但数据完整性较好
#everysec 异步操作，每秒记录，如果一秒内发生宕机，会发生数据丢失，默认策略为everysec
#no是指不进行持久化
appendfsync everysec

#指定Rewrite是相关的配合，percentage是增长了上次文件的多少，min-size是多于多少M时才进行Rewrite
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

```



### 事务

**是什么**

- 可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按顺序的执行而不会被其他命令插入
- Redis对事务的支持是弱支持

**应用场景**

在一个队列中，一次性 顺序性 排他性的执行一系列命令。

**命令执行**

- 在开启事务后，向队列中添加命令时，如果命令在添加时就报错(如命令报错 sett k1 1)，执行事务时所有的操作都会撤销
- 在开启事务后，向队列中添加命令时，如果在添加时不报错，在执行时报错，仅仅报错的命令不执行，其他命令依旧执行
- 在Redis中添加了CAS即悲观锁与乐观锁

```shell
#取消事务，放弃事务快内所有命令
discard
#执行事务快中的所有命令
exec
#标记一个事务快的开始
multi
#监视一个或多个key,如果在事务执行之前，这些key被其他命令改动，那么事务被打断
watch key [key....]
#取消watch命令对所有key的监视
unwatch
```



### 发布订阅机制(很少使用)

**是什么**

进程间的一种消息通信模式:发送者(pub)发生消息，订阅者(sub)接收消息

### 主从复制，读写分离

**是什么**

- 主从复制:主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，master以写为主，slaver以读为主
- 读写分离:master即主机可以读写(通常用作写)，slave只能读，不能写
- 当master主机因为故障原因宕机后，从机依旧是从机，会处于待命模式，当主机恢复时，从机可以恢复与主机的连接
- 当slave从机因为故障原因宕机以后，从机恢复后重新启动，不能自动完成与master主机的联机，除非配置了conf文件

**适用场景**

读写分离 容灾恢复

**实际应用**

- 需要配置从库，不需要配置主库
- 配置命令：在从机上slaveof  主库IP 主库端口

**复制的原理**

- Slave启动成功连接到Master以后会发生一个Sync命令
- Master接到命令启动后台的存盘进程，同时收集所有所有接收到的用于修改数据集的命令，在后台进程执行完毕以后，master将传送整个数据文件到slave,以完成一次完全tongbu
- 全量复制:slave服务在接收到数据文件后，将其存盘并加载到内存中，全量复制一般只发生在从机第一次连接主机时
- 增量复制:master继续将新的所有收集到的修改指令一次传给slave,完成同步，在从机与主机建立连接以后，主机发生的所有更改基本为增量同步



命令操作

```shell
#准备工作
1.复制多份redis.conf文件，依次为redis6379.conf   redis6380.conf     redis6381.conf
2.修改端口号
3.修改pid文件名称
4.开启允许后台运行即daemonize yes
5.修改日志文件名称
6.修改rbd文件名称


#一个主机两个从机
    #启动多个redis进程模拟多个redis
    redis-server /redis/redis63**.cong
    redis-cli -p 63**
    #这个命令可以查看当前Redis的基本信息
    info replication   
    #在从机上输入下面的命令，设置这个Redis为主机的从机
    slaveof 127.0.0.1 6379

#薪火相传模式
#上一个slave可以是下一个Slave的master，Slave同样可以接受其他slave的连接和同步请求，那么该slave就作为了链条中下一个的master，可以有效减轻master的写压力，中间的redis依旧为slave

#反客为主
#当master主机因为故障原因宕机，多个slave中的一个可以通过下面命令转变身份为master主机
slaveof no one
```



**哨兵(sentinel)模式(反客为主的自动版)**

是什么

- 能够后台监控主机是否故障，如果主机发生故障，如果发生故障自动将从库转为主库
- 当原先的故障主机恢复并启动以后，这个Redis及其会自动的转换为slave

命令操作

```shell
#新建sentinel.conf配置文件
touch sentinel.conf
#在配置文件中加入下面内容
#这里的1就表示主机挂掉后，从机进行投票，得票得到后成为主机
sentinel monitor 被监控主机的名字(自己起) ip  端口 1
#启动哨兵
redis-sentinel /redis/sentinel.conf
```



### Jedis

SpringBoot+Redis的整合参见SpringBoot文档



