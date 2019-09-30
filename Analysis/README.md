# Analysis

## 正排与倒排索引

## 分词器（Analyzer）

### Character Filters
> 针对原始文本进行处理，可以对文本进行字符的添加、修改和删除。  
>每一个分词器可以没有或有多个Character Filters

### Tokenizer
> 将文本按照一定的规则切分成单词，负责记录单词的顺序和位置以及起始偏移量  
每个分词器有且仅有一个Tokenizer

### Token Filters
> 对Tokenizer处理的单词进行新增、修改和删除操作，不改变位置和偏移量

## Analyze API

### 直接指定analyzer进行测试

```
POST _analyze
{
  "analyzer": "whitespace",
  "text": "hello the world"
}
```
---
```
{
  "tokens": [
    {
      "token": "hello",
      "start_offset": 0,
      "end_offset": 5,
      "type": "word",
      "position": 0
    },
    {
      "token": "the",
      "start_offset": 6,
      "end_offset": 9,
      "type": "word",
      "position": 1
    },
    {
      "token": "world",
      "start_offset": 10,
      "end_offset": 15,
      "type": "word",
      "position": 2
    }
  ]
}
```

### [直接指定索引中的字段进行测试](https://www.elastic.co/guide/en/elasticsearch/reference/current/_testing_analyzers.html)


## [自带分词器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html)

- Standard Analyzer

![configuration](image/standard-configuration.png) 
![configuration](image/standard_definition.png) 


## 自定义分词器
> 好像只能在索引中才能使用分词器的参数
### [Character Filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html)

```
POST _analyze
{
  "tokenizer":      "keyword", 
  "char_filter":  [ "html_strip" ],
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}
```
---
```
{
  "tokens": [
    {
      "token": """

I'm so happy!

""",
      "start_offset": 0,
      "end_offset": 32,
      "type": "word",
      "position": 0
    }
  ]
}
```

### Tokenizers

### Token Filters