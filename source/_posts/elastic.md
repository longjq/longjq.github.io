---
title: Elasticsearch
date: 2017-07-29 10:40:50
categories: Elastic
tags:
  - Elastic
---
###  什么是Elasticsearch

#### 关于Elasticsearch的一些说明

* Node（节点）：单个的装有Elasticsearch服务并提供故障转移和扩展的服务器
* Cluster（集群）：一个集群就是由一个或多个node组织在一起，共同工作，共同分享整个数据具有负载均衡功能的集群
* Document（文档）：一个文档是一个可被索引的基础信息单元
* Index（索引）：索引就是一个拥有几分相似特征的文档集合
* Type（类型）：一个索引中，你可以定义一种或躲着类型
* Field（列）：Field是Elasticsearch的最小单位，相当于数据的某一列
* Shards（分片）：Elasticsearch将索引分成若干份，每个部分就是一个shard
* Replicas（复制）：Replicas是索引一份或多份拷贝

#### 与一般的关系型数据库的对比

| 关系型数据库（例如mysql） | 非关系型数据库（Elasticsearch） |
| --------------- | ---------------------- |
| 数据库Database     | 索引Index                |
| 表Table          | 类型Type                 |
| 数据行Row          | 文档Document             |
| 数据列Column       | 字段Field                |

#### 架构图

![架构图](https://ss0.bdstatic.com/94oJfD_bAAcT8t7mm9GUKT-xh_/timg?image&quality=100&size=b4000_4000&sec=1472129463&di=6cb8d5b6d510439cd0cfb32654655c6e&src=http://images.cnitblog.com/blog2015/603001/201503/031612296601250.png)

### bulk批量操作

| action（行为） | 解释            |
| ---------- | ------------- |
| create     | 当文档不存在时创建     |
| index      | 创建新文档或替换已有的文档 |
| update     | 局部更新文档        |
| delete     | 删除一个文档        |
<!--more-->
例：`{"delete":{"_index":"library","_type":"books","_id":"1"}}`

下面是一些批量操作

```
POST /librarys/books/_bulk
{ "index": { "_id":1 } }
{ "title":"ELASTICSEARCH: THE DEFINETIVE GIDE2","price":5 }
{ "index": { "_id":2 } }
{ "title":"ELASTICSEARCH: cookbook2","price":51 }
{ "index": { "_id":3 } }
{ "title":"THINK IN PHP2","price":5 }
{ "index": { "_id":4 } }
{ "title":"THINK IN PATTERN","price":5 }
{ "index": { "_id":5 } }
{ "title":"HELLO WORLD","price":5 }

GET /librarys/books/_mget
{
    "ids" : ["1","2","3","4","5"]
}

POST /librarys/books/_bulk
{ "delete": {"_index": "librarys", "_type": "books", "_id": "1"} }
{ "create": {"_index": "music","_type":"classical","_id":"1"}}
{ "title": "love is forvery"}
{ "index": { "_index": "music", "_type": "classical"}}
{ "title": "live is forvery"}
{ "update": { "_index":"librarys","_type":"books","_id":"2"}}
{ "doc": {"price":"99999"}}
```

`PS:提交的Body数据必须一行一个action操作下一行为body数据，中间必须要用\n隔开`

#### bulk处理文档大小的最佳值

* 数据加载在每个节点里的RAM里
* 请求的数据超过一定大小，那bulk的处理性能就会下降
* 文档数据大小跟硬件配置，文档复杂度，以及当前集群负载有关

`PS:刚开始可以先处理小一点的数据，根据插件查看性能，如果没有问题再加大处理的数据`

### 版本控制

#### 乐观锁和悲观锁

悲观锁：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作，并发量大的时候会产生性能阻塞，产生一些性能问题

乐观锁：假定不会发生并发冲突，只在提交操作时检查是否违反数据的完整性，如果提交时发现目标被修改了，会提交失败，同时会产生大量的查询操作，因为需要检查数据完整性，所以数据的实时性较差，读取的数据都是脏读、脏数据

#### 内部版本和外部版本

内部版本控制：使用自带的`_version`整型的自增长关键字，修改数据后，`_version`会自动加1

```
# 新建一条数据，注意查看返回的_version关键字，版本为1
PUT /librarys/books/1
{
    "title": "Elasticsearch: THE definetive guid",
    "name" : {
        "first":"longjq",
        "last":"Tong"
    },
    "publish_date":"2015-12-12",
    "price":59.99
}

# 查询id=1的数据，查看_version版本字段，版本为1
GET /librarys/books/1

# 修改id=1的数据中的字段，查看_version版本，版本为2，说明已修改成功
POST /librarys/books/1/_update
{
    "doc":{
        "price":15
    }
}

# 指定修改id=1，version=2的数据，返回version=3，说明修改成功
POST /librarys/books/1/_update?version=2
{
    "doc":{
        "price":10
    }
}

```



外部版本控制：为了保持_version与外部版本控制的数值一致，使用version_type=external关键字，检查数据当前的version值是否小于请求中的version值，只有请求的version值大于数据中的version值的时候，才能修改成功

```
# 覆盖操作，使用外部控制version_type=external参数，指定id=1的version的版本为version=15的版本
PUT /librarys/books/1?version=15&version_type=external
{
    "title": "Elasticsearch: THE definetive guid",
    "name" : {
        "first":"longjq",
        "last":"Tong"
    },
    "publish_date":"2015-12-12",
    "price":33.33
}
# 外部控制version版本与控制的版本一致时，操作失败，小于也失败，这里不演示，可自行操作，总而言之，给的version一定要大于数据中的version值
PUT /librarys/books/1?version=15&version_type=external
{
    "title": "Elasticsearch: THE definetive guid",
    "name" : {
        "first":"longjq",
        "last":"Tong"
    },
    "publish_date":"2015-12-12",
    "price":33.33
}

GET /librarys/books/1
```

`PS:给定version时，其值必须为整型`

### Mapping 映射

#### 什么是Mapping映射

就是类似于mysql的表结构，创建索引的时候，预先定义相关字段的类型以及相关的属性

`PS：如果没有设置映射的话，Elasticsearch会自动分析数据内容，创建字段的映射，这里可以理解为手动设置映射关系`

#### 它有什么用

让索引建立的更加细致和完善

#### 映射的分类

分为`静态映射`和`动态映射`

#### 字段类型

[去官网查看](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)

`PS:记录该学习笔记的时候，使用的是Elasticsearch 2.3的版本`

以下截图为常用字段，2.3版本新增了很多字段，建议去官网查看

![常用字段](C:\Users\Administrator\Pictures\fields.png)

#### 属性方法

[去官网查看](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)

除了定义字段类型，还可以给字段添加相关属性

如何理解这个属性方法？

字段类型是约束存入的数据类型格式，属性方法是给该位置的字段添加相关的属性，属性不同会产生不同的效果，类似于mysql的default、主键、自增长

截图是常用属性方法

![常用属性方法](C:\Users\Administrator\Pictures\field_fun.png)

#### 动态映射

* 什么是动态映射

文档中碰到一个以前没见过的字段时，动态映射自动决定该字段的类型，并对该字段添加映射

* 如何配置动态映射

通过dynamic属性进行控制

true：默认值，动态添加自动；false：忽略新自动；strict：碰到陌生自动，抛出异常。

* 适用范围

适用在根对象上或者object类型的任意字段上

#### 代码演示

* 建立和查看映射

```json
# 建立映射
POST /library
{
    "settings": {
        "number_of_shards":5,
        "number_of_replicas": 1
    },
    "mappings": {
        "books": {
            "properties": {
                "title": { "type": "string" },
                "name": { "type": "string", "index": "not_analyzed" },
                "publish_date": { "type": "date", "index" : "not_analyzed"},
                "price": { "type": "double" },
                "number": { "type": "integer" }
            }
        }
    }
}

# 动态映射(注意：2.3的版本貌似已经修改，该处建议查看官网文档)
PUT /library
{
    "mappings": {
        "books": {
            "dynamic": "strict",
            "properties": {
                "title": { "type": "string" },
                "name": { "type": "string", "index": "not_analyzed" },
                "publish_date": { "type": "date", "index" : "not_analyzed"},
                "price": { "type": "double" },
                "number": { 
                    "type": "object",
                    "dynamic": true
                }
            }
        }
    }
}

# 查看library索引信息，包含映射信息
GET /library

# 获取library索引映射信息
GET /library/_mapping

# 获取library索引下的books映射信息
GET /library/_mapping/books

# 获取整个集群内的索引信息
GET /_all/_mapping

# 获取整个集群内的type为books和music的映射信息
GET /_all/_mapping/books,music

```

* 删除映射

```json
# 删除映射
DELETE /library/books/_mapping
DELETE /library/_mapping/books
```

* 更新映射

很遗憾，Mapping映射一旦建立，就不能修改！！！

只能新建另外一个索引，然后将之前的数据导入到新的索引中

------------操作方法---------------
1.给现有索引定义一个别名，并且把现有的索引指向这个别名

```
PUT /现有索引/_alias/别名A
```

2.新创建一个索引，定义好需要的映射

3.把别名指向新的索引，并取消之前的索引的指向

```
POST /_aliases
{
	"actions":[
  		{ "remove": { "index": "现有索引名","alias": "别名A" } }，
  		{ "add": { "index": "新建索引名","alias": "别名A" } }，
	]
}
```

通过以上三步骤可实现索引平衡过渡，且是零停机执行

### 查询

#### 基本查询

利用Elasticsearch内置查询条件进行查询，文档最后有示例数据

```json
GET /library/books/

# 指定index名以及type名的搜索
GET /library/books/_search?q=title:ELASTICSEARCH

# 指定index名没有type名的搜索，跨type类型搜索
GET /library/_search?q=title:ELASTICSEARCH

# 没有index名也没有type名的搜索，跨index索引搜索
GET /_search?q=title:ELASTICSEARCH

#------------------------------------------------
# term查询
# term查询：查询某字段里有某个关键字的文档
GET /library/books/_search
{
    "query": {
        "term": {
            "preview": "ut"
        }
    }
}


# terms查询：查询某个字段有多个关键词的文档
# minimum_match：最小匹配，1 说明两个关键词里最少要有一个 2 说明文档这两个关键词必须存在
# minimum_match 该参数在2.3的版本中不支持在terms中使用了，请去官网查询
GET /library/books/_search
{
    "query":{
        "terms": {
           "preview": [
              "doloremque",
              "Odit"
           ],
          "minimum_match": 2
        }
    }
}

# -----------------------------------------------


GET /library/books/_search?q=title:ELASTICSEARCH

# 控制查询返回数量
# 相当于mysql中的limit
# form ：从哪个结果开始返回，2.3版本中，整型变量变成了对象，请去官网查询
# size ：定义返回最大的结果数
GET /library/books/_search
{
    "from": 0,
    "size": 20,
    "query": {
        "term": {
           "title": {
              "value": "ELASTICSEARCH"
           }
        }
    }
}
```

































































#### 组合查询

把多个基本查询组合在一起的复合性查询

#### 过滤

查询同时，通过filter条件在不影响打分的情况下筛选出想要的数据



library数据

<span id="library">

```json
[
    {
        "title": "minima",
        "price": 49.45,
        "preview": "Earum inventore velit quidem incidunt alias rem esse pariatur. Laborum sed quo nulla consectetur voluptatem consequatur delectus. Culpa quo tempora atque explicabo sed officia eos consequatur.",
        "publish_date": "2011-02-13"
    },
    {
        "title": "eveniet",
        "price": 46.59,
        "preview": "Aliquam necessitatibus nostrum pariatur ut hic. Voluptatum rerum occaecati natus nostrum dolor. Quibusdam omnis quod voluptas nostrum.",
        "publish_date": "1986-10-27"
    },
    {
        "title": "voluptatem",
        "price": 29.99,
        "preview": "Voluptatem a recusandae inventore error. Ut et beatae nulla enim saepe et. Pariatur fuga quam ut. Ducimus qui ut est ut dolor.",
        "publish_date": "2012-07-12"
    },
    {
        "title": "tempore",
        "price": 74.42,
        "preview": "Eveniet facilis dolor repellat nesciunt eum quas. Ipsa ab harum et et. Autem fuga voluptas modi omnis ad et. Inventore in autem cum atque molestiae odit perferendis.",
        "publish_date": "1981-11-26"
    },
    {
        "title": "voluptatem",
        "price": 99.97,
        "preview": "Dolores quis expedita libero id neque. Non est culpa perspiciatis repudiandae deserunt quaerat eius. Quo laboriosam officiis voluptate voluptas quia corporis ut.",
        "publish_date": "1987-01-05"
    },
    {
        "title": "dolores",
        "price": 35.27,
        "preview": "Porro atque necessitatibus sed doloremque qui. Enim dolores illum dolorum qui incidunt.",
        "publish_date": "2006-03-19"
    },
    {
        "title": "pariatur",
        "price": 22.87,
        "preview": "Quis facere blanditiis voluptatem at non aperiam ab ut. Voluptatum ipsum necessitatibus et et. Voluptatem culpa quia architecto. Deleniti aut officia temporibus est.",
        "publish_date": "1982-09-09"
    },
    {
        "title": "et",
        "price": 37.13,
        "preview": "Odit doloremque magni accusamus cupiditate soluta nam assumenda qui. Labore nulla voluptatem reiciendis. Excepturi ex sapiente dolor aut totam provident. Velit et ut adipisci quaerat nihil qui.",
        "publish_date": "2015-09-25"
    },
    {
        "title": "maxime",
        "price": 8.78,
        "preview": "Quaerat eos repudiandae inventore. Voluptates qui est illum ad sed. Necessitatibus ab pariatur quod.",
        "publish_date": "2011-10-31"
    },
    {
        "title": "aspernatur",
        "price": 40.39,
        "preview": "Aut quibusdam tempora rerum. Doloremque tempore laborum dolores mollitia fugiat. Nemo blanditiis exercitationem occaecati modi repudiandae.",
        "publish_date": "2008-01-06"
    }
]
```

</span>

