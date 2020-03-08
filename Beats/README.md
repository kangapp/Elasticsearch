# Beats
> Beats are open source data shippers that you install as agents on your servers to send operational data to Elasticsearch

![](images/beats-platform.png)

## Filebeat
>Filebeat is a lightweight shipper for forwarding and centralizing log data. Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
![](images/filebeat.png)
### 简介
- 读取日志文件,但不做数据的解析处理
- 保证数据"At Lease Once",即数据不会丢失
- 其他功能：「处理多行数据」、「解析json格式数据」、「简单过滤功能」

### Filebeats使用

#### 安装(开箱即用)
 [安装文档](https://www.elastic.co/guide/en/beats/filebeat/7.6/filebeat-installation.html)

#### 配置(filebeats.yaml)

#### 配置模板(index template)

#### 配置Kibana Dashboards

#### 运行
> ./filebeat -e -c filebeat.yml -d "publish"

### Ingest Node - Pipeline
>es数据预处理

### Filebeat Modules
>提供众多开箱即用的module，简化配置流程，相关配置位于module文件夹下

#### nginx module
- `./filebeat modules list`
>查看module列表（enabled or disabled）
- `./filebeat modules enabled nginx`
>开启你需要启用的module
- Prospector|Input 配置
>access/config/nginx-access.yml
- Index Template 配置
> files.yml
- Ingest Pipeline 配置
> access/ingest/default.json
- Kibana Dashboards 配置
> Kibana