# Search API
> 实现对es中存储的数据进行查询分析，endpoint为_search
```
GET /<index>/_search

POST /<index>/_search

GET /_search

POST /_search
```
> \<index>:索引名称的逗号分隔列表或通配符表达式

## 查询的形式

### [query string as a parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html#search-uri-request)
> 请求URI搜索不支持完整的Elasticsearch Query DSL，但便于测试
```
GET twitter/_search?q=user:kimchy
```

#### 查询参数
- q
  > 指定查询的语句，语法为`Query String Syntax` 
- df
  > q中不指定字段时默认查询的字段，如果不指定，es会查询所有的字段
- sort
  > 排序
- timeout
  > 指定超时时间，默认不超时
- from,size
  > 分页参数

```
GET my_index/_search?q=alfred&df=user&sort=age:asc&from=4&size=10&timeout=1s
```

### [request body](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)

>可以使用搜索DSL来执行搜索请求，该搜索DSL包括查询DSL
```
GET /twitter/_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```