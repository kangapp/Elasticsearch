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
```
GET my_index/_search?q=alfred&df=user&sort=age:asc&from=4&size=10&timeout=1s
```
- q  
  指定查询的语句，语法为[Query String Syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html):   
  > `term与phrase`    
  > alfred way 等效于alfred OR way   
  > "alfred way" 词语查询，要求先后顺序  
  > `泛查询`    
  >alfred等效于在所有字段去匹配该term    
  >`指定字段`    
  >name:alfred  
 
- df
  > q中不指定字段时默认查询的字段，如果不指定，es会查询所有的字段
- sort
  > 排序
- timeout
  > 指定超时时间，默认不超时
- from,size
  > 分页参数

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
## [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
> Elasticsearch提供了基于JSON的完整查询DSL（特定于域的语言）来定义查询。将查询DSL视为查询的AST（抽象语法树），它由两种子句组成：

### Leaf query clauses(叶子查询字句)
> 叶子查询子句在特定字段中查找特定值，例如match，term或range查询。这些查询可以自己使用。

#### 全文匹配
>针对text类型的字段进行全文检索，会对查询语句先进行分词处理，如match、match_phrase等query类型

- [match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html)
```
GET my-index/_search
{
  "query": {
    "match": {
      "username": "jack tom"
    }
  }
}
```
> operator参数控制单词间的匹配关系，默认是OR
```
GET my-index/_search
{
  "profile": true,
  "query": {
    "match": {
      "username": {
        "query": "tom jack",
        "operator": "and"
      }
    }
  }
}
```
> minimum_should_match参数指定必须匹配子句的最小数量
```
GET my-index/_search
{
  "profile": true,
  "query": {
    "match": {
      "username": {
        "query": "tom jack",
        "minimum_should_match": 2
      }
    }
  }
}
```
- `match_phrase`
> 检索时对词语顺序有要求
```
GET my-index/_search
{
  "query": {
    "match_phrase": {
      "job": {
        "query": "math english"
      }
    }
  }
}
```
> slot参数表示词条相隔多远时仍能将文档视为匹配，即需要移动词条几次可匹配
```
GET my-index/_search
{
  "query": {
    "match_phrase": {
      "job": {
        "query": "english chinese math",
        "slop": 3
      }
    }
  }
}
```
> analyzer参数指定何种分析器来对该短语进行分词处理
```
GET /_search
{
    "query": {
        "match_phrase" : {
            "message" : {
                "query" : "this is a test",
                "analyzer" : "my_analyzer"
            }
        }
    }
}
```

#### 单词匹配
>不会对查询语句作分词处理，直接去匹配字段的倒排索引，如term、terms、range等query类型

### Compound query clauses(复合查询字句)
> 复合查询子句包装其他叶查询或复合查询，并用于以逻辑方式组合多个查询（例如bool或dis_max查询），或更改其行为（例如constant_score查询）