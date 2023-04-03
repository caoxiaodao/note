# 创建索引

- 请求体
  
  ```
  curl -X PUT "localhost:9200/test?pretty" -H 'Content-Type: application/json' -d'
  {
      "settings" : {
           "number_of_shards" : 3,//主分片
          "number_of_replicas" : 2//副本数量
      },
      "mappings" : {
          "_doc" : {//7.0以后没有type的概念了，所以这个可以不用写了
              "properties" : {
                  "field1" : { "type" : "text" }//字段和类型
              }
          }
      },
     "aliases" : {//索引别名
          "alias_1" : {},
          "alias_2" : {
              "filter" : {
                  "term" : {"user" : "kimchy" }
              },
              "routing" : "kimchy"
          }
      }
  }
  '
  ```
  
  - setting设置详解见 [Index Modules | Elasticsearch Guide [6.8] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/index-modules.html) TODO
  
  - mapping详情可参见数据类型
  
  - aliases TODO

- 返回体
  
  - acknoledged：索引是否创建成功；为false不一定是失败了，因为可能只是请求超时了，超时之后可能依旧成功了
  
  - shards_acknowledged：每个主分片是否有足够的副本复制；为false不一定是失败了，因为可能只是请求超时了，超时之后可能依旧成功了

# 删除索引

- 请求体

```
curl -X DELETE "localhost:9200/twitter?pretty"
```

- 返回体

```
{"acknowledged": true}
```

# 更新索引

## open/close索引

- 请求体
  
  ```
  curl -X POST "localhost:9200/my_index/_close?ignore_unavailable=true&pretty"
  curl -X POST "localhost:9200/my_index/_open?pretty"
  ```
  
  - ignore_unavailable=true：索引不存在也不报错
  
  - **close的索引不占用内存资源，占用磁盘资源；但是不要长时间保留close的索引，因为在节点进行扩容删除的时候close索引的数据是不会重新分配的，会导致数据丢失**

- 返回体：参见创建索引

## shrink/split索引

- shrink索引
  
  某类数据量很小，但是分片数量比较庞大，降低集群整体的性能
  
  - 操作条件
    
    - 目标索引不存在
    
    - 源索引比目标索引有更多的主分片，目标索引是源索引的一个系数，质数只能压缩为1个主分片
    
    - 所有分片的包含文档数量不超过2,147,483,519
    
    - 磁盘空间足够
  
  - 操作步骤
    
    ```
    curl -X PUT "localhost:9200/my_source_index/_settings?pretty" -H 'Content-Type: application/json' -d'
    {
      "settings": {
    //重新定位索引分片到某个节点（所有分片需要定位到同一个节点-同一个硬盘）
        "index.routing.allocation.require._name": "shrink_node_name", 
        "index.blocks.write": true //禁止写操作
      }
    }
    '
    ```
    
    ```
    url -X POST "localhost:9200/my_source_index/_shrink/my_target_index?copy_settings=true&pretty" -H 'Content-Type: application/json' -d'
    
    {
      "settings": {
        "index.routing.allocation.require._name": null,//清除定位要求
        "index.blocks.write": null,//清除禁止写操作同步
        "index.number_of_replicas": 1,//目标副本
        "index.number_of_shards": 1,//目标主分片
        "index.codec": "best_compression"
      },
      "aliases": {//别名
        "test4_new_alias": {}
      }
    
    }
    ```

- split
  
  初期因为对数据量评估不足，导致分片数量太小，单分片数据量太大，索引能力太低
  
  - 操作条件
    
    - 目标索引不存在
    
    - 索引的主分片比目标索引的主分片少，且目标索引的主分片数量是原索引主分片数量的倍数
    
    - number_of_routing_shards最大能扩充的数量，设置才可以扩充，乘系数扩充
    
    - 有足够的磁盘空间容纳现有索引的第二个副本
  
  - 操作步骤
    
    - ```
      curl -X PUT 'localhost:31600/twitter4_new/_settings' 
      -H 'Content-Type: application/json' 
      -d '
      {
        "settings": {
          "index.blocks.write": true
        }
      }'
      ```
    
    - ```
      curl -X POST 'localhost:31600/ti_v0/_split/ti_v0_spilut' \
      --header 'Content-Type: application/json' \
      --data '
      {
        "settings":{
          "index.number_of_shards":10
        }
      
      }'
      ```

## mapping相关

- 查询
  
  ```
  curl -X GET "localhost:9200/{索引名称}/_mapping/{类型名称}?pretty"
  curl -X GET "localhost:9200/{索引名称}/_mapping/{类型名称}/field/{字段名称}?pretty"
  ```
  
  - 可以使用通配符

- 修改
  
  ```
  curl -X PUT "localhost:9200/twitter/_mapping/_doc?pretty" -H 'Content-Type: application/json' -d'
  {
    "properties": {
      "email": {
        "type": "keyword"
      }
    }
  }
  '
  ```
  
  - 可以增加已存在索引mapping的字段
  
  - 已存在字段不可修改，例外情况如下：object类型扩充，属性扩展；详见[Mapping parameters | Elasticsearch Guide [6.8] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/mapping-params.html)

## settings相关

- 查询
  
  ```
  curl -X GET "localhost:9200/twitter/_settings/{setting的名称}"
  ```

- 更新
  
  ```
  curl -X PUT "localhost:9200/twitter/_settings?pretty" -H 'Content-Type: application/json' -d'
  {
      "index" : {
          "refresh_interval" : "-1"
      }
  }
  '
  ```
  
  - 批量操作时更新refresh的间隔
  
  - 增加分词器
  
  - 增加副本等等

# 查询索引

## 查询是否存在

`curl -I "localhost:9200/twitter?pretty"` 200表示索引存在，404表示索引不存在

## 查询详细信息

- 请求体
  
  ```
  curl -X GET "localhost:9200/twitter?pretty"
  ```
  
  - 可以使用通配符
  
  - include_type_name=false 返回去除type（7.0以后无类型说法）

- 返回体：参考创建索引

## flush&refresh

- 数据写入buffer缓存区和translog日志当中

- buffer满了或者间隔1s，讲buffer中的内容写入文件系统cache中，然后清空buffer继续使用（refresh）

- translog达到一定条数或者30min之后，段内存数据刷入磁盘，translog被清空，创建新的translog

- segment每refresh一次都会产生新的，segement过多会导致索引过慢，所以es会对segment进行合并，合并过程中会将原本.del的记录进行真正的删除操作
  
  ```
  //flush
  curl -X POST "localhost:9200/twitter/_flush?pretty"
  //synced flush
  curl -X POST "localhost:9200/twitter/_flush/synced?pretty"
  // refresh
  curl -X POST "localhost:9200/twitter/_refresh?pretty"
  //merge
  curl -X POST "localhost:9200/twitter/_forcemerge?pretty"
  ```

# 索引恢复

```
curl -X GET "localhost:9200/{索引}/_recovery?human&detailed=true&pretty"
```

索引恢复是es数据恢复的过程，master节点挂掉，集群重新都会触发索引恢复过程；主要处理客户端写入成功，但是为进行flush的lucene分段

- 主分片恢复流程：从translog中自我重建segment恢复，
  
  - init:恢复尚未启动，从恢复那刻起被标记为init阶段
  
  - index：从lucene读取最后一次提交的分段信息，版本号；更新当前索引版本
  
  - verify_index：索引验证（不执行，数据量太大会很耗时）
  
  - translog：根据最后一次提交信息快照，确定那些数据需要重放；重放完毕后讲新的lucene数据刷入磁盘
  
  - finalize：执行refresh操作，缓冲数据写入文件，不刷盘，数据保存再cache中
  
  - done：之前再次执行refresh，然后更新分片状态

- 副本分片恢复流程
  
  - - init:本阶段是在副本节点执行的，其把恢复任务开始时设置为INIT阶段，其副本准备向主分片节点发送StartRecoveryRequest的请求，其请求中包含本次要恢复的shard相关信息，如shardId等
    
    - index：主要负责将分片的Lucene数据复制到副分片中（中间不阻塞索引请求，所以后面需要重放translog）
      
      - 如果可以基于恢复请求中的SequenceNumber进行恢复，则跳过该阶段
      
      - 如果主副分片有相同的synid且doc数量相同，则跳过该阶段（synced flush）
    
    - verify_index：索引验证（不执行，数据量太大会很耗时）
    
    - translog：主要是讲主分片的translog数据发送到副分片节点进行重放
    
    - finalize，done：参加主副本

# 状态查询 TODO

```
请求
curl -X GET "localhost:9200/{索引名称}/_stats?pretty"
返回 详见https://blog.csdn.net/qq_27818541/article/details/112909576
{
    "_shards": {
        "total": 10,//分片总数
        "successful": 5,//成功返回数
        "failed": 0//失败返回数
    },
    "_all": {
        "primaries": {//主分片统计信息
            "docs": {
                "count": 0,
                "deleted": 0
            },
            "store": {
                "size_in_bytes": 1305//存储数据所占字节
            },
            "indexing": {
                "index_total": 0,//索引总次数
                "index_time_in_millis": 0,//索引操作总耗时
                "index_current": 0,//当前正在执行索引操作的个数
                "index_failed": 0,//失败的索引操作次数
                "delete_total": 0,// 执行删除索引操作的次数
                "delete_time_in_millis": 0,        // 删除索引操作总耗时
                "delete_current": 0,// 当前正在执行删除索引操作的个数
                "noop_update_total": 0,// 空更新总次数（检测到空更新的次数
                "is_throttled": false,// 索引是否处在merge throttling control（合并节流控制状态）
                "throttle_time_in_millis": 0 // 索引处在merge throttling control(合并节流状态)的时间开销
            },
            "get": {
                "total": 0, //  get api总调用次数
                "time_in_millis": 0, // get api总耗时。
                "exists_total": 0, // 命中的次数
                "exists_time_in_millis": 0,  // 命中的操作总耗时
                "missing_total": 0,// 未命中的总次数
                "missing_time_in_millis": 0,
                "current": 0// 当前正在执行的个数
            },
            "search": {
                "open_contexts": 0, // 正在打开的查询上下文个数
                "query_total": 15,// 查询出来的总数据条数
                "query_time_in_millis": 0,// 查询阶段所耗费的时间
                "query_current": 0,// 当前正在查询个数
                "fetch_total": 0, // Fetch操作的次数
                "fetch_time_in_millis": 0,
                "fetch_current": 0,  // 正在fetch的次数
                "scroll_total": 0,  // 通过scroll api查询数据总条数
                "scroll_time_in_millis": 0,
                "scroll_current": 0,
                "suggest_total": 0,  // 通过suggest api获取的推荐总数量
                "suggest_time_in_millis": 0,
                "suggest_current": 0
            },
            "merges": {
                "current": 0,
                "current_docs": 0,
                "current_size_in_bytes": 0,
                "total": 0,
                "total_time_in_millis": 0,
                "total_docs": 0,
                "total_size_in_bytes": 0,
                "total_stopped_time_in_millis": 0,
                "total_throttled_time_in_millis": 0,
                "total_auto_throttle_in_bytes": 104857600
            },
            "refresh": {
                "total": 15,
                "total_time_in_millis": 0,
                "listeners": 0
            },
            "flush": {
                "total": 5,
                "periodic": 0,
                "total_time_in_millis": 0
            },
            "warmer": {
                "current": 0,
                "total": 5,
                "total_time_in_millis": 0
            },
            "query_cache": {
                "memory_size_in_bytes": 0,
                "total_count": 0,
                "hit_count": 0,
                "miss_count": 0,
                "cache_size": 0,
                "cache_count": 0,
                "evictions": 0
            },
            "fielddata": {
                "memory_size_in_bytes": 0,
                "evictions": 0
            },
            "completion": {
                "size_in_bytes": 0
            },
            "segments": {
                "count": 0, // 该索引目前拥有的总段数
                "memory_in_bytes": 0,// 该索引缓存在内存中字节数
                "terms_memory_in_bytes": 0, // 倒排索引(term)缓存在内中所占字节数
                "stored_fields_memory_in_bytes": 0, // 该索引定义为stored_fields字段在内存中缓存的字节数
                "term_vectors_memory_in_bytes": 0,// 该索引term_vectors(词向量)在内存中所占字节数量
                "norms_memory_in_bytes": 0, // 该索引存储对应norms=true的字段当前在内存中缓存字节数
                "points_memory_in_bytes": 0,  // 与地理位置相关的缓存数据
                "doc_values_memory_in_bytes": 0,  // 设置为doc_values缓存在内存中的字节数（doc_values,列式存储）
                "index_writer_memory_in_bytes": 0,  // 用于优化索引写的缓存（减少写磁盘的频率
                "version_map_memory_in_bytes": 0,   // 关于文档的版本映射所占内存大小
                "fixed_bit_set_memory_in_bytes": 0, // fixed_bit_set内存，专门用来做nested查询的
                "max_unsafe_auto_id_timestamp": -1, // fixed_bit_set内存，专门用来做nested查询的
                "file_sizes": {}
            },
            "translog": {
                "operations": 0,
                "size_in_bytes": 1375,
                "uncommitted_operations": 0,
                "uncommitted_size_in_bytes": 1375,
                "earliest_last_modified_age": 0
            },
            "request_cache": {
                "memory_size_in_bytes": 0,
                "evictions": 0,
                "hit_count": 0,
                "miss_count": 0
            },
            "recovery": {
                "current_as_source": 0,
                "current_as_target": 0,
                "throttle_time_in_millis": 0
            }
        },
        "total": {
            "docs": {
                "count": 0,
                "deleted": 0
            },
            "store": {
                "size_in_bytes": 1305
            },
            "indexing": {
                "index_total": 0,
                "index_time_in_millis": 0,
                "index_current": 0,
                "index_failed": 0,
                "delete_total": 0,
                "delete_time_in_millis": 0,
                "delete_current": 0,
                "noop_update_total": 0,
                "is_throttled": false,
                "throttle_time_in_millis": 0
            },
            "get": {
                "total": 0,
                "time_in_millis": 0,
                "exists_total": 0,
                "exists_time_in_millis": 0,
                "missing_total": 0,
                "missing_time_in_millis": 0,
                "current": 0
            },
            "search": {
                "open_contexts": 0,
                "query_total": 15,
                "query_time_in_millis": 0,
                "query_current": 0,
                "fetch_total": 0,
                "fetch_time_in_millis": 0,
                "fetch_current": 0,
                "scroll_total": 0,
                "scroll_time_in_millis": 0,
                "scroll_current": 0,
                "suggest_total": 0,
                "suggest_time_in_millis": 0,
                "suggest_current": 0
            },
            "merges": {
                "current": 0,
                "current_docs": 0,
                "current_size_in_bytes": 0,
                "total": 0,
                "total_time_in_millis": 0,
                "total_docs": 0,
                "total_size_in_bytes": 0,
                "total_stopped_time_in_millis": 0,
                "total_throttled_time_in_millis": 0,
                "total_auto_throttle_in_bytes": 104857600
            },
            "refresh": {
                "total": 15,
                "total_time_in_millis": 0,
                "listeners": 0
            },
            "flush": {
                "total": 5,
                "periodic": 0,
                "total_time_in_millis": 0
            },
            "warmer": {
                "current": 0,
                "total": 5,
                "total_time_in_millis": 0
            },
            "query_cache": {
                "memory_size_in_bytes": 0,
                "total_count": 0,
                "hit_count": 0,
                "miss_count": 0,
                "cache_size": 0,
                "cache_count": 0,
                "evictions": 0
            },
            "fielddata": {
                "memory_size_in_bytes": 0,
                "evictions": 0
            },
            "completion": {
                "size_in_bytes": 0
            },
            "segments": {
                "count": 0,
                "memory_in_bytes": 0,
                "terms_memory_in_bytes": 0,
                "stored_fields_memory_in_bytes": 0,
                "term_vectors_memory_in_bytes": 0,
                "norms_memory_in_bytes": 0,
                "points_memory_in_bytes": 0,
                "doc_values_memory_in_bytes": 0,
                "index_writer_memory_in_bytes": 0,
                "version_map_memory_in_bytes": 0,
                "fixed_bit_set_memory_in_bytes": 0,
                "max_unsafe_auto_id_timestamp": -1,
                "file_sizes": {}
            },
            "translog": {
                "operations": 0,
                "size_in_bytes": 1375,
                "uncommitted_operations": 0,
                "uncommitted_size_in_bytes": 1375,
                "earliest_last_modified_age": 0
            },
            "request_cache": {
                "memory_size_in_bytes": 0,
                "evictions": 0,
                "hit_count": 0,
                "miss_count": 0
            },
            "recovery": {
                "current_as_source": 0,
                "current_as_target": 0,
                "throttle_time_in_millis": 0
            }
        }
    },
    "indices": {
        "event20230221_v0": {
            "uuid": "HEd19q-eS3at1UH71p0VSQ",
            "primaries": {
                "docs": {
                    "count": 0,
                    "deleted": 0
                },
                "store": {
                    "size_in_bytes": 1305
                },
                "indexing": {
                    "index_total": 0,
                    "index_time_in_millis": 0,
                    "index_current": 0,
                    "index_failed": 0,
                    "delete_total": 0,
                    "delete_time_in_millis": 0,
                    "delete_current": 0,
                    "noop_update_total": 0,
                    "is_throttled": false,
                    "throttle_time_in_millis": 0
                },
                "get": {
                    "total": 0,
                    "time_in_millis": 0,
                    "exists_total": 0,
                    "exists_time_in_millis": 0,
                    "missing_total": 0,
                    "missing_time_in_millis": 0,
                    "current": 0
                },
                "search": {
                    "open_contexts": 0,
                    "query_total": 15,
                    "query_time_in_millis": 0,
                    "query_current": 0,
                    "fetch_total": 0,
                    "fetch_time_in_millis": 0,
                    "fetch_current": 0,
                    "scroll_total": 0,
                    "scroll_time_in_millis": 0,
                    "scroll_current": 0,
                    "suggest_total": 0,
                    "suggest_time_in_millis": 0,
                    "suggest_current": 0
                },
                "merges": {
                    "current": 0,
                    "current_docs": 0,
                    "current_size_in_bytes": 0,
                    "total": 0,
                    "total_time_in_millis": 0,
                    "total_docs": 0,
                    "total_size_in_bytes": 0,
                    "total_stopped_time_in_millis": 0,
                    "total_throttled_time_in_millis": 0,
                    "total_auto_throttle_in_bytes": 104857600
                },
                "refresh": {
                    "total": 15,
                    "total_time_in_millis": 0,
                    "listeners": 0
                },
                "flush": {
                    "total": 5,
                    "periodic": 0,
                    "total_time_in_millis": 0
                },
                "warmer": {
                    "current": 0,
                    "total": 5,
                    "total_time_in_millis": 0
                },
                "query_cache": {
                    "memory_size_in_bytes": 0,
                    "total_count": 0,
                    "hit_count": 0,
                    "miss_count": 0,
                    "cache_size": 0,
                    "cache_count": 0,
                    "evictions": 0
                },
                "fielddata": {
                    "memory_size_in_bytes": 0,
                    "evictions": 0
                },
                "completion": {
                    "size_in_bytes": 0
                },
                "segments": {
                    "count": 0,
                    "memory_in_bytes": 0,
                    "terms_memory_in_bytes": 0,
                    "stored_fields_memory_in_bytes": 0,
                    "term_vectors_memory_in_bytes": 0,
                    "norms_memory_in_bytes": 0,
                    "points_memory_in_bytes": 0,
                    "doc_values_memory_in_bytes": 0,
                    "index_writer_memory_in_bytes": 0,
                    "version_map_memory_in_bytes": 0,
                    "fixed_bit_set_memory_in_bytes": 0,
                    "max_unsafe_auto_id_timestamp": -1,
                    "file_sizes": {}
                },
                "translog": {
                    "operations": 0,
                    "size_in_bytes": 1375,
                    "uncommitted_operations": 0,
                    "uncommitted_size_in_bytes": 1375,
                    "earliest_last_modified_age": 0
                },
                "request_cache": {
                    "memory_size_in_bytes": 0,
                    "evictions": 0,
                    "hit_count": 0,
                    "miss_count": 0
                },
                "recovery": {
                    "current_as_source": 0,
                    "current_as_target": 0,
                    "throttle_time_in_millis": 0
                }
            },
            "total": {
                "docs": {
                    "count": 0,
                    "deleted": 0
                },
                "store": {
                    "size_in_bytes": 1305
                },
                "indexing": {
                    "index_total": 0,
                    "index_time_in_millis": 0,
                    "index_current": 0,
                    "index_failed": 0,
                    "delete_total": 0,
                    "delete_time_in_millis": 0,
                    "delete_current": 0,
                    "noop_update_total": 0,
                    "is_throttled": false,
                    "throttle_time_in_millis": 0
                },
                "get": {
                    "total": 0,
                    "time_in_millis": 0,
                    "exists_total": 0,
                    "exists_time_in_millis": 0,
                    "missing_total": 0,
                    "missing_time_in_millis": 0,
                    "current": 0
                },
                "search": {
                    "open_contexts": 0,
                    "query_total": 15,
                    "query_time_in_millis": 0,
                    "query_current": 0,
                    "fetch_total": 0,
                    "fetch_time_in_millis": 0,
                    "fetch_current": 0,
                    "scroll_total": 0,
                    "scroll_time_in_millis": 0,
                    "scroll_current": 0,
                    "suggest_total": 0,
                    "suggest_time_in_millis": 0,
                    "suggest_current": 0
                },
                "merges": {
                    "current": 0,
                    "current_docs": 0,
                    "current_size_in_bytes": 0,
                    "total": 0,
                    "total_time_in_millis": 0,
                    "total_docs": 0,
                    "total_size_in_bytes": 0,
                    "total_stopped_time_in_millis": 0,
                    "total_throttled_time_in_millis": 0,
                    "total_auto_throttle_in_bytes": 104857600
                },
                "refresh": {
                    "total": 15,
                    "total_time_in_millis": 0,
                    "listeners": 0
                },
                "flush": {
                    "total": 5,
                    "periodic": 0,
                    "total_time_in_millis": 0
                },
                "warmer": {
                    "current": 0,
                    "total": 5,
                    "total_time_in_millis": 0
                },
                "query_cache": {
                    "memory_size_in_bytes": 0,
                    "total_count": 0,
                    "hit_count": 0,
                    "miss_count": 0,
                    "cache_size": 0,
                    "cache_count": 0,
                    "evictions": 0
                },
                "fielddata": {
                    "memory_size_in_bytes": 0,
                    "evictions": 0
                },
                "completion": {
                    "size_in_bytes": 0
                },
                "segments": {
                    "count": 0,
                    "memory_in_bytes": 0,
                    "terms_memory_in_bytes": 0,
                    "stored_fields_memory_in_bytes": 0,
                    "term_vectors_memory_in_bytes": 0,
                    "norms_memory_in_bytes": 0,
                    "points_memory_in_bytes": 0,
                    "doc_values_memory_in_bytes": 0,
                    "index_writer_memory_in_bytes": 0,
                    "version_map_memory_in_bytes": 0,
                    "fixed_bit_set_memory_in_bytes": 0,
                    "max_unsafe_auto_id_timestamp": -1,
                    "file_sizes": {}
                },
                "translog": {
                    "operations": 0,
                    "size_in_bytes": 1375,
                    "uncommitted_operations": 0,
                    "uncommitted_size_in_bytes": 1375,
                    "earliest_last_modified_age": 0
                },
                "request_cache": {
                    "memory_size_in_bytes": 0,
                    "evictions": 0,
                    "hit_count": 0,
                    "miss_count": 0
                },
                "recovery": {
                    "current_as_source": 0,
                    "current_as_target": 0,
                    "throttle_time_in_millis": 0
                }
            }
        }
    }
}
```

# 分词器使用

-  按照索引字段进行分词
  
  ```
  curl -X GET "localhost:9200/analyze_sample/_analyze?pretty" -H 'Content-Type: application/json' -d'
  {
   "field" : "obj1.field1",
   "text" : "this is a test"
  }
  '
  ```

- 内置分词器进行分词
  
  ```
  curl -X GET "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'
  {
    "analyzer" : "standard",
    "text" : "this is a test"
  }
  '
  ```

# 模板

## 获取

```
curl -X GET "localhost:31600/_template/{模板名称}?pretty"
curl -X GET "localhost:9200/_template?pretty"//获取所有
```

    返回值

## 创建

```
curl -X PUT "localhost:9200/_template/template_1?pretty" -H 'Content-Type: application/json' -d'
{
    "index_patterns" : ["event*"],//匹配某类索引
     "order" : 1,//如果一个索引对于两个模板，模板进行合并，冲突时采用order高的
    "settings" : {
        "number_of_shards" : 1
    },
   "version": 123,//版本号
    "aliases" : {
        "alias1" : {},
        "alias2" : {
            "filter" : {
                "term" : {"user" : "kimchy" }
            },
            "routing" : "kimchy"
        },
        "{index}-alias" : {} 
    }
}
'
```

## 更新 todo

## 删除

```
curl -X DELETE "localhost:9200/_template/template_1?pretty"
```
