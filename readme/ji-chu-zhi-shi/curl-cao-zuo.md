# curl操作

## 通用语句

- 多个索引共同搜索
  
  - `test* 或者*test 或者test*t 或者 *test*或者-test`   (-)是排除

- 日志格式支持
  
  - <静态文本{动态的时间表达式{日期呈现格式|时区}}>
    
    - 最近三天 
    
    ```
    # GET /<logstash-{now/d-2d}>,<logstash-{now/d-1d}>,<logstash-{now/d}>/_search
    GET /%3Clogstash-%7Bnow%2Fd-2d%7D%3E%2C%3Clogstash-%7Bnow%2Fd-1d%7D%3E%2C%3Clogstash-%7Bnow%2Fd%7D%3E/_search
    {
      "query" : {
        "match": {
          "test": "data"
        }
      }
    }
    ```

- 格式化结果
  
  - ?pretty=true
  
  - ?format=yaml
  
  - ?human=false

- 过滤显示结果的字段
  
  - 示例 curl -X GET "localhost:9200/_search?q=elasticsearch&filter_path=took,-_shards，hits.hits._id,hits.hits._score&pretty"

## 通用配置

### action.auto_create_index

 全局配置，新增文档的时候，如果索引不存在是报错还是自动创建

### action.destructive_requires_name

通配符启用

### indices.requests.cache.size

全局配置,请求缓存,indices.requests.cache.size: 1%一般是heap的1%

### index.requests.cache.enable

索引级别，请求缓存是否使用 true/false

### search.default_allow_partial_results

全局配置,true:在超时或者部分失败的情况下产生部分结果；false：无法产生部分结果，直接返回失败

## 通用参数

### search_type

es有很多shard，每一个shard是一个lucene的索引，保存了自身的TF和DF统计信息。一个shard只知道自身的出现次数，不是整个cluster；但是ES维护了全局的TF/DF统计信息

- dfs_query_then_fetch
  
  - 不常用，精准效率低
  
  - 使用全局的TF/DF信息进行打分

- query_then_fetch 
  
  - 默认，常用，不准确，效率高
  
  - 使用自身的TF/DF信息进行打分

### request_cache

- true或者false

- shard/节点级别的缓存

- 只缓存size=0的数据（缓存hits.total, aggregations, suggestions不缓存hits）

- 缓存键使用整个请求jsonbody，索引json有不一样缓存就无法使用

- 分片数据实际发生变化时失效，缓存满了之后使用最少的缓存键被删除；

- 刷新时间越长，缓存时间越长

### allow_partial_search_results

参见search.default_allow_partial_results

### batched_reduce_size

默认情况下，聚合操作在协调节点需要等所有的分片都取回结果后才执行，使用`batched_reduce_size`参数可以不等待全部分片返回结果，而是在指定数量的分片返回结果之后就可以先处理一部分(`reduce`)。 这样可以避免协调节点在等待全部结果的过程中占用大量内存，避免极端情况下可能导致的OOM。**该字段的默认值为512，从ES 5.4开始支持。

### terminate_after

每个分片要收集的最大文件数，达到这个数字后，查询的执行将提前终止。如果设置了，响应将有一个布尔字段 terminated_early 来表示查询的执行是否真的终止了。默认情况下，没有terminate_after。

## 数据类型

### 字符串

- text
  
  - 会分词后进行索引
  
  - 支持模糊，精确查询
  
  - 不支持聚合

- keyword
  
  - 不进行分词，直接索引
  
  - 支持模糊，精确查询
  
  - 支持聚合

- 混用:可以使用name字段进行模糊查询和分词，name.keyword进行聚合
  
  - ```
    curl --location --request PUT 'localhost:31600/testmapping3' \
    --header 'Content-Type: application/json' \
    --data '{
      "mappings": {
        "_doc": {
          "properties": {
            "name": {
              "type": "text",
              "fields":{
                  "keyword":{
                      "type":"keyword"
                  }
              }
            }
          }
        }
      }
    }'
    ```

### 数字

- long，integer，short，byte

- double，float，half_float,scaled_float

### 日期

- ```
  curl -X PUT "localhost:9200/my_index?pretty" -H 'Content-Type: application/json' -d'
  {
    "mappings": {
      "_doc": {
        "properties": {
          "date": {
            "type":   "date",
            "format": "strict_date_optional_time||yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
          }
        }
      }
    }
  }
  '
  ```

- strict_date_optional_time严格的年月日格式：仅支持"yyyy-MM-dd"、"yyyyMMdd"、"yyyyMMddHHmmss"、"yyyy-MM-ddTHH:mm:ss"、"yyyy-MM-ddTHH:mm:ss.SSS"、"yyyy-MM-ddTHH:mm:ss.SSSZ"格式
  
  不支持常用的"yyyy-MM-dd HH:mm:ss"等格式。注意，"T"和"Z"是固定的字符

- epoch_millis时间的ms值

- Boolean布尔：true，“true”

- 二进制
  
  - 可以存储base64编码字符串，默认不支持检索和store
    
    - store就是独立于source之外单独存储一份数据，如果数据量太大，可以store存储，source不在存储，获取时指定只获取source可以减少数据量
    
    - [doc_values]可以指定是否要支持检索

### 区间类型

- integer_range,long__range,float_range,double_range,date_range
  
  - ```
    -- 创建索引
    curl -X PUT "localhost:9200/range_index?pretty" -H 'Content-Type: application/json' -d'
    {
      "settings": {
        "number_of_shards": 2
      },
      "mappings": {
        "_doc": {
          "properties": {
            "expected_attendees": {
              "type": "integer_range"
            },
            "time_frame": {
              "type": "date_range", 
              "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
            }
          }
        }
      }
    }
    '
    -- 插入数据
    curl -X PUT "localhost:9200/range_index/_doc/1?refresh&pretty" -H 'Content-Type: application/json' -d'
    {
      "expected_attendees" : { 
        "gte" : 10,
        "lte" : 20
      },
      "time_frame" : { 
        "gte" : "2015-10-31 12:00:00", 
        "lte" : "2015-11-01"
      }
    }
    '
    
    -- 查询
    curl --location --request GET 'localhost:31600/testmapping3/_doc/_search' \
    --header 'Content-Type: application/json' \
    --data '{
      "query": {
        "range": {
          "expected_attendees": {
            "gte":"1",
            "lte":"11",
             "relation" : "CONTAINS" 
          }
        }
      }
    }'
    ```
  
  - relation关系
    
    - intersects查询和文档存 储范围有交叉即可（默认）
    
    - within 文档范围在查询范围内
    
    - contains 文档范围包含查询范围

### 复杂类型

- Array数组
  
  - 设置mapping的时候无需指定arrays类型，但是long，text等都支持多值，但是必须类型一样，或者是可以进行强制类型转换

- Object对象

- Nested类型：一种可以让层级对应各自索引查询的object对象，和object的不同见
  
  详细见：[Nested datatype | Elasticsearch Guide [6.8] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/nested.html)[Nested datatype | Elasticsearch Guide [6.8] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/nested.html)

### 特定类型

- GEO地理位置
  
  - Geo-point：一个经纬度确定的点
    
    - 字符串是纬度，经度；数组是经度，纬度
    
    - ```
      // 1）geo_bounding_box_shape经纬确定的矩形
      GET location/_search
      {
        "query": {
          "geo_bounding_box": {
            "locationStr": {
              "top_left": [116.498353, 40.187328],
              "bottom_right": [116.610461, 40.084509]
            }
          }
        }
      }
      //2)geo_distance 选中中心点和半径进行查询
      GET location/_search
      {
        "query": {
          "geo_distance": {
            "distance": "5km",
            "locationStr": "40.174697,116.5864"
          }
        },
        "sort": [
          {
            "_geo_distance": {
              "locationStr": "40.174697,116.5864",
              "order": "asc",
              "unit": "km",
              "distance_type": "plane"
            }
          }
        ]
      }
      //返回
      {
          "_index" : "location",
          "_type" : "_doc",
          "_id" : "2",
          "_score" : null,
          "_source" : {
            "locationStr" : "40.19103839805197,116.5624013764374"
          },
          "sort" : [
            2.7309695122568725 //距离目标地点的直线距离
          ]
        }
      
      　//　3）geo_polygon多边形查询
      GET location/_search
      {
        "query": {
          "geo_polygon": {
            "locationStr": {
              "points": [
                "40.178012,116.577188",
                "40.169329, 116.586315",
                "40.178288, 116.591813"
              ]
            }
          }
        }
      }
      
      　　4）geo_shape查询（查询两个地理位置形状的位置关系）
      GET /my_index_geo_shape/_search
      {
          "query":{
              "bool": {
                  "must": {
                      "match_all": {}
                  },
                  "filter": {
                      "geo_shape": {
                          "location": {
                              "shape": {
                                  "type": "envelope",
                                  "coordinates" : [[116.327854,39.90075], [117.065057,39.063287]]
                              },
                              "relation": "within"
                          }
                      }
                  }
              }
          }
      }
      INTERSECTS - 和查询位置橡胶的存储位置。
      DISJOINT - 和查询位置完全无交集的存储位置
      WITHIN - 查询几何图形包含的所有存储集合
      CONTAINS - 存储集合包含查询几何图形的所有文档。
      ```

```
    ```

- Geo-shape：一个经纬度确定的点，一个中心点+半径确定的圆，多个点确定的多边形
```

- IP类型
  
  - 支持ipv4和ipv6类型
  
  - ```
    curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
    {
      "query": {
        "term": {
          "ip_addr": "192.168.0.0/16"
        }
      }
    }
    '
    CIDR 表示法: [ip_address]/[prefix_l
    ```

- completion自动补全类型

- token_count令牌计数类型：是一个int类型，接受字符串值，然后索引字符串中标记数
  
  - ```
    curl -X PUT "localhost:9200/my_index?pretty" -H 'Content-Type: application/json' -d'
    {
      "mappings": {
        "_doc": {
          "properties": {
            "name": { 
              "type": "text",
              "fields": {
                "length": { 
                  "type":     "token_count",
                  "analyzer": "standard"
                }
              }
            }
          }
        }
      }
    }
    '
    curl -X PUT "localhost:9200/my_index/_doc/1?pretty" -H 'Content-Type: application/json' -d'
    { "name": "John Smith" }
    '
    curl -X PUT "localhost:9200/my_index/_doc/2?pretty" -H 'Content-Type: application/json' -d'
    { "name": "Rachel Alice Williams" }
    '
    curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
    {
      "query": {
        "term": {
          "name.length": 3 
        }
      }
    }
    '
    返回Rachel Alice Williams
    ```

- percolator父子索引：//TODO

- alias别名

# 

## cat查询

#### 

- op_type=create文档id存在则报错；当没有该设置时文档存在则更新，不存在则新增
  
  ```
  curl -X PUT "localhost:9200/twitter/_doc/1?op_type=create&pretty" -H 'Content-Type: application/json' -d'
  {
      "user" : "kimchy",
      "post_date" : "2009-11-15T14:12:12",
      "message" : "trying out Elasticsearch"
  }
  '
  h"
  }
  ```

- 

- 等待活动分片：索引请求返回前需要等待多少个分片写入成功，
  
  - 最小值1（默认，主分片）
  
  - 最大值 副本数量+1

- 索引自增
  
  - TODO
