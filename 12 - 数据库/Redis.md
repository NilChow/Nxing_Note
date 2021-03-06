[TOC]

### 应用场景

​	因为传统的关系型数据库如Mysql已经不能适⽤所有的场景了， 比如秒杀的库存扣减，APP首页的访问流量⾼峰等等，都很容易把数据库打崩，所以引入了缓存中间件，比如Redis



### 基本数据结构

基本：

*   string：字符串
*   hash：字典
*   list：列表
*   set：集合
*   sorted-set：有序集合



中级：

*   HyperLogLog：
*   Geo：
*   Pub/Sub：



高级：

*   Redis Module：
*   BloomFilter(**重点**)：
*   Redis-Search：
*   Redis-ML：



### 发布订阅



### 分布式锁



### 持久化

##### 介绍

​	Redis的数据都存放在内存中，如果没有配置持久化，Redis重启后数据就全丢失了，所以需要开启持久化功能，将数据保存在磁盘上，当Redis重启后，可以从磁盘中恢复数据。

##### RDB和AOF持久化

*   RDB：可以在指定的时间间隔内生成数据集的时间点快照
*   AOF：原理是将Redis的操作命令以追加的方式写入文件，在服务启动时，通过执行这些命令来还原数据集
*   同时使用：同时使用以上两种持久化方式的情况下，redis会优先使用AOF来还原数据，因为AOF保存的数据通常比RDB的更加完整



##### RDB持久化

###### 介绍

​	RDB持久化是指，在指定时间间隔内，将内存中的数据集快照写入磁盘。

###### 工作机制

​	默认下，redis将内存中的数据快照保存在名为dump.rdb的二进制文件中。实际操作过程是fork一个子进程，先将数据集写临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。

###### 配置方式

​	在redis配置文件中，对save选项进行配置：

​	Save 60 10

​	表示在60s后，至少有10个key发生变化，就dump内存快照



##### AOF持久化

###### 介绍

​	以日志的形式记录redis服务所处理的每一个操作的命令(查询除外)，可以打开看到详细的操作记录

###### 工作机制

​	当redis重启时，通过执行AOF文件中的命令来重建数据集

###### 配置方式

​	在redis配置文件中存在三种同步方式：

​	appendfsync always：每次有命令写入AOF都会执行同步

​	appendfsync everysec：每秒同步一次，该策略为缺省策略

​	appendfsync no：从不同步，高效但数据不会被持久化

###### 

##### 优缺点对比

*   RDB：体积小，恢复速度快，能最大化redis的性能，但丢失数据的可能性更大
*   AOF：体积大，恢复速度慢，比较消耗redis的性能，但数据安全性更高

|    命令    |     RDB      |     AOF      |
| :--------: | :----------: | :----------: |
| 启动优先级 |      低      |      高      |
|    体积    |      小      |      大      |
|  恢复速度  |      快      |      慢      |
| 数据安全性 | 容易丢失数据 | 根据策略决定 |

​	如果可能承受数分钟内的数据丢失，那么大可以使用RDB持久化方式；并不推荐只使用AOF持久化的方式，因为RDB非常便于数据备份，且恢复速度要快得多

​	若Redis重启时既有rdb文件也有aof文件，那么会优先选择后者，因为后者更安全





### 缓存淘汰策略

##### 介绍

​	当Redis内存超出物理内存的限制时，内存的数据就会开始和磁盘产生频繁的交换。交换会让Redis的性能急剧下降，对于访问量比较频繁的Redis来说，这样的龟速效率基本等于不可用。

​	在生产环境中，一般是不允许Redis出现交换行为的。为了限制最大使用内存，Redis提供了maxmemory来限制内存超出期望大小。

​	当实际内存超出了限制时，Redis提供了几种可选的策略，来让用户决定如何腾出新的空间来继续提供读写服务。



##### 策略

|      方式       |                             解释                             |
| :-------------: | :----------------------------------------------------------: |
|   noeviction    | 不会继续服务写请求(DEL、读服务可以继续)。这样可以保证不会丢失数据，但会让线上的业务不能持续进行。**默认淘汰策略** |
|  volatile-lru   | 淘汰设置了过期时间的key，最少使用的key优先被淘汰。没有设置过期时间的key不会被淘汰，可以保证需要持久化的数据不会突然丢失 |
|  volatile-ttl   |       淘汰设置了过期时间的key，即将过期的key优先被淘汰       |
| volatile-random |              淘汰设置了过期时间的key，随机淘汰               |
|   allkeys-lru   |             从所有数据中，挑选最少使用的key淘汰              |
| allkeys-random  |                    从所有数据中，随机淘汰                    |



##### 总结

*   volatile-xxx：只淘汰设置了过期时间的key
*   all-xxx：从所有key中进行淘汰
*   如果只拿redis做缓存，那应该使用all-xxx的淘汰策略；如果想使用redis的持久化，应当优先选用volatile-xxx策略



### 集群部署

##### 介绍

​	是一个提供在多个Redis的节点间共享数据的程序集



##### 集群的几种方式

*   主从模式
*   哨兵(Sentinel)模式
*   集群(Cluster)模式



##### 主从模式

###### 介绍

​	主从模式是三种集群方式中最简单的，在主从模式中，Redis节点被分为master和slave两类

*   master可以进行读写操作，当操作导致数据变化时，会自动同步给slave
*   salve一般都是只读的，并且接收master同步过来的数据
*   一个master可以拥有多个salve，而一个salve只能对应一个master
*   master挂了之后，不影响slave的读，但redis不再提供写操作(因为不会在slave中选一个作为master)，当master重启后，重新提供写操作的服务
*   slave挂了之后，不影响其他slave和master的读写功能，重启后从master同步数据

###### 工作机制

​	当slave启动后，主动向master发送SYNC同步命令，master收到命令后，在后台保存快照(RDB持久化)，和缓存快照这段时间的命令。然后将这些快照文件，和缓存的命令，发送给slave。slave收到后就会进行数据同步。

​	在复制初始化之后，master每次接收到写命令都会同步给slave，保持主从的一致性。

###### 配置方式

*   主节点配置

    主节点几乎不需要配置改动，如果需要配置认证密码

    *   masterauth选项：masterauth *masterpasswd*

*   从节点配置
    *   slaveof选项：slave *masterip* *masterport*
    *   masterauth选项：masterauth *masterpasswd*



##### 哨兵(Sentinel)模式

###### 介绍

​	主从模式的弊端就是不具备高可用性，当master挂掉后，redis就是去了写操作的功能。

​	因此sentinel模式应运而生，它就是建立在主从模式的基础上，当master挂了之后，提供了从slave中选取新的master的功能。

###### 特性

*   当master挂了之后，sentinel会在slave中选取一个新的master，并修改他们的配置文件(比如其他salve的slaveof属性会指向新的master)
*   当之前的master重启后，它将作为slave接收新的master的同步数据
*   sentinel也是一个进程，有挂掉的可能性，所以sentinel也会启动多个，形成一个sentinel集群
*   多sentinel的时候，几个sentinel之间也会互相监控
*   sentinel最好不要和redis部署在同一台服务器，不然redis的服务挂了之后，sentinel也挂了

###### 工作机制

*   主观下线

​	每个sentinel每秒向它所知的master、slave及其他sentinel发送PING命令，如果距离最后一个PING命令恢复的时间超过了**down-after-millseconds**选项所设置的值，则这个节点实例会被标记为主观下线。

*   客观下线

​	如果一个master被标记为主观下线，则正在监视这个master的所有sentinel要每秒确认一次master的确进入了主观下线状态。当有数量足够多(大于等于配置文件指定)的sentinel在指定时间内确认master的确进入了主观下线状态，则master会被标记为客观下线。若没有足够的sentinel同意master下线，或master重新向sentinel的PING命令返回有效回复，master的下线状态就会消除。

*   INFO命令

​	一般情况下，sentinel会以10s一次的频率向所有已知的master和slave发送INFO命令，当master客观下线时，这个频率变为1s一次。

###### 配置方式

*   主节点配置

    与主从模式相同

*   从节点配置

    与主从模式相同

*   哨兵配置

    *   protected-mode：protected-mode no

        禁止保护模式

    *   sentinel monitor配置：sentinel monitor *master-group-name* *ip* *port* *quorum*

        例子：sentinel monitor mymaster 127.0.0.1 6379 1

        quorum是哨兵判断实例是否下线的个数参数

    *   sentinel down-after-milliseconds配置：master最长相应时间，超过这件时间则视为master主观下线

        sentinel down-after-milliseconds mymaster 30000

    *   sentinel parallel-syncs配置：限制主从切换之后，最多的并行同步数据的从节点数量

        sentinel parallel-syncs  mymaster 1

        这个参数越大，那么主从切换完成的时间就越短，也会导致大量从节点不可提供读服务

    *   sentinel auth-pass配置：sentinel auth-pass mymaster 123456

*   启动哨兵

    命令：$ redis-server sentinel.conf --sentinel



##### 集群(Cluster)模式

###### 介绍

​	sentinel模式基本可以满足一般的生产需求，具备高可用性。但是当数据量大到一台服务器存不下时，就无法满足了。这是需要将数据进行分片，将数据存储到多个redis实例中。

​	cluster模式就是为了解决单机redis容量有限的问题，将数据按照规则分配到多台机器。

​	cluster模式可以看成是sentinel模式和主从模式的结合体，通过cluster可以实现master的重选功能，如果配置两个副本三个分片的话，就需要六个redis实例。

​	使用cluster模式，只需要将redis配置文件中的cluster-enable配置打开即可。每个集群中，至少需要三个master才能正常运行，新增节点非常方便。

###### 工作机制

*   多个redis节点网络互联，数据共享

*   所有的节点都是一主一从(或一主多从)，从不提供服务，仅作为备用
*   不支持同时处理多个key，因为redis需要把key均匀分布到各个节点上，并发量很高的情况下同时创建key-value会降低性能，并导致不可预测的行为
*   可以在线增加、删除节点
*   客户端可以连接任何一个主节点进行读写

###### 配置方式







### 事务



### 缓存雪崩、穿透、击穿

##### 缓存雪崩

###### 介绍

​	当大量的Key在同一时间过期失效，一时间大量的请求全部落到了数据库上，造成了数据库没有响应挂掉了，而重启后又被新的请求挂掉，这样就造成了缓存雪崩。

###### 解决方案

 1.    过期时间随机

       ​	设置过期时间时，尽量避免在同一时间生效。在批量往redis存数据的时候，把每个过期时间都加上一个随机值，这样保证了热点数据不会在同一时间大面积失效。

 2.    数据均匀分布

       ​	如果redis是集群部署，那么可以将数据均匀分布在不同redis库中，也能避免同时失效的问题

3.  不设置过期时间
4.  跑定时任务，在缓存失效前刷进新的缓存



##### 缓存穿透

###### 介绍

​	当有大量Redis中不存在的数据的请求(比如恶意攻击)，由于redis中没有这样的数据，无法将请求拦截，而穿透到了数据库上，导致数据库压力过大而挂掉，就叫缓存穿透。

###### 解决方案

1.  基础验证

    ​	用户验证，token认证，请求参数验证，不合法的请求参数直接return

2.  添加“空数据”

    ​	对于redis和数据库中都没有的数据，可以将key对应的value写为null，然后设置个过期时间，这样可以防止短时间内大量的暴力攻击(或正常的错误请求)

3.  布隆过滤器

    ​	利用布隆过滤器快速判断key是否存在，不存在直接返回，避免大量查询落到数据库上



##### 缓存击穿

###### 介绍

​	与雪崩类似，但与之不同的是，这里说的是单个key突然失效而导致的大量请求打到数据库。某个热点key，在不停的抗这高并发，在key失效的一瞬间，持续的高并发访问击穿缓存直接访问到数据库，导致数据库挂掉。

​	比如拼多多出了个iPhone12一元秒杀活动，过期时间为30s，此时吸引了很多用户抢购，就在30s过期的时候，大量的请求突然全部打到了数据库，导致数据库挂掉。

###### 解决方案

1.  不设置过期时间

2.  加互斥锁

    ​	上面的现象是多个线程同时去查询这条数据，我们可以在第一个查询请求的时候使用一个互斥锁来锁住它，后面的请求就会等待，等第一个请求拿到了数据，然后就写进了redis缓存，当后面的请求进来时，发现已经有了缓存，就直接走缓存了。



##### 总结

​	雪崩时大面积的key同时失效，穿透是redis里面不存在这个key，击穿是某个key突然失效，最终受害者都是数据库



##### 思考

*   未雨绸缪：将redis、MySQL采用高可用的集群部署，防止单点
*   亡羊补牢：服务中进行限流+降级等限制，防止Mysql被打崩溃
*   重振旗鼓：Redis配置好持久化，挂掉重启，快速恢复缓存数据



### 布隆过滤器

