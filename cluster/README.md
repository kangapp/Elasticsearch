# ElasticSearch集群

> es支持集群模式，是一个分布式系统。  
es集群由多个es示例组成  
不同集群通过集群名字来区分，可通过cluster.name来修改  
每一个es实例本质是一个JVM进程，可通过node.name来修改名字

## Cerebro
> ElasticSearch监控工具
- 下载地址：https://github.com/lmenezes/cerebro/releases

## 构建es集群

```
./bin/elasticsearch -Ecluster.name=my_cluster -Epath.data=my_cluster_node1 -Enode.name=node1 -Ehttp.port=5200 -d

./bin/elasticsearch -Ecluster.name=my_cluster -Epath.data=my_cluster_node2 -Enode.name=node2 -Ehttp.port=5300 -d

/bin/elasticsearch -Ecluster.name=my_cluster -Epath.data=my_cluster_node3 -Enode.name=node3 -Ehttp.port=5400 -d
```

### Cluster State
> Cluster State 存储在每个节点上，master维护最新版本并同步给其他节点
- 集群的节点信息，比如节点名称、连接地址等
- 索引信息，包括索引名称、mapping、setting等
- 集群中所有分片的位置
- 所有集群级别的设置

### [Node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)
> 每当您启动Elasticsearch实例时，您都在启动节点。连接的节点的集合称为群集。  
默认情况下，群集中的每个节点都可以处理HTTP和传输流量。  
所有节点都知道群集中的所有其他节点，并且可以将客户端请求转发到适当的节点。

#### Master Node
> 主节点负责集群范围内的轻量级操作，例如创建或删除索引，跟踪哪些节点是集群的一部分以及确定将哪些碎片分配给哪些节点  
Master Node通过集群所有节点选举产生，可以被选举的节点称为master-aligible节点，其配置为`node.master:true`，这个是节点的默认配置

#### Date Node
> 数据节点保存数据并执行与数据相关的操作，例如CRUD，搜索和聚合，配置为`node.data:true`

### 副本与分片
>索引已经创建，增加节点能否提高索引的数据容量？  
不能，索引创建时分片的分布已经确定，新增的节点无法利用。

>索引已经创建，增加副本数能否提高索引的读取吞吐量？（3个分片3个节点）  
不能，新增的副本也是分布在相同的节点，利用了相同的资源，要增加吞吐量，还需要增加节点。

### 集群状态
> 查看集群状态:`GET _cluster/health`
- green 健康状态，指所有主副分片都正常分配
- yellow 指所有主分片都正常分配，但有副本分片未正常分配
- red 有主分片未分配

### 故障转移
> 假设集群由三个节点组成，node1、node2和node3，其中node1是主节点，此时的集群状态是green。  
当node1所在机器宕机导致服务终止，node2和node3发现node1无法响应一段时间后会发起master选举，这里选择node2为主节点，此时由于node1的主分片下线，集群状态为red  
node2发现node1的主分片未分配，会将副本分片提升为主分片，此时所有的主分片正常分配，集群状态为yellow  
node2为node1的分片生成新的副本，集群状态变为green