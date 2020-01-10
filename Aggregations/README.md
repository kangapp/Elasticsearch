
目录
=================

   * [<a href="https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html" rel="nofollow">Aggregations</a>](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)
      * [作用范围]()
      * [类型]()
         * [Bucketing]()
         * [Metric]()
            * [单值分析]()
            * [多值分析]()
         * [Pipeline]()
            * [Parent(父辈，基于histogram)]()
            * [Sibling(同辈)]()
         * [Matrix]()
      * [排序]()


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

## 作用范围
> 默认作用范围query的结果集，可以通过以下方法改变其作用范围
- filter/filters(属于bucket的一种，将当前聚合范围缩小)
```
POST /sales/_search?size=0
{
    "aggs" : {
        "t_shirts" : {
            "filter" : { "term": { "type": "t-shirt" } },
            "aggs" : {
                "avg_price" : { "avg" : { "field" : "price" } }
            }
        }
    }
}
```
---
```
{
    ...
    "aggregations" : {
        "t_shirts" : {
            "doc_count" : 3,
            "avg_price" : { "value" : 128.33333333333334 }
        }
    }
}
```
- post_filter
>作用于文档过滤，但在聚合分析后生效
```
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "brand": "gucci" } 
      }
    }
  },
  "aggs": {
    "colors": {
      "terms": { "field": "color" } 
    },
    "color_red": {
      "filter": {
        "term": { "color": "red" } 
      },
      "aggs": {
        "models": {
          "terms": { "field": "model" } 
        }
      }
    }
  },
  "post_filter": { 
    "term": { "color": "red" }
  }
}
```
- global
> 无视query过滤条件，基于全部文档进行分析
```
POST /sales/_search?size=0
{
    "query" : {
        "match" : { "type" : "t-shirt" }
    },
    "aggs" : {
        "all_products" : {
            "global" : {}, 
            "aggs" : { 
                "avg_price" : { "avg" : { "field" : "price" } }
            }
        },
        "t_shirts": { "avg" : { "field" : "price" } }
    }
}
```
---
```
{
    ...
    "aggregations" : {
        "all_products" : {
            "doc_count" : 7, 
            "avg_price" : {
                "value" : 140.71428571428572 
            }
        },
        "t_shirts": {
            "value" : 128.33333333333334 
        }
    }
}
```

## 类型
### Bucketing
> 分桶类型，类似SQL中的`GROUP BY`语法，按照一定的规则将文档分配不同的桶中，达到分类分析的目的。与指标聚合不同，分桶聚合可以保存子聚合，即允许通过添加子分析进行进一步分析，子分析可以使`Bucket`，也可以是`Metric`

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
### Metric
> 指标分析类型，如计算最大值、最小值、平均值等等

#### 单值分析
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

#### 多值分析
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

### Pipeline
> 管道分析类型，基于上一级局和分析的结果进行再分析  
`buckets_path`取的是相对路径  
分析结果会输出到原结果中，根据输出结果的位置不同，可以分为以下两类:

#### Parent(父辈，基于histogram)
>结果内嵌到现有的聚合分析结果中
- derivative(求导)
```
#按照年龄对平均工资求导 
POST employees/_search
{
  "size": 0,
  "aggs": {
    "age": {
      "histogram": {
        "field": "age",
        "min_doc_count": 1,
        "interval": 1
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        },
        "derivative_avg_salary":{
          "derivative": {
            "buckets_path": "avg_salary"
          }
        }
      }
    }
  }
}
```
---
```
"aggregations" : {
  "age" : {
    "buckets" : [
      {
        "key" : 20.0,
        "doc_count" : 1,
        "avg_salary" : {
          "value" : 9000.0
        }
      },
      {
        "key" : 21.0,
        "doc_count" : 1,
        "avg_salary" : {
          "value" : 16000.0
        },
        "derivative_avg_salary" : {
          "value" : 7000.0
        }
      }
    ]
  }
}
```
- cumulative_sum(累积求和)
```
POST employees/_search
{
  "size": 0,
  "aggs": {
    "age": {
      "histogram": {
        "field": "age",
        "min_doc_count": 1,
        "interval": 1
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        },
        "cumulative_salary":{
          "cumulative_sum": {
            "buckets_path": "avg_salary"
          }
        }
      }
    }
  }
}
```
- moving_avg(移动平均数)

#### Sibling(同辈)
>结果与现有聚合分析结果同级，常用的有以下聚合函数，和`Metric`类似  

`min_bucket`|`max_bucket`|`avg_bucket`|`sum_bucket`  
`stats_bucket`|`extended_stats_bucket`  
`percentiles_bucket`
```
GET my-index/_search
{
  "size": 0, 
  "aggs": {
    "terms_job": {
      "terms": {
        "field": "job.keyword",
        "size": 5
      },
      "aggs": {
        "avg_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    },
    "min_age_by_job":{
      "min_bucket": {
        "buckets_path": "terms_job>avg_age"
      }
    }
  }
}
```
---
```
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 12,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "terms_job": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 2,
      "buckets": [
        {
          "key": "math chinese",
          "doc_count": 5,
          "avg_age": {
            "value": 28.4
          }
        },
        {
          "key": "math",
          "doc_count": 2,
          "avg_age": {
            "value": 27.5
          }
        },
        {
          "key": "chinese",
          "doc_count": 1,
          "avg_age": {
            "value": 28
          }
        },
        {
          "key": "english",
          "doc_count": 1,
          "avg_age": {
            "value": 45
          }
        },
        {
          "key": "english chinese",
          "doc_count": 1,
          "avg_age": {
            "value": 21
          }
        }
      ]
    },
    "min_age_by_job": {
      "value": 21,
      "keys": [
        "english chinese"
      ]
    }
  }
}
```

### Matrix
> 

## 排序

> 可以使用聚合函数自带的关键数据进行排序
- _count 文档数(默认)
```
GET /_search
{
    "aggs" : {
        "genres" : {
            "terms" : {
                "field" : "genre",
                "order" : { "_count" : "asc" }
            }
        }
    }
}
```
- _key 按照key值排序
```
GET /_search
{
    "aggs" : {
        "genres" : {
            "terms" : {
                "field" : "genre",
                "order" : { "_key" : "asc" }
            }
        }
    }
}
```
> 根据子聚合分析结果进行排序
- 返回单值的子聚合
```
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "term_job": {
      "terms": {
        "field": "job.keyword",
        "size": 10,
        "order": {
          "max_age": "desc"
        }
      },
      "aggs": {
        "max_age": {
          "max": {
            "field": "age"
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
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 12,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "term_job": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "math chinese",
          "doc_count": 5,
          "max_age": {
            "value": 49
          }
        },
        {
          "key": "english",
          "doc_count": 1,
          "max_age": {
            "value": 45
          }
        },
        {
          "key": "math",
          "doc_count": 2,
          "max_age": {
            "value": 34
          }
        },
        {
          "key": "chinese",
          "doc_count": 1,
          "max_age": {
            "value": 28
          }
        },
        {
          "key": "english chinese",
          "doc_count": 1,
          "max_age": {
            "value": 21
          }
        },
        {
          "key": "english chinese math",
          "doc_count": 1,
          "max_age": {
            "value": 21
          }
        },
        {
          "key": "english math",
          "doc_count": 1,
          "max_age": {
            "value": 21
          }
        }
      ]
    }
  }
}
```
- 返回多值的子聚合
```
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "term_job": {
      "terms": {
        "field": "job.keyword",
        "size": 10,
        "order": {
          "filter_by_age>stats_age.min": "desc"
        }
      },
      "aggs": {
        "filter_by_age": {
          "filter": {
            "range": {
              "age": {
                "gte": 20
              }
            }
          },
          "aggs": {
            "stats_age": {
              "stats": {
                "field": "age"
              }
            }
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
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 12,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "term_job": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "english",
          "doc_count": 1,
          "filter_by_age": {
            "doc_count": 1,
            "stats_age": {
              "count": 1,
              "min": 45,
              "max": 45,
              "avg": 45,
              "sum": 45
            }
          }
        },
        {
          "key": "chinese",
          "doc_count": 1,
          "filter_by_age": {
            "doc_count": 1,
            "stats_age": {
              "count": 1,
              "min": 28,
              "max": 28,
              "avg": 28,
              "sum": 28
            }
          }
        },
        {
          "key": "math chinese",
          "doc_count": 5,
          "filter_by_age": {
            "doc_count": 3,
            "stats_age": {
              "count": 3,
              "min": 28,
              "max": 49,
              "avg": 37,
              "sum": 111
            }
          }
        },
        {
          "key": "english chinese",
          "doc_count": 1,
          "filter_by_age": {
            "doc_count": 1,
            "stats_age": {
              "count": 1,
              "min": 21,
              "max": 21,
              "avg": 21,
              "sum": 21
            }
          }
        },
        {
          "key": "english chinese math",
          "doc_count": 1,
          "filter_by_age": {
            "doc_count": 1,
            "stats_age": {
              "count": 1,
              "min": 21,
              "max": 21,
              "avg": 21,
              "sum": 21
            }
          }
        },
        {
          "key": "english math",
          "doc_count": 1,
          "filter_by_age": {
            "doc_count": 1,
            "stats_age": {
              "count": 1,
              "min": 21,
              "max": 21,
              "avg": 21,
              "sum": 21
            }
          }
        },
        {
          "key": "math",
          "doc_count": 2,
          "filter_by_age": {
            "doc_count": 2,
            "stats_age": {
              "count": 2,
              "min": 21,
              "max": 34,
              "avg": 27.5,
              "sum": 55
            }
          }
        }
      ]
    }
  }
}
```