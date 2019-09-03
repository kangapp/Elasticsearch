# Elasticsearch

## 基本概念

### Cluster和Node

> Elastic本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个Elastic实例  
单个Elastic实例称为一个节点（node），一组节点构成一个集群（cluster）

### Index

> 索引是具有某些类似特征的文档集合

```bash
//查看当前节点的所有Index  
curl -X GET 'http://localhost:9200/_cat/indices?v'
```

### Document

> Index里面单条记录称为文档，许多条Document构成一个Index
- Json Object,由字段(Field)，常见数据类型(todo)
- 有唯一的ID标识(可自行指定或es自动生成)
- 元数据(todo)

### Type

> 索引的逻辑分组，用来过滤Document

```bash
//列出每个Index所包含的Type  
curl 'localhost:9200/_mapping?pretty=true'
```

## Elasticsearch的安装

### [下载并安装Elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)

### [下载并安装Kibana](https://www.elastic.co/cn/downloads/kibana)

## Elasticsearch使用
### Rest API

> 1、Curl命令行  
> 2、Kibana DevTools
#### 索引API
- 创建索引
> `PUT /test_index`
```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "test_index"
}
```
- 查看索引
> 查看现有索引：`GET _cat/indices`
```
yellow open .kibana    fD0tR-G-TJCuSba2G3bxnQ 1 1 3 2 11.1kb 11.1kb
yellow open test_index YPv3LtrMQvStMLNEMblWaQ 5 1 0 0  1.1kb  1.1kb
yellow open customer   99Yq4u7TSbW_izQIbyRSpg 5 1 1 0  4.4kb  4.4kb
```
- 删除索引
> `DELETE /test_index`
```
{
  "acknowledged": true
}
```

#### 文档API
- 指定ID创建文档
> 如果索引不存在，es会自动创建对应的index和type  
> `PUT /test_index/doc/1
{
  "username":"liufukang",
  "age":24
}`
```
{
  "_index": "test_index",
  "_type": "doc",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```
- 不指定ID创建文档
> `POST /test_index/doc
{
  "username":"tom",
  "age":18
}`
```
{
  "_index": "test_index",
  "_type": "doc",
  "_id": "Dykb8WwBGp2nCf5FqQGw",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```
- 查询指定ID文档
> `GET /test_index/doc/1`
```
{
  "_index": "test_index",
  "_type": "doc",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "username": "liufukang",
    "age": 24
  }
}
```
- 查询所有的文档
> `GET /test_index/doc/_search`

>`{
  "query": {
    "term": {
      "_id":1
    }
  }
}`
```
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "test_index",
        "_type": "doc",
        "_id": "Dykb8WwBGp2nCf5FqQGw",
        "_score": 1,
        "_source": {
          "username": "tom",
          "age": 18
        }
      },
      {
        "_index": "test_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "username": "liufukang",
          "age": 24
        }
      }
    ]
  }
}
```
- 批量查询

> `GET _mget`  
```
{
  "docs":[
    {
      "_index":"test_index",
      "_type":"doc",
      "_id":1
    },
    {
      "_index":"test_index",
      "_type":"doc",
      "_id":3
    }
    ]
}
```
> 同一个index  
> `GET test_index/_mget`
```
{
  "docs":[
    {
      "_type":"doc",
      "_id":1
    },
    {
      "_type":"doc",
      "_id":3
    }
    ]
}
```
> 同index同type  
> `GET test_index/doc/_mget`
```
{
  "ids":[1,3]
}
```
---
```
{
  "docs": [
    {
      "_index": "test_index",
      "_type": "doc",
      "_id": "1",
      "_version": 13,
      "found": true,
      "_source": {
        "username": "liufukang",
        "age": 24
      }
    },
    {
      "_index": "test_index",
      "_type": "doc",
      "_id": "3",
      "_version": 4,
      "found": true,
      "_source": {
        "username": "jack",
        "age": 19
      }
    }
  ]
}
```
- 批量写入文档  
[_bulk](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)

## 项目

- [ElasticSearch搜房网实战](https://github.com/kangapp/Elasticsearch/tree/master/project)
