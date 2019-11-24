# Mapping
>Mapping is the process of defining how a document, and the fields it contains, are stored and indexed

## 作用
- 定义Index下的字段名(Field Name)
- 定义字段的类型，数值型、字符串型、布尔型
- 定义倒排索引的相关配置，是否索引、记录position

## 自定义mapping
> Mapping中的字段类型一旦设定后，禁止直接修改  
---
>查看mapping配置
```
GET my-index/_mapping
```
### [参数](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)
- `dynamic(控制字段的新增):[true|false|strict]`
> ture(默认):允许新增字段  
false:文档可以正常写入，但无法对新字段进行索引，即无法进行查询等操作  
strict:文档不能写入，报错
```
PUT /my-index
{
  "mappings": {
    "doc": {
      "dynamic":"strict",
      "properties": {
        "age":    { "type": "integer" },  
        "email":  { "type": "keyword"  }, 
        "name":   { "type": "text"  }     
      }
    }
  }
}
```
---
- `copy to`  
> 将字段值复制到目标字段  
目标字段不会出现在_source中，仅作查询使用
```
PUT my-index
{
  "mappings": {
    "doc": {
      "properties": {
        "first_name":    { "type": "text", "copy_to": "full_name"},
        "last_name":  { "type": "text", "copy_to": "full_name"}, 
        "full_name":   { "type": "text"  }     
      }
    }
  }
}
```
```
GET my-index/_search
{
  "query": {
    "match": {
      "full_name": {
        "operator": "and",
        "query": "John Smith"
      }
    }
  }
}
```
---
- `index: [true|false]`  
>控制当前字段是否索引，默认为true，即记录索引
```
PUT my-index
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie":    { "type": "text", "index": false }
      }
    }
  }
}
```
> 未索引的字段查询会产生异常
```
GET my-index/_search
{
  "query": {
    "match": {
      "cookie": "cookie message" 
    }
  }
}
```
`"caused_by": {
            "type": "illegal_argument_exception",
            "reason": "Cannot search on field [cookie] since it is not indexed."
          }`

---
- `index_options: [docs|freqs|positions|offsets]`
>用于控制倒排索引记录的内容  
docs:只记录doc id  
freqs:记录doc id和词频  
positions:记录doc id、词频和分词位置  
offsets:记录doc id、词频、分词位置和偏移量  
text类型默认是positions、其他默认为docs，记录内容越多，占用空间越大  
```
PUT my-index
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie":    { "type":"text", "index_options":"positions"}
      }
    }
  }
}
```
---
- `null_value`
>当字段遇到null值得处理策略，es默认会忽略该值  
设置该参数可以设定默认值  
```
PUT my-index
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie":    { "type": "text", "null_value": "NULL" }
      }
    }
  }
}
```

### [数据类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html#mapping-types)
- 核心数据类型
- 复杂数据类型
- 地理数据类型
- 专用类型
- 多字段特性

### [Dynamic Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-mapping.html)
> es可以自动识别文档字段类型，从而创建mapping

