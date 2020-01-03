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
> 返回一系列数值类型的统计值，包含`min`、`max`、`avg`、`sum`、`count`，还可扩展方差和标准差等
```
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "stats_age": {
      "stats": {
        "field": "age"
      }
    }
  }
}
```
---
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
    "total": 6,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "stats_age": {
      "count": 6,
      "min": 18,
      "max": 28,
      "avg": 21.666666666666668,
      "sum": 130
    }
  }
}
```
- percentiles
> 百分位数统计，例第95个百分位数代表大于所有值95%的值
```
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "per_age": {
      "percentiles": {
        "field": "age",
        "percents": [
          1,
          5,
          25,
          50,
          75,
          95,
          99
        ]
      }
    }
  }
}
```
---
```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 9,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "per_age": {
      "values": {
        "1.0": 13.4,
        "5.0": 15,
        "25.0": 21,
        "50.0": 21,
        "75.0": 28,
        "95.0": 43,
        "99.0": 47.8
      }
    }
  }
}
```
- percentile_ranks
> 百分位数统计，和percentile映射关系相反
```
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "per_rank_age": {
      "percentile_ranks": {
        "field": "age",
        "values": [
          24,
          35
        ]
      }
    }
  }
}
```
---
```
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 9,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "per_rank_age": {
      "values": {
        "24.0": 65.07936507936508,
        "35.0": 72.96296296296296
      }
    }
  }
}
```

## Pipeline
> 管道分析类型，基于上一级局和分析的结果进行再分析

## Matrix
> 