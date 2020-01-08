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
> 分桶类型，类似SQL中的`GROUP BY`语法，按照一定的规则将文档分配不同的桶中，达到分类分析的目的。与指标聚合不同，分桶聚合可以保存子聚合

- terms
> 适用于`keyword`类型字段，如果是`text`类型字段，需要开启`fielddata`，根据分词结果来分桶
```
GET my-index/_search
{
  "size": 0, 
  "aggs": {
    "terms_job": {
      "terms": {
        "field": "job",
        "size": 5
      }
    }
  }
}
```
---
> 文档统计的数值并不是在任何情况下都是完全准确的  
The default `shard_size` is (size * 1.5 + 10),从每个分片中获取前`shard_size`个结果，并将它们组合成最终的前`size`个列表，将产生以下结果  
`doc_count_error_upper_bound`:  该值表示一个term的最大潜在的文档数量，但未计入最终结果列表  
`sum_other_doc_count`: 没有用作最终统计的文档的数量
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
    "terms_job": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "math",
          "doc_count": 8
        },
        {
          "key": "chinese",
          "doc_count": 7
        },
        {
          "key": "english",
          "doc_count": 3
        }
      ]
    }
  }
}
```
- range
> 通过指定数值范围来制定分桶规则
```
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "range_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "key": "<24", 
            "to": 24
          },
          {
            "from": 25,
            "to": 30
          },
          {
            "from": 50
          }
        ]
      }
    }
  }
```
---
```
{
  "took": 0,
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
    "range_age": {
      "buckets": [
        {
          "key": "<24",
          "to": 24,
          "doc_count": 6
        },
        {
          "key": "25.0-30.0",
          "from": 25,
          "to": 30,
          "doc_count": 1
        },
        {
          "key": "50.0-*",
          "from": 50,
          "doc_count": 0
        }
      ]
    }
  }
}
```

- data_range
>日期支持[Date Math](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#date-math)
```
POST /sales/_search?size=0
{
    "aggs": {
        "range": {
            "date_range": {
                "field": "date",
                "format": "MM-yyyy",
                "ranges": [
                    { "to": "now-10M/M" }, 
                    { "from": "now-10M/M" } 
                ]
            }
        }
    }
}
```
---
```
{
    ...
    "aggregations": {
        "range": {
            "buckets": [
                {
                    "to": 1.4436576E12,
                    "to_as_string": "10-2015",
                    "doc_count": 7,
                    "key": "*-10-2015"
                },
                {
                    "from": 1.4436576E12,
                    "from_as_string": "10-2015",
                    "doc_count": 0,
                    "key": "10-2015-*"
                }
            ]
        }
    }
}
```
- histogram
> 直方图，以固定间隔的策略来分割数据  
`interval`:间隔值  
`min_doc_count`:计数值超过该值才显示  
`extended_bounds`:设定直方图范围
```
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "histogram_age": {
      "histogram": {
        "field": "age",
        "interval": 5,
        "min_doc_count": 1,
        "extended_bounds": {
          "min": 0,
          "max": 50
        }
      }
    }
  }
}
```
---
```
{
  "took": 8,
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
    "histogram_age": {
      "buckets": [
        {
          "key": 10,
          "doc_count": 1
        },
        {
          "key": 15,
          "doc_count": 1
        },
        {
          "key": 20,
          "doc_count": 4
        },
        {
          "key": 25,
          "doc_count": 1
        },
        {
          "key": 30,
          "doc_count": 1
        },
        {
          "key": 45,
          "doc_count": 1
        }
      ]
    }
  }
}
```
- date_histogram
> 和普通的`histogram`相似，但是只适用于日期和日期范围  
有两种方式指定间隔:[`Calendar intervals`](elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-datehistogram-aggregation.html#calendar_intervals)、[`Fixed intervals`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-datehistogram-aggregation.html#fixed_intervals)


> `calendar_interval`只能指定单位数量间隔
```
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "month"
            }
        }
    }
}
```

```
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : {
            "date_histogram" : {
                "field" : "date",
                "fixed_interval" : "30d"
            }
        }
    }
}
```
> `key_as_string`的显示和[`format`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html)指定的日期格式有关
```
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "1M",
                "format" : "yyyy-MM-dd" 
            }
        }
    }
}
```
---
```
{
    ...
    "aggregations": {
        "sales_over_time": {
            "buckets": [
                {
                    "key_as_string": "2015-01-01",
                    "key": 1420070400000,
                    "doc_count": 3
                },
                {
                    "key_as_string": "2015-02-01",
                    "key": 1422748800000,
                    "doc_count": 2
                },
                {
                    "key_as_string": "2015-03-01",
                    "key": 1425168000000,
                    "doc_count": 2
                }
            ]
        }
    }
}
```
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

- top_hits
>该聚合器旨在用作子聚合器，以便可以按存储分区汇总最匹配的文档
```
POST /sales/_search?size=0
{
    "aggs": {
        "top_tags": {
            "terms": {
                "field": "type",
                "size": 3
            },
            "aggs": {
                "top_sales_hits": {
                    "top_hits": {
                        "sort": [
                            {
                                "date": {
                                    "order": "desc"
                                }
                            }
                        ],
                        "_source": {
                            "includes": [ "date", "price" ]
                        },
                        "size" : 1
                    }
                }
            }
        }
    }
}
```
---
```
{
  ...
  "aggregations": {
    "top_tags": {
       "doc_count_error_upper_bound": 0,
       "sum_other_doc_count": 0,
       "buckets": [
          {
             "key": "hat",
             "doc_count": 3,
             "top_sales_hits": {
                "hits": {
                   "total" : {
                       "value": 3,
                       "relation": "eq"
                   },
                   "max_score": null,
                   "hits": [
                      {
                         "_index": "sales",
                         "_type": "_doc",
                         "_id": "AVnNBmauCQpcRyxw6ChK",
                         "_source": {
                            "date": "2015/03/01 00:00:00",
                            "price": 200
                         },
                         "sort": [
                            1425168000000
                         ],
                         "_score": null
                      }
                   ]
                }
             }
          },
          {
             "key": "t-shirt",
             "doc_count": 3,
             "top_sales_hits": {
                "hits": {
                   "total" : {
                       "value": 3,
                       "relation": "eq"
                   },
                   "max_score": null,
                   "hits": [
                      {
                         "_index": "sales",
                         "_type": "_doc",
                         "_id": "AVnNBmauCQpcRyxw6ChL",
                         "_source": {
                            "date": "2015/03/01 00:00:00",
                            "price": 175
                         },
                         "sort": [
                            1425168000000
                         ],
                         "_score": null
                      }
                   ]
                }
             }
          },
          {
             "key": "bag",
             "doc_count": 1,
             "top_sales_hits": {
                "hits": {
                   "total" : {
                       "value": 1,
                       "relation": "eq"
                   },
                   "max_score": null,
                   "hits": [
                      {
                         "_index": "sales",
                         "_type": "_doc",
                         "_id": "AVnNBmatCQpcRyxw6ChH",
                         "_source": {
                            "date": "2015/01/01 00:00:00",
                            "price": 150
                         },
                         "sort": [
                            1420070400000
                         ],
                         "_score": null
                      }
                   ]
                }
             }
          }
       ]
    }
  }
}
```

## Pipeline
> 管道分析类型，基于上一级局和分析的结果进行再分析

## Matrix
> 