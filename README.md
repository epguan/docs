[TOC]
#  <center> Elasticsearch的实用介绍</center >
![image](https://github.com/epguan/docs/image/es.png)
## Elasticsearch 是什么？
### 简介
Elasticsearch  是一个开源的搜索引擎，擅长海量数据处理和实时分析，建立在一个全文搜索引擎库 [Apache Lucene™](https://lucene.apache.org/core/) 基础之上。 Lucene  可以说是当下最先进、高性能、全功能的搜索引擎库--无论是开源还是私有。Elasticsearch的目的是使全文检索变得简单， 通过隐藏 Lucene  的复杂性，取而代之的提供一套简单一致的 RESTful API。
然而，Elasticsearch  不仅仅是 Lucene，并且也不仅仅只是一个全文搜索引擎。 它可以被下面这样准确的形容：
1. 一个分布式的实时文档存储，每个字段 可以被索引与搜索
2. 一个分布式实时分析搜索引擎
3. 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据
### Elasticsearch 的哲学
Elasticsearch 做了很多努力和尝试来让复杂的事情变得简单，很大程度上来说 elasticsearch 的成功来源于此。Elasticsearch  能运行在你的笔记本电脑上，或者扩展到上百台服务器上去处理PB级数据。
Elasticsearch  中没有一个单独的组件是全新的或者是革命性的。全文搜索很久之前就已经可以做到了， 就像早就出现了的分析系统和分布式数据库。 革命性的成果在于将这些单独的，有用的组件融合到一个单一的、一致的、实时的应用中。它对于初学者而言有一个较低的门槛， 而当你的技能提升或需求增加时，它也始终能满足你的需求。
### Elasticsearch 与 Hbase比较
Elasticsearch 和 Hbase 都是非常流行的大数据存储方案，有相似也有很大不同。 Hbase 是运行与 hadoop 之上的 Nosql 数据库，它的数据模型：表、行、列族、Cells，它对数据模型的主要操作是 Get, Put, Scan, 和 Delete。
 Elasticsearch 适合应用于流式写入，频繁多字段查询、分析的场景。不适合频繁大量写入、更新数据的场景。 
 Hbase 适合频繁大量数据写入、更新，频繁的 key-value 查询的场景。不适合多字段查询的场景
### 
##  哪些公司在用 elasticsearch ？
![avatar](/Users/dill/Desktop/who_use_es.jpg)
* 国外有Wikipedia、StackOverflow、Github、Facebook、Quora、LinkedIn、Netflix等公司都在使用Elasticsearch
	1. Github 使用 Elasticsearch 搜索20TB的数据，包括13亿的文件和1300亿行的代码•擅长海量数据处理和实时分析
	2. Wikipedia 使用 ES 提供全文搜索并高亮关键字，以及输入实时搜索(search-as-you-type)和搜索纠错(did-you-mean)等搜索建议功能
* 国内像百度、阿里巴巴、腾讯、新浪等公司都在使用
	1. 百度在casio、云分析、网盟、预测、文库、直达号、钱包、风控等业务上都应用了ES，单集群每天导入30TB+数据，总共每天60TB+。

## Elasticsearch 怎么使用？
### 背景知识
在应用程序中对象很少只是一个简单的键和值的列表。通常，它们拥有更复杂的数据结构，可能包括日期、地理信息、其他对象或者数组等。
把对象存储到关系型数据中时，必须将这个对象扁平化来适应表结构，而且又不得不在每次查询时重新构造对象。
Elasticsearch 是面向文档的，意味着它存储整个对象或文档。Elasticsearch 不仅存储文档，而且索引每个文档的内容使之可以被检索。在 Elasticsearch 中，你对文档进行索引、检索、排序和过滤，而不是对行列数据。这是一种完全不同的思考数据的方式，也是Elasticsearch 能支持复杂全文检索的原因。
### 基本概念
1. 接近实时（NRT）
Elasticsearch是一个接近实时的搜索平台。这意味着，从索引一个文档直到这个文档能够被搜索到有一个轻微的延迟（通常是1秒）。

2. 节点（node）
一个运行中的 Elasticsearch 实例称为一个节点。

3. 集群（cluster）去中心化设计
集群是由一个或者多个拥有相同 cluster.name  配置的节点组成，它们共同承担数据和负载的压力。当有节点加入集群或者从集群中移除节点时，集群将会重新平均分布所有的数据。当一个节点被选举称为主节点时，它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。而主节点并不需要涉及到文档级别的变更和搜索等操作，所以集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。作为用户，我们可以将请求发送到集群中的任何节点，包括主节点。每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到我们存储我们所需文档的节点。

![avatar](/Users/dill/Desktop/es_cluster.jpeg)

4. 分片和复制（shard&replica）
Index 可以存储大量的数据，可以超过单个节点的硬件限制。Elasticsearch 提供了将索引划分成多份的能力，这些份就叫做分片。一个分片是一个底层的工作单元，它仅并保存了全部数据中的一部分。一个分片是一个 Lucene 的实例，以及它本身就是一个完整的搜索引擎。我们的文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互。在 Elasticsearch 集群中，索引分片（sharding）是自动完成的，而且所有分片索引（shard）是作为一个整体呈现给用户的。需要注意的是，尽管索引分片这个过程是自动的，但是在应用中需要事先调整好参数。因为集群中分片的数量需要在索引创建前配置好，而且服务器启动后是无法修改的，至少目前无法修改。
分片作用：允许你水平分割/扩展你的内容容量；允许你在分片之上进行分布式的、并行的操作，进而提高性能/吞吐量。
Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片，或者直接叫复制。复制的作用：在分片/节点失败的情况下，提供了高可用性；扩展你的搜索量/吞吐量，因为搜索可以在所有的复制上并行运行。

5. 索引（index）
一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。如果用关系型数据库模型对比，索引的地位与数据库实例（database）相当。索引存放和读取的基本单元是文档（document）。
```
GET accounts
```

6. 类型（type）
一个类型是你的索引的一个逻辑上的分类/分区。

7.  文档（document）
在 Elasticsearch 的世界中，文档是主要的存在实体。所有的 Elasticsearch 应用需求到最后都可以统一建模成一个检索模型：检索相关文档。文档由一个或者多个域组成，每个域由一个域名和一个或多个值组成。在Elasticsearch 中，每个文档都可能会有不同的域集合；也就是说文档是没有固定的模式和统一的结构。文档之间保持结构的相似性即可。从客户端的角度来看，文档就是一个 JSON 对象。

8. mapping
Elasticsearch 的mapping非常类似于静态语言中的数据类型：声明一个变量为int类型的变量， 以后这个变量都只能存储int类型的数据。同样的， 一个number类型的mapping字段只能存储number类型的数据。
同语言的数据类型相比，mapping还有一些其他的含义，mapping不仅告诉ES一个field中是什么类型的值， 它还告诉ES如何索引数据以及数据是否能被搜索到。
```
GET accounts/_mapping
```

### 基本操作（增删改查）
#### 查看基本信息
```js
curl  -XGET http://192.168.2.139:9200
{
  "name" : "HusRq8k",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "Avp4VsJvQrCRbtUVzO_I9w",
  "version" : {
    "number" : "6.3.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "eb782d0",
    "build_date" : "2018-06-29T21:59:26.107521Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```
#### 添加一个文档
```js
POST /accounts/person/1
curl -XPOST "http://192.168.2.139:9200/accounts/person/1" -H 'Content-Type: application/json' -d'
{
    "name" : "John",
    "lastname" : "Doe",
    "job_description" : "Systems administrator and Linux specialit"
}'
```
文档被创建后返回相应信息
```js
{
  "_index": "accounts",
  "_type": "person",
  "_id": "1",
  "_version": 2,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}
```

#### 获取文档
```js
GET /accounts/person/1 
curl -XGET "http://192.168.2.139:9200/accounts/person/1"
```
返回结果中包含元数据和文档的完整内容（放到 _source 字段中）：
```js
{
  "_index": "accounts",
  "_type": "person",
  "_id": "1",
  "_version": 3,
  "found": true,
  "_source": {
    "name": "John",
    "lastname": "Doe",
    "job_description": "Systems administrator and Linux specialit"
  }
}
```
#### 更新文档
```js
POST /accounts/person/1/_update
curl -XPOST "http://192.168.2.139:9200/accounts/person/1/_update" -H 'Content-Type: application/json' -d'
{
      "doc":{
       "job_description" : "Systems administrator and Linux specialist"
     }
}'
```
操作成功后，文档会被更改，可以从再次查询获得的结果验证：
```js
{  
"_index":"accounts",  
"_type":"person",  
"_id":"1",  
"_version":2,  
"found":true,  
"_source":{  
"name":"John",  
"lastname":"Doe",  
"job_description":"Systems administrator and Linux specialist"  
}  
}
```
为了准备接下来的内容，再添加一条索引：
```js
POST /accounts/person/2 
{ 
	"name" : "John", 
	"lastname" : "Smith",
	 "job_description" : "Systems administrator" 
 }
```
#### 查询文档
##### 全文检索
```js
curl -XGET "http//192.168.2.139:9200/_search?q=john"
```
返回结果中将包含先前添加的两个文档，因为两个文档中均包含 John：
```js
{
  "took": 873,
  "timed_out": false,
  "_shards": {
    "total": 53,
    "successful": 53,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "accounts",
        "_type": "person",
        "_id": "2",
        "_score": 0.2876821,
        "_source": {
          "name": "John",
          "lastname": "Smith",
          "job_description": "Systems administrator"
        }
      },
      {
        "_index": "accounts",
        "_type": "person",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "name": "John",
          "lastname": "Doe",
          "job_description": "Systems administrator and Linux specialist"
        }
      }
    ]
  }
}
```
猜猜以下查询会返回哪些文档：
```js
GET /_search?q=smith
GET /_search?q=job_description:john
```
查询时指定索引和类型：
```js
GET /accounts/person/_search?q=job_description:linux
```
#### 删除
删除文档：
```js
DELETE /accounts/person/1
```
删除索引：
```js
DELETE /accounts
```
### 深入搜索
#### 查询全部
```js
GET /shakespeare/_search 
{ "query": { "match_all": {} } }
```
#### 精确查找
```js
POST /shakespeare/line/_search/
{
    "query":{
        "match" : {
            "play_name" : "Henry IV"
        }
    }
}
```

#### 组合过滤器
```js
POST shakespeare/line/_search/
curl -XPOST "http://192.168.2.139:9200/shakespeare/line/_search/" -H 'Content-Type: application/json' -d'
{
    "query":{
     "bool": {
         "must" : [
             {
                 "match" : {
                     "play_name" : "Henry IV"
                 }
             },
             {
                 "match" : {
                     "speaker" : "KING HENRY IV"
                 }
             }
         ]
     }
    }
}'
```

#### 聚合查询
```
GET /shakespeare/_search
{
    "size":0,
    "aggs" : {
        "Total plays" : {
            "cardinality" : {
                "field" : "play_name.keyword"
            }
        }
    }
}
```


## Elasticsearch 用途是什么？
![avatar](/Users/dill/Desktop/es_platform.png)
### Kibana 
