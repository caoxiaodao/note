### 基本原理

#### 乐观并发控制

#### 写入原理

- 写入单个文档
  
  - 客户端向集群的某个节点发起请求，该节点就是协调节点
  
  - 协调节点根据文档id确定文档所在分片，根据集群路由表信息确定主分片位置，发送请求到主分片所在节点
  
  - 执行主分片写操作；写入成功后并行转发至副本分片所在节点，等所有副本分片都写入成功后，发送报告至协调节点，协调节点发送至客户端

- 写入故障处理 TODO
  
  - 主节点故障
    
    - 给主节点发送确定信息，最多等待1分钟，然后选择其中一个副本提升为新的主副本，将请求发送给新的节点进行处理
  
  - 副本节点故障：只有主分片运行，就会有数据丢失的风险

#### 读取原理

- 读取单个文档
  
  - 客户端发送请求至某个节点，该节点就是协调节点
  
  - 协调节点根据文档ID确定文档所在分片
  
  - 通过集群状态路由表信息确定副本信息，请求转发至副本分片所在节点读取信息（多个副本分片客户端轮询来负载均衡）
  
  - 副本分片节点返回信息至协调节点，然后返回至客户端

- 读取多个文档
  
  - query阶段
    
    - 客户端向集群的某个节点发起请求，该节点就是协调节点
    
    - 协调节点将查询转发至索引的每一个主分片或者副本分片中
    
    - 每个分片本地执行查询，使用本地的term，document frequency信息进行发奋，结果添加到ftrom+size的本地有序优先队列中
    
    - 每个分片返回各自队列的所有文档id和排序值给协调节点，协调节点合并至自己的优先队列中，产生一个全局排序后的列表
  
  - fetch阶段
    
    - 协调节点只要了要获取那些信息，但是还没有具体的数据，需要执行多个get请求获取数据
    
    - 分片所在节点向协调节点返回数据
    
    - 所有数据返回完成，然会给客户端

### 写入文档

```
// 带有id
curl -X PUT "localhost:9200/twitter/_doc/1?op_type=create&pretty" -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
h"
}
//自动生成id
curl -X POST "localhost:9200/twitter/_doc/?pretty" -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

- op_type=create文档id存在则报错；当没有该设置时文档存在则更新，不存在则新增

- 路由配置
  
  ```
   curl -X POST "localhost:9200/twitter/_doc?routing=kimchy&pretty" -H 'Content-Type: application/json' -d'
  {
      "user" : "kimchy",
      "post_date" : "2009-11-15T14:12:12",
      "message" : "trying out Elasticsearch"
  }
  '
  ```
  
  - 分片获取方式`hash(routing)%分片数量`，es默认使用_id为routing确定分片位置，然后进行写入和查询
  
  - 指定routing后搜索效率会增加，但是会导致同一分片上面_id不唯一和索引数据分布不均的问题
  
  - `index.routing_partition_size`设置索引根据routing路由shard的数量，可解决一部分数据分布不均问题

- 等待活动分片：wait_for_active_shards，等待多少分片在线才可以执行写入操作；如果我们有1个主分片，2个副本，则可以设置为1-3任意数字

- timeout：timeout=5m，主分片如果不可用，等待多久返回执行结果，默认是1分钟

- 版本控制
  
  - version=2：默认使用内部版本控制，但是可以采用数据库储存版本然后使用外部版本控制
  
  - version_type=external：新版本大于存储版本可以成功更新，否则报错
  
  - version_type=external_gte新版本等于或者大于存储版本可以成功更新，否则报错

- 返回值
  
  ```
  {
      "_shards" : {
          "total" : 2,//碎片数量（1个主碎片+1个副本碎片）
          "failed" : 0,//失败数量
          "successful" : 2 //成功数量
      },
      "_index" : "twitter",//索引名称
      "_type" : "_doc",//类型
      "_id" : "1",//id
      "_version" : 1,//版本号
      "_seq_no" : 0,
      "_primary_term" : 1,
      "result" : "created"//新增，（也可能是updated）
  }
  ```

### 更新文档

#### 单个更新

- 基本用法
  
  - ```
    curl -X POST "localhost:9200/test/_doc/1/_update?pretty" -H 'Content-Type: application/json' -d'
    {
        "doc" : {
            "name" : "new_name",/原来不存在该字段
            "tag" : "blue"//原来存在该字段
        },
         "detect_noop": false//不管字段有没有更新都强制执行更新
    //docid存在则更新，不存在则新增；如果没有该字段docid不存在，则会报错
         "doc_as_upsert" : true
    }
    '
    ```

- 脚本更新
  
  - ```
    curl -X POST "localhost:9200/test/_doc/1/_update?pretty" -H 'Content-Type: application/json' -d'
    {
        "script" : {
            "source": "if (ctx._source.tags.contains(params.tag)) { ctx.op = \u0027delete\u0027 } else { ctx.op = \u0027none\u0027 }",
            "lang": "painless",
            "params" : {
                "tag" : "green"
            }
        }
    }
    '
    ```

- upserts
  
  - ```
    curl -X POST "localhost:9200/sessions/session/dh3sgudg8gsrgl/_update?pretty" -H 'Content-Type: application/json' -d'
    {
        //false ：存在则执行script不存在则执行upset
        // true : 存在执行script不存在执行以upsert为基础执行script
        "scripted_upsert":true,
        "script" : {
            "id": "my_web_session_summariser",
            "params" : {
                "pageViewEvent" : {
                    "url":"foo.com/bar",
                    "response":404,
                    "time":"2014-01-01 12:32"
                }
            }
        },
        "upsert" : {
                  "counter" : 1
          }
    }
    '
    ```

#### 批量更新

```
curl -X POST "localhost:9200/twitter/_update_by_query?pretty" -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
'
```

- 如果索引"dynamic": false，添加数据的时候没有对应prop数据，后面更新prop数据之后，该字段依旧只是存储，无法查询，执行`POST test/_update_by_query?refresh&conflicts=proceed` 就可以查询了

- routing，conflicts，wait_for_active_shards

- scroll_size：1000:批量更新是每批次多少条数据

- scroll=10m：scroll查询保存的时间

- requests_per_second：500 每秒执行多少条数据；
  
  - 删除或者更新大数据的时候放慢执行速度以免es会挂掉
  
  - 如果每批次1000条，每秒500个request，那么执行完一批需要1000/500=2s的时间，写入1000条数据0.5s的时间，就可以休息1.5s的时间

- slice
  
  ```
  // 手动分片
  POST twitter/_update_by_query
  {
    "slice": {
      "id": 0,
      "max": 2
    },
    "script": {
      "source": "ctx._source['extra'] = 'test'"
    }
  }  
  //自动分片             
  POST twitter/_update_by_query?refresh&slices=5
  {
    "script": {
      "source": "ctx._source['extra'] = 'test'"
    }
  }
  ```
  
  - 并行执行，执行效率变快
  
  - 返回值
    
    ```
    {
        "took": 214,
        "timed_out": false,
        "total": 4,
        "updated": 4,
        "deleted": 0,
        "batches": 3,
        "version_conflicts": 0,
        "noops": 0,
        "retries": {
            "bulk": 0,
            "search": 0
        },
        "throttled_millis": 0,
        "requests_per_second": -1.0,
        "throttled_until_millis": 0,
        "slices": [
            {
                "slice_id": 0,
                "total": 1,
                "updated": 1,
                "deleted": 0,
                "batches": 1,
                "version_conflicts": 0,
                "noops": 0,
                "retries": {
                    "bulk": 0,
                    "search": 0
                },
                "throttled_millis": 0,
                "requests_per_second": -1.0,
                "throttled_until_millis": 0
            },
            {
                "slice_id": 1,
                "total": 0,
                "updated": 0,
                "deleted": 0,
                "batches": 0,
                "version_conflicts": 0,
                "noops": 0,
                "retries": {
                    "bulk": 0,
                    "search": 0
                },
                "throttled_millis": 0,
                "requests_per_second": -1.0,
                "throttled_until_millis": 0
            },
            {
                "slice_id": 2,
                "total": 2,
                "updated": 2,
                "deleted": 0,
                "batches": 1,
                "version_conflicts": 0,
                "noops": 0,
                "retries": {
                    "bulk": 0,
                    "search": 0
                },
                "throttled_millis": 0,
                "requests_per_second": -1.0,
                "throttled_until_millis": 0
            },
            {
                "slice_id": 3,
                "total": 1,
                "updated": 1,
                "deleted": 0,
                "batches": 1,
                "version_conflicts": 0,
                "noops": 0,
                "retries": {
                    "bulk": 0,
                    "search": 0
                },
                "throttled_millis": 0,
                "requests_per_second": -1.0,
                "throttled_until_millis": 0
            },
            {
                "slice_id": 4,
                "total": 0,
                "updated": 0,
                "deleted": 0,
                "batches": 0,
                "version_conflicts": 0,
                "noops": 0,
                "retries": {
                    "bulk": 0,
                    "search": 0
                },
                "throttled_millis": 0,
                "requests_per_second": -1.0,
                "throttled_until_millis": 0
            }
        ],
        "failures": []
    }
    ```

### 删除文档

#### 根据id删除文档

```
curl -X DELETE "localhost:9200/twitter/_doc/1?pretty"
// 如果分片不可用，等待多久时间
curl -X DELETE "localhost:9200/twitter/_doc/1?timeout=5m&pretty"
```

- routing

- wait_for_active_shards

- refresh

#### 根据查询条件删除文档

```
curl -X POST "localhost:9200/twitter/_delete_by_query?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { 
    "match": {
      "message": "some message"
    }
  }
}
// conflicts=proceed版本冲突会继续删除
'
curl -X POST "localhost:9200/twitter/_doc/_delete_by_query?conflicts=proceed&pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}
'
```

- 依次执行多个搜索请求，找到所有匹配文档并删除，每找到一批文档就会执行一批的删除操作；如果某些批量操作被拒绝（重试10次），已删除的数据不会回滚

- routing=1

- scroll_size=5000（每批数据的大小）

- refresh：刷新所有涉及删除的分片，和delete api不同只删除收到删除请求的分片

- wait_for_active_shards：等待多少分片在线才可以执行删除操作

- wait_for_completion

- scroll：scroll查询可以保存的时间，不是全量处理数据的时间，是处理本次请求中数据的时间即可

- requests_per_second：每秒执行的请求数量，控制处理速率的

- pretty、refresh、timeout

### 文档读操作

###### get

```
curl -X GET "localhost:9200/twitter/_doc/0?pretty"
// head查询是否存在
curl -I "localhost:9200/twitter/_doc/0?pretty"
```

- source字段
  
  ```
  curl -X GET "localhost:9200/twitter/_doc/0?_source_includes=*.id&_source_excludes=entities&pretty"
  ```

- 只获取source字段
  
  ```
  curl -X GET "localhost:9200/twitter/_doc/1/_source?_source_includes=*.id&_source_excludes=entities&pretty"
  ```

- stored字段
  
  ```
  curl -X GET "localhost:9200/twitter/_doc/1?stored_fields=tags,counter&pretty"
  ```
  
  - store字段会占用另外的磁盘空间，但是store之后的字段聚合和搜索性能都更好，如果source字段过多的时候，需要特别使用的字段可以设置为store

- refresh：慎用，因为会刷新相关碎片之后再执行查询

- routing：只查询特定routing上面的文档数据

- 副本数量越多，get性能越好 ODO

###### mget

```
curl -X GET "localhost:9200/test/_doc/_mget?pretty" -H 'Content-Type: application/json' -d'
{
    "ids" : ["1", "2"]
}
'
```

### bulk

详见 [Elasticsearch(037)：es中批量操作之bulk_es bulk_瘦子没有夏天的博客-CSDN博客](https://blog.csdn.net/weixin_39723544/article/details/109237175)

```
curl -X POST "localhost:9200/_bulk?pretty" -H 'Content-Type: application/json' -d'
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }//新增
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }//删除
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }//新增
{ "field1" : "value3" }//上一步的请求体
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }//更新
{ "doc" : {"field2" : "value2"} }//上一步的请求体
'
```

### reindex

```
curl -X POST "localhost:9200/_reindex?pretty" -H 'Content-Type: application/json' -d'
{
 "conflicts": "proceed",//报版本冲突之后继续处理，最终返回版本冲突数据
  "source": {
    "index": "twitter",
     "type": "_doc",
    "size": 10000,
    "sort": { "date": "desc" }
     "query": {
        "term": {
         "user": "kimchy"
       }
      }
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "internal",
     "op_type": "create".//目标索引数据已存在则报版本冲突
  },
 "script": {
    "source": "if (ctx._source.foo == 'bar') {ctx._version++; ctx._source.remove('foo')}",
    "lang": "painless"
  }
}
'
```

- 只是复制数据，不会同步setting，mapping的设置

- version_type
  
  - external:只同步不存在数据和源数据版本大于目标索引的数据
  
  - internal:不管版本如何都会将源索引覆盖掉目标索引

- 可以从跨es集群进行操作详见 [Reindex API | Elasticsearch Guide [6.8] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/docs-reindex.html)

## 集群操作
