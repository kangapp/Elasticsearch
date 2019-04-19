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


