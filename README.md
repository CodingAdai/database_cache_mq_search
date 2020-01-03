# 数据库&缓存&消息队列&搜索


## 搜索引擎

### Elasticsearch
You know, for search

#### 术语
* 索引：An index is like a table in a relational database. It has a mapping which contains a type, which contains the fields in the index.An index is a logical namespace which maps to one or more primary shards and can have zero or more replica shards.
* 文档：A document is a JSON document which is stored in Elasticsearch. It is like a row in a table in a relational database. Each document is stored in an index and has a type and an id.
* 节点：A node is a running instance of Elasticsearch which belongs to a cluster. Multiple nodes can be started on a single server for testing purposes, but usually you should have one node per server.
* 集群：
* 分片：Each document is stored in a single primary shard. When you index a document, it is indexed first on the primary shard, then on all replicas of the primary shard.
* 副本：Each primary shard can have zero or more replicas. 故障转移和提高性能。


#### 元字段（meta-fields）
* _index
* _type
* _id
* _source
* _size
* _score

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



#### REST APIs 
Index APIs

Document APIs


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




#### 相关性算分，算法：TF-IDF（5） 现在默认是BM 25

词频（term frequency）





#### 集群

分布式特性：

储存的水平扩容，支持PB级数据。
提高系统可用性
选主过程

脑裂问题

集群状态

节点类型
* 处理请求的节点（Coordination node）
* 数据节点（Data node）
分片数的设定
问：primary shard 数量默认为啥是1？（设置为多少比较合适？）

分片路由



---
## 分布式

分布式选举

分布式共识

分布式锁





---

## 关系数据库
MySQL

---

## 文档数据库

MongoDB



---
## 消息队列
RocketMQ

分布式应用系统提供异步解耦和削峰填谷的能力

核心概念：




---
## 缓存
Redis
















