# [Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html) 
> 聚合分析作为seach的一部分，其基本的api结构如下：
```
["aggregations"|"aggs"] : {
    "<aggregation_name>" : {
        "<aggregation_type>" : {
            <aggregation_body>
        }
        [,"meta" : {  [<meta_data_body>] } ]?
        [,"aggregations" : { [<sub_aggregation>]+ } ]?
    }
    [,"<aggregation_name_2>" : { ... } ]*
}
```
## Bucketing
> 分桶类型，类似SQL中的`GROUP BY`语法

## Metric
> 指标分析类型，如计算最大值、最小值、平均值等等

### 单值分析
> 输出一个分析结果
- min | max | avg | sum
```
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "min_age": {
      "min": {
        "field": "age"
      }
    },
    "max_age": {
      "max": {
        "field": "age"
      } 
    },
    "avg_age": {
      "avg": {
        "field": "age"
      }
    },
    "sum_age": {
      "sum": {
        "field": "age"
      }
    }
  }
}
```
---
```
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 6,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "max_age": {
      "value": 28
    },
    "avg_age": {
      "value": 21.666666666666668
    },
    "sum_age": {
      "value": 130
    },
    "min_age": {
      "value": 18
    }
  }
}
```
- cardinality
> 用于计算不同值得近似计数，类似SQL中的`distinct count`的概念
```
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "count_job": {
      "cardinality": {
        "field": "job.keyword"
      }
    }
  }
}
```
---
```
{
  "took": 21,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 6,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "count_job": {
      "value": 5
    }
  }
}
```

### 多值分析
> 输出多个分析结果
- stats


## Pipeline
> 管道分析类型，基于上一级局和分析的结果进行再分析

## Matrix
> 