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

核心概念：
NameServer（2个或2个以上，避免单点故障）


Broker Server

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
















