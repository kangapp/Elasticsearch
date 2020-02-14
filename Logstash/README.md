# Logstash

## Logstash架构
> The Logstash event processing pipeline has three stages: inputs → filters → outputs. `Inputs` generate events, `filters` modify them, and `outputs` ship them elsewhere. Inputs and outputs support `codecs` that enable you to encode or decode the data as it enters or exits the pipeline without having to use a separate filter.

![](images/logstash.png)
> 每一个Batcher对应一个工作线程

![](images/logstash_d.png)
### Queue
#### 分类
- In Memory
> 无法处理进程Crash、机器宕机等情况，会导致数据丢失
- [Persistent Queue](https://www.elastic.co/guide/en/logstash/current/persistent-queues.html) In Disk
> 可处理进程Crash等情况，保证数据不丢失  
保证数据至少消费一次  
充当缓冲区，可以替代kafka等消息队列的作用
---
> 相关配置：
> - queue.type：persisted（默认是memory）
> - queue.max_bytes：4gb（队列存储的最大数据量）

### 线程
- Input Thread

- Pipeline Worker Thread
> 相关配置：
> - pipeline.workers|- w（pipeline线程数，即filter_output处理线程数，默认数cpu核数）
> - pipeline.batch.size|-b（Batcher一次批量获取待处理文档数）
> - pipeline.batch.delay|-u（Batcher等待的时长，单位为ms）

## Logstash配置
### logstash设置相关配置文件
- logstash.yml
- jvm.options
### pipeline配置文件
#### 字段类型
- Boolean  
isFaile => true
- Number  
port => 33  
- String  
name => "Hello World"
- Array  
- Hash

#### 字段值引用
- 直接引用，使用[]
- 在字符串中引用字段值，使用%{}

## [Input Plugins](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)
### sdtin

- add_field
> Value Type:hash  
Default Value:{}  
给Event添加新的字段
- codec
> Value Type:[codec](https://www.elastic.co/guide/en/logstash/7.5/configuration-file-structure.html#codec)(编解码器)  
Default Value:"line"
- tags
> Value Type:array  
> 给Event添加标签用于后续处理
- type
> Value Type:String  

`input-stdin.conf`
```
input {
    stdin {
        codec => "plain"
        tags => ["test"]
        type => "std"
        add_field => {"key" => "value"}
    }
}

output {
    stdout {
        codec => "rubydebug"
    }
}
```
---

`echo "Hello World"|bin/logstash -f config/input-stdin.conf`
```
{
    "message" => "Hello World",
    "host" => "liufukangdeMacBook-Pro.local",
    "@timestamp" => 2020-02-09T14:26:16.711Z,
    "tags" => [
        [0] "test"
    ],
    "type" => "std",
    "key" => "value",
    "@version" => "1"
}

```
### file

### kafka