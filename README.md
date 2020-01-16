# 数据库&缓存&消息队列&搜索引擎


## 搜索引擎

### Elasticsearch
You know, for search

### 用户案例


#### 术语
* 索引：An index is like a table in a relational database. It has a mapping which contains a type, which contains the fields in the index.An index is a logical namespace which maps to one or more primary shards and can have zero or more replica shards.
* 文档：A document is a JSON document which is stored in Elasticsearch. It is like a row in a table in a relational database. Each document is stored in an index and has a type and an id.
* 节点：A node is a running instance of Elasticsearch which belongs to a cluster. Multiple nodes can be started on a single server for testing purposes, but usually you should have one node per server.
* 集群：
* 分片：Each document is stored in a single primary shard. When you index a document, it is indexed first on the primary shard, then on all replicas of the primary shard.
* 副本：Each primary shard can have zero or more replicas. 故障转移和提高性能。


#### 启动集群
集群配置（JVM、日志）
堆大小
默认配置，JVM heap 最小和最大值为1 GB。
通过jvm.options文件配置JVM相关参数。
不超过物理内存的50%（因为ElasticSearch还需要系统内存做其他的事情）

重要的ElasticSearch配置：

设置数据和日志目录。 path.data可以设置多个目录。
集群名称
节点名称
network.host（生产环境不使用127.0.0.1回环地址）
JVM参数

重要的系统配置：





#### Mapping
 元字段（meta-fields）
* _index
* _type
* _id
* _source
* _size
* _score


#### CRUD

Index APIs
```http request
PUT /<index>



PUT /twitter
{
    "settings" : {
        "index" : {
            "number_of_shards" : 3, 
            "number_of_replicas" : 2 
        }
    }
}


PUT /test
{
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "properties" : {
            "field1" : { "type" : "text" }
        }
    }
}

```



Document APIs
```http request

POST /<index>/_create/<_id>




POST twitter/_create/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```



GET <index>/_source/<_id>   // 直接返回原始数据


Search APIs
https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html

支持两种形式：

query string
 request body


请求形式：

```
GET /<index>/_search
{
  "query": {<parameters>}
}
```





#### 搜索流程
第一阶段：query
用户发出搜索请求到ES节点，节点收到请求后，会以Coordinating节点的身份，在主副分片中随机选择分片，发送查询请求。
被选中的分片执行查询，进行排序，然后都会返回from+size排序后的文档id和排序值给Coordinating节点。

第二阶段：fetch
Coordinating Node会将Query阶段，把每个分片获取的排序后的文档id列表，重新进行排序，选取from到from+size个文档的id
以multi get请求的方式，到相应的分片获取详细的文档数据。

文档中的这部分介绍了一些：https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#coordinating-node


#### Analysis




#### Query DSL (Domain Specific Language)
Query Context：判断文档是否匹配，同时计算相关性得分

Filter Context：判断文档是否匹配，不计算也不影响相关性得分。


#### Compound queries
包括有：
bool query（must, should, must_not, or filter clauses），must_not 实际在filter context中执行。



#### Full text queries
包括有：

match：根据提供的文本、数字、日期 查询文档，提供的文本会被analyzed

match pharse：分析提供的文本，创建短语查询

query string（有些难度）

#### Term-level queries
term query  精确词语查询文档（提示：避免对text类型字段 进行 term query。text类型字段的分词 会导致 查询不到相应结果。）

多字段Mapping（对text类型字段设置一个keyword 可以做term查询）

栗子：

```json
{
  "query": {
     "term": {
       "authorId": {
         "value": 10032010223
       }
     }
  }
}
```

fuzzy query 
相似查询

terms query  
对字段的一个或多个值 精确查询

terms set query 对字段（数组或列表类型）中的几个值精确查询。（可设置匹配个数）






#### Geo queries


#### Aggregations
聚合

1. metrics
平均值、最大最小值、和值、 等等
2. bucket
Terms、Range

3. pipeline 





#### 相关性算分，算法：TF-IDF（5） 现在默认是BM 25

词频（term frequency）





#### 集群

高可用
至少3个master节点，避免单点故障。



安全

监控


分布式特性：

储存的水平扩容，支持PB级数据。
提高系统可用性
选主过程

脑裂问题


集群状态

节点类型
* 协调节点（Coordination node，分发请求，聚合数据）
* 数据节点（Data node）
* 主节点（Master）（索引操作，集群状态变更）
* master eligible node（可以参与 master 选举成为master，每个节点默认为master eligible node）
* machine learning
* ingest node


分片数的设定

primary shard设置过小，如果索引数据增长很快，无法通过增加节点对数据扩展
设置过大，导致单个shard数据较小，导致单个节点上的分片较多，影响性能
副本分片数过多，会降低集群整体的写性能。
（官网介绍 elastic.co/guide/en/elasticsearch/reference/current/scalability.html#it-depends）
通过测试确定最佳配置。


问：primary shard 数量默认为啥是1？（设置为多少比较合适？）

分片路由


跨集群复制


深度分页问题
当查询from=900，size=100时，需要在每个分片上查询1000出个文档，然后聚合所有结果(1000*需要查询的分片数)，最后再通过排序选取1000个结果。
页数越大，需要的内存越多，为了避免深度分页带来的开销，ES默认限定到10000个文档。

scroll api


并发控制

内部版本控制
```
  "_seq_no" : 89550,
  "_primary_term" : 1,
```
官方文档中此处有描述：https://www.elastic.co/guide/en/elasticsearch/reference/current/optimistic-concurrency-control.html（不全面）



缓存
Node query cache
由该节点的所有shard共享，只缓存Filter Context相关内容。
cache 采用LRU算法。
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-cache.html

Shard request cache
缓存每个shard的本地执行结果。
只缓存请求size=0, 不缓存hits, 缓存 hits.total, aggregations, and suggestions.



分片生命周期
对于时间序列索引，一般有4个阶段


备份和恢复






---
## 分布式

分布式选举

分布式共识

分布式锁





---

## 关系数据库
MySQL
https://yq.aliyun.com/topic/100

版本特性

安装配置


安全
1、密码
2、数据权限
3、备份


MySQL如何使用索引
https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html



备份与恢复

物理备份
Physical backups consist of raw copies of the directories and files that store database contents. This type of backup is suitable for large, important databases that need to be recovered quickly when problems occur.

逻辑备份
Logical backups save information represented as logical database structure (CREATE DATABASE, CREATE TABLE statements) and content (INSERT statements or delimited-text files). This type of backup is suitable for smaller amounts of data where you might edit the data values or table structure, or recreate the data on a different machine architecture.


使用mysqldump 进行逻辑备份
使用mysqlimport 导入备份数据

使用Binary Log 进行时间点和增量备份。



优化

数据库级别优化
表的结构是否合适。正确的数据类型。例如，执行频繁更新的应用程序通常具有许多表而具有很少的列，而分析大量数据的应用程序通常具有较少的表而具有很多列。

是否设置了正确的索引提供查询效率。

是否使用了合适的储存引擎。

应用程序是否使用了合适的锁策略。


优化 SELECT 语句
select ...where 设置索引，explain 查看使用的索引
每一行结构都执行函数调用会导致效率低
最小化查询中全表扫描的次数


硬件级别优化




InnoDB Storage Engine
Supports transactions, row-level locking, and foreign keys 
multiversion concurrency control 多版本并发控制（MVCC）
Buffer Pool
redo log
undo log

Shared and Exclusive Locks（共享锁和排它锁）
意向锁
记录锁
Gap Locks（间隙锁）
Next-Key Locks
Insert Intention Locks


非锁定一致性读


online DDL

INFORMATION_SCHEMA


复制







---

## 文档数据库

MongoDB



---
## 消息队列
RocketMQ

分布式应用系统提供异步解耦和削峰填谷的能力




配置
brokerRole 默认为ASYNC_MASTER
flushDiskType 默认为 ASYNC_FLUSH
brokerId 为0，代表master， >0为slave



核心概念：
NameServer（2个或2个以上，避免单点故障）

 lightweight service discovery and routing
轻量级服务发现和路由
每个NameServer 会记录完整的路由信息。
broker管理（心跳机制检查 broker 是否live。。10秒检查一次。broker如何注册？）
Routing管理


Broker Server（可以多主多从）
message storage by providing lightweight TOPIC and QUEUE mechanisms.
通过轻量级Topic和Queue机制，提供消息储存。
支持pull和push模式，
消息储存和消息传递，消息查询。
会向所有NameServer 注册自己

组成模块：
Remoting Module （负责与客户端通信交互，自定义通信协议TCP。）
Client Manager（管理client，维护topic订阅）
Store Service（提供消息储存和查询。最复杂和最重要的部分，涉及可靠以及高性能存储设计）
HA Service （负责主从同步）
ndex Service（构建索引，快速消息查询）



Topic


Tag


Message



消息类型
1. 普通消息
2. 顺序消息
3. 广播消息
4. 定时消息
5. 事务消息



Topic与Tag最佳实践（https://help.aliyun.com/document_detail/95837.html）
通过合理的使用 Topic 和 Tag，可以让业务结构清晰，更可以提高效率。


消费幂等

消息重复的场景如下：

发送时消息重复
当一条消息已被成功发送到服务端并完成持久化，此时出现了网络闪断或者客户端宕机，导致服务端对客户端应答失败。 如果此时生产者意识到消息发送失败并尝试再次发送消息，消费者后续会收到两条内容相同并且 Message ID 也相同的消息。

投递时消息重复
消息消费的场景下，消息已投递到消费者并完成业务处理，当客户端给服务端反馈应答的时候网络闪断。为了保证消息至少被消费一次，消息队列 RocketMQ 版的服务端将在网络恢复后再次尝试投递之前已被处理过的消息，消费者后续会收到两条内容相同并且 Message ID 也相同的消息。

负载均衡时消息重复（包括但不限于网络抖动、Broker 重启以及消费者应用重启）
当消息队列 RocketMQ 版的 Broker 或客户端重启、扩容或缩容时，会触发 Rebalance，此时消费者可能会收到重复消息。




broker 角色：
Broker Role is ASYNC_MASTER, SYNC_MASTER or SLAVE.

FlushDiskType :ASYNC_FLUSH 和SYNC_FLUSH






http://rocketmq.cloud/zh-cn/docs/design-store.html

消息存储是RocketMQ中最为复杂和最为重要的一部分，本节将分别从RocketMQ的消息存储整体架构、PageCache与Mmap内存映射以及RocketMQ中两种不同的刷盘方式三方面来分别展开叙述。

1 消息存储整体架构
消息存储架构图中主要有下面三个跟消息存储相关的文件构成。

(1) CommitLog：消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容,消息内容不是定长的。单个文件大小默认1G ，文件名长度为20位，左边补零，剩余为起始偏移量，比如00000000000000000000代表了第一个文件，起始偏移量为0，文件大小为1G=1073741824；当第一个文件写满了，第二个文件为00000000001073741824，起始偏移量为1073741824，以此类推。消息主要是顺序写入日志文件，当文件满了，写入下一个文件；

(2) ConsumeQueue：消息消费队列，引入的目的主要是提高消息消费的性能，由于RocketMQ是基于主题topic的订阅模式，消息消费是针对主题进行的，如果要遍历commitlog文件中根据topic检索消息是非常低效的。Consumer即可根据ConsumeQueue来查找待消费的消息。其中，ConsumeQueue（逻辑消费队列）作为消费消息的索引，保存了指定Topic下的队列消息在CommitLog中的起始物理偏移量offset，消息大小size和消息Tag的HashCode值。consumequeue文件可以看成是基于topic的commitlog索引文件，故consumequeue文件夹的组织方式如下：topic/queue/file三层组织结构，具体存储路径为：$HOME/store/consumequeue/{topic}/{queueId}/{fileName}。同样consumequeue文件采取定长设计，每一个条目共20个字节，分别为8字节的commitlog物理偏移量、4字节的消息长度、8字节tag hashcode，单个文件由30W个条目组成，可以像数组一样随机访问每一个条目，每个ConsumeQueue文件大小约5.72M；

(3) IndexFile：IndexFile（索引文件）提供了一种可以通过key或时间区间来查询消息的方法。Index文件的存储位置是：$HOME \store\index${fileName}，文件名fileName是以创建时的时间戳命名的，固定的单个IndexFile文件大小约为400M，一个IndexFile可以保存 2000W个索引，IndexFile的底层存储设计为在文件系统中实现HashMap结构，故rocketmq的索引文件其底层实现为hash索引。

在上面的RocketMQ的消息存储整体架构图中可以看出，RocketMQ采用的是混合型的存储结构，即为Broker单个实例下所有的队列共用一个日志数据文件（即为CommitLog）来存储。RocketMQ的混合型存储结构(多个Topic的消息实体内容都存储于一个CommitLog中)针对Producer和Consumer分别采用了数据和索引部分相分离的存储结构，Producer发送消息至Broker端，然后Broker端使用同步或者异步的方式对消息刷盘持久化，保存至CommitLog中。只要消息被刷盘持久化至磁盘文件CommitLog中，那么Producer发送的消息就不会丢失。正因为如此，Consumer也就肯定有机会去消费这条消息。当无法拉取到消息后，可以等下一次消息拉取，同时服务端也支持长轮询模式，如果一个消息拉取请求未拉取到消息，Broker允许等待30s的时间，只要这段时间内有新消息到达，将直接返回给消费端。这里，RocketMQ的具体做法是，使用Broker端的后台服务线程—ReputMessageService不停地分发请求并异步构建ConsumeQueue（逻辑消费队列）和IndexFile（索引文件）数据。

2 页缓存与内存映射
页缓存（PageCache)是OS对文件的缓存，用于加速对文件的读写。一般来说，程序对文件进行顺序读写的速度几乎接近于内存的读写速度，主要原因就是由于OS使用PageCache机制对读写访问操作进行了性能优化，将一部分的内存用作PageCache。对于数据的写入，OS会先写入至Cache内，随后通过异步的方式由pdflush内核线程将Cache内的数据刷盘至物理磁盘上。对于数据的读取，如果一次读取文件时出现未命中PageCache的情况，OS从物理磁盘上访问读取文件的同时，会顺序对其他相邻块的数据文件进行预读取。

在RocketMQ中，ConsumeQueue逻辑消费队列存储的数据较少，并且是顺序读取，在page cache机制的预读取作用下，Consume Queue文件的读性能几乎接近读内存，即使在有消息堆积情况下也不会影响性能。而对于CommitLog消息存储的日志数据文件来说，读取消息内容时候会产生较多的随机访问读取，严重影响性能。如果选择合适的系统IO调度算法，比如设置调度算法为“Deadline”（此时块存储采用SSD的话），随机读的性能也会有所提升。

另外，RocketMQ主要通过MappedByteBuffer对文件进行读写操作。其中，利用了NIO中的FileChannel模型将磁盘上的物理文件直接映射到用户态的内存地址中（这种Mmap的方式减少了传统IO将磁盘文件数据在操作系统内核地址空间的缓冲区和用户应用程序地址空间的缓冲区之间来回进行拷贝的性能开销），将对文件的操作转化为直接对内存地址进行操作，从而极大地提高了文件的读写效率（正因为需要使用内存映射机制，故RocketMQ的文件存储都使用定长结构来存储，方便一次将整个文件映射至内存）。

3 消息刷盘


(1) 同步刷盘：如上图所示，只有在消息真正持久化至磁盘后RocketMQ的Broker端才会真正返回给Producer端一个成功的ACK响应。同步刷盘对MQ消息可靠性来说是一种不错的保障，但是性能上会有较大影响，一般适用于金融业务应用该模式较多。

(2) 异步刷盘：能够充分利用OS的PageCache的优势，只要消息写入PageCache即可将成功的ACK返回给Producer端。消息刷盘采用后台异步线程提交的方式进行，降低了读写延迟，提高了MQ的性能和吞吐量。





RocketMQ消息队列集群主要包括NameServe、Broker(Master/Slave)、Producer、Consumer4个角色，基本通讯流程如下：

(1) Broker启动后需要完成一次将自己注册至NameServer的操作；随后每隔30s时间定时向NameServer上报Topic路由信息。

(2) 消息生产者Producer作为客户端发送消息时候，需要根据消息的Topic从本地缓存的TopicPublishInfoTable获取路由信息。如果没有则更新路由信息会从NameServer上重新拉取，同时Producer会默认每隔30s向NameServer拉取一次路由信息。

(3) 消息生产者Producer根据2）中获取的路由信息选择一个队列（MessageQueue）进行消息发送；Broker作为消息的接收者接收消息并落盘存储。

(4) 消息消费者Consumer根据2）中获取的路由信息，并再完成客户端的负载均衡后，选择其中的某一个或者某几个消息队列来拉取消息并进行消费。

从上面1）~3）中可以看出在消息生产者, Broker和NameServer之间都会发生通信（这里只说了MQ的部分通信），因此如何设计一个良好的网络通信模块在MQ中至关重要，它将决定RocketMQ集群整体的消息传输能力与最终的性能。

rocketmq-remoting 模块是 RocketMQ消息队列中负责网络通信的模块，它几乎被其他所有需要网络通信的模块（诸如rocketmq-client、rocketmq-broker、rocketmq-namesrv）所依赖和引用。为了实现客户端与服务器之间高效的数据请求与接收，RocketMQ消息队列自定义了通信协议并在Netty的基础之上扩展了通信模块。

1 Remoting通信类结构


2 协议设计与编解码
在Client和Server之间完成一次消息发送时，需要对发送的消息进行一个协议约定，因此就有必要自定义RocketMQ的消息协议。同时，为了高效地在网络中传输消息和对收到的消息读取，就需要对消息进行编解码。在RocketMQ中，RemotingCommand这个类在消息传输过程中对所有数据内容的封装，不但包含了所有的数据结构，还包含了编码解码操作。

Header字段	类型	Request说明	Response说明
code	int	请求操作码，应答方根据不同的请求码进行不同的业务处理	应答响应码。0表示成功，非0则表示各种错误
language	LanguageCode	请求方实现的语言	应答方实现的语言
version	int	请求方程序的版本	应答方程序的版本
opaque	int	相当于reqeustId，在同一个连接上的不同请求标识码，与响应消息中的相对应	应答不做修改直接返回
flag	int	区分是普通RPC还是onewayRPC得标志	区分是普通RPC还是onewayRPC得标志
remark	String	传输自定义文本信息	传输自定义文本信息
extFields	HashMap<String, String>	请求自定义扩展信息	响应自定义扩展信息


可见传输内容主要可以分为以下4部分：

(1) 消息长度：总长度，四个字节存储，占用一个int类型；

(2) 序列化类型&消息头长度：同样占用一个int类型，第一个字节表示序列化类型，后面三个字节表示消息头长度；

(3) 消息头数据：经过序列化后的消息头数据；

(4) 消息主体数据：消息主体的二进制字节数据内容；

3 消息的通信方式和流程
在RocketMQ消息队列中支持通信的方式主要有同步(sync)、异步(async)、单向(oneway) 三种。其中“单向”通信模式相对简单，一般用在发送心跳包场景下，无需关注其Response。这里，主要介绍RocketMQ的异步通信流程。



4 Reactor多线程设计
RocketMQ的RPC通信采用Netty组件作为底层通信库，同样也遵循了Reactor多线程模型，同时又在这之上做了一些扩展和优化。



上面的框图中可以大致了解RocketMQ中NettyRemotingServer的Reactor 多线程模型。一个 Reactor 主线程（eventLoopGroupBoss，即为上面的1）负责监听 TCP网络连接请求，建立好连接，创建SocketChannel，并注册到selector上。RocketMQ的源码中会自动根据OS的类型选择NIO和Epoll，也可以通过参数配置）,然后监听真正的网络数据。拿到网络数据后，再丢给Worker线程池（eventLoopGroupSelector，即为上面的“N”，源码中默认设置为3），在真正执行业务逻辑之前需要进行SSL验证、编解码、空闲检查、网络连接管理，这些工作交给defaultEventExecutorGroup（即为上面的“M1”，源码中默认设置为8）去做。而处理业务操作放在业务线程池中执行，根据 RomotingCommand 的业务请求码code去processorTable这个本地缓存变量中找到对应的 processor，然后封装成task任务后，提交给对应的业务processor处理线程池来执行（sendMessageExecutor，以发送消息为例，即为上面的 “M2”）。从入口到业务逻辑的几个步骤中线程池一直再增加，这跟每一步逻辑复杂性相关，越复杂，需要的并发通道越宽。

线程数	线程名	线程具体说明
1	NettyBoss_%d	Reactor 主线程
N	NettyServerEPOLLSelector_%d_%d	Reactor 线程池
M1	NettyServerCodecThread_%d	Worker线程池
M2	RemotingExecutorThread_%d	业务processor处理线程池




---
## 缓存
Redis
https://promotion.aliyun.com/ntms/act/redis5.html
https://yq.aliyun.com/topic/129

in-memory data structure store, used as a database, cache and message broker

安装

./redis-server /path/to/redis.conf


配置

bind (安全起见，不配置为绑定所有网络接口)
保护模式
daemonize（yes）
pidfile
port
loglevel
logfile
数据目录dir
maxmemory


安全
可控的网络环境（如防火墙保护）
仅在您使用的网络接口上侦听
密码身份认证
加密通信



复制（Replication）



持久化


高可用



数据类型
Stream


问题
热点key
数据迁移
















