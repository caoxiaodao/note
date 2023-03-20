# URI查询

# RequestBody查询

## 基本使用

- 请求体
  
      curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
      {
          "from" : 0, "size" : 10,//分页限制
          "query" : {//条件体
              "term" : { "user" : "kimchy" }
          },
          "sort":[
           { "post_date" : {"order" : "asc"}},
              "user",
              { "name" : "desc" },
              { "age" : "desc" },
              "_score"
           ],
         "_source": ["obj*"],//source显示字段过滤   
         "stored_fields": "_none_",//stored字段过滤
      }
      '

- 返回值
  
      {
          "took": 1,//花费时间
          "timed_out": false,//请求是否超时
          "_shards":{//分片信息
              "total" : 1,
              "successful" : 1,
              "skipped" : 0,
              "failed" : 0
          },
          "hits":{
              "total" : 1,//总数据条数
              "max_score": 1.3862944,//分数
              "hits" : [//详情信息
                  {
                      "_index" : "twitter",
                      "_type" : "_doc",
                      "_id" : "0",
                      "_score": 1.3862944,
                      "_source" : {
                          "user" : "kimchy",
                          "message": "trying out Elasticsearch",
                          "date" : "2009-11-15T14:12:12",
                          "likes" : 0
                      }
                  }
              ]
          }
      }

## sort详解

### mode

    如果字段值是数组类型，排序可选如下模式min,max，sum，avg,median(中位数)

### missing

_last：不存在值的时候排队到尾部
 _first,不存在值的时候排队到头部

### unmapped_type

 {"unmapped_type" : "long"}类型不是long的不参与排序

### script based sorting

如果想根据两个字段的计算值进行结果排序，手动书写script脚本

    curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
    {
        "query" : {
            "term" : { "user" : "kimchy" }
        },
        "sort" : {
            "_script" : {
                "type" : "number",
                "script" : {
                    "lang": "painless",
                    "source": "doc['field_name'].value * params.factor",
                    "params" : {
                        "factor" : 1.1
                    }
                },
                "order" : "asc"
            }
        }
    }
    '
    
    

### nest Object

    curl -XGET 'localhost:31600/test1/_search' 
    --data '{
    
        "query" : {
          "match" : { "date" :"123456" }
       },
        "sort" : [
            {"number":{
                "order":"asc", "missing":"_last"
            }},
            { "nestObject.beauty" :{
                 "order":"desc",
                 "missing" : "_last",
                "nested":{
                     "path":"nestObject",
                    "filter": {
                       "match" : { "nestObject.name" :"高珊珊" }
                    }
                }
            }}
      ]
    }'
    
    
    返回说明：name是高珊珊或者不是高珊珊的数据都会返回，filter会影响得分，从而影响返回顺序

### 

### track_scores

正常情况下返回如下左图

![](../image/curl-search/2023-03-20-17-42-40-image.png)

配置track_scores=tue返回如图

![](../image/curl-search/2023-03-20-17-44-10-image.png)

## post filter

  聚合之后再进行的条件过滤；例如你有衬衣的如下信息：品牌，颜色，样式；你希望通过一个查询可以得出如下结论：A品牌的红色衬衫的详细信息+A品牌的衬衣流行颜色+A品牌红色衬衣流行样式

    curl -X GET "localhost:9200/shirts/_search?pretty" -H 'Content-Type: application/json' -d'
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
    '
    

## Highlight

-  场景：在百度查询的时候你经常会发现搜索结果当中你的查询内容被高亮了，这就是highlight的作用；

- 工作原理
  
  - 字段文本分段
    
    - plain：游标end_offset超过fragment_size乘以创建的片段数量
    
    - unifined&fast Vector：java的BreakIterator；只要`fragment_size`允许就会是一个有效句子
  
  - 查询最匹配的分段
    
    - plain：标记流中创建内存索引，执行原始查询条件
    
    - fast Vector：预先索引的术语向量；无需创建内存索引
    
    - Unified highlighter：预先索引的术语向量或预先索引的术语偏移可用即使用，不可用创建内存索引
  
  - 给查询词汇高亮：使用用户设置的高亮原则将查询词汇加上html标签

- 配置
  
  - type:类型unifined，plain，fast Vector
  
  - boundary_chars：边界字符（断句字符）：.,!?
  
  - boundary_max_scan：边界字段的距离
  
  - boundary_scanner：边界扫描器
    
    - chars：fvh有效,`boundary_max_scan`控制
    
    - sentence/word：java的BreakIterator；可以设置boundary_scanner_locale
  
  - boundary_scanner_locale
    
    - 使用哪种语言搜索句子边界
  
  - number_of_fragments：返回的最大片段数量
  
  - fragment_size：突出显示片段的大小
  
  - pre_tags：html前标签
  
  - post_tags：html后标签

- 示例

  

## scroll

## searchAfter
