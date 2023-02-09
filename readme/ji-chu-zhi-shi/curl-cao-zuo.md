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

## 文档指令

### 读写文档原理

- 写入单个文档
  
  - 客户端向集群的某个节点发起请求，该节点就是协调节点
  
  - 协调节点根据文档id确定文档所在分片，根据集群路由表信息确定主分片位置，发送请求到主分片所在节点
  
  - 执行主分片写操作；写入成功后并行转发至副本分片所在节点，等所有副本分片都写入成功后，发送报告至协调节点，协调节点发送至客户端

- 写入故障处理 TODO
  
  - 主节点故障
    
    - 给主节点发送确定信息，最多等待1分钟，然后选择其中一个副本提升为新的主副本，将请求发送给新的节点进行处理
  
  - 副本节点故障：只有主分片运行，就会有数据丢失的风险

- 读取单个文档
  
  - 客户端发送请求至某个节点，该节点就是协调节点
  
  - 协调节点根据文档ID确定文档所在分片
  
  - 通过集群状态路由表信息确定副本信息，请求转发至副本分片所在节点读取信息（多个副本分片客户端轮询来负载均衡）
  
  - 副本分片节点返回信息至协调节点，然后返回至客户端

- search文档
  
  - query阶段
    
    - 客户端向集群的某个节点发起请求，该节点就是协调节点
    
    - 协调节点将查询转发至索引的每一个主分片或者副本分片中
    
    - 每个分片本地执行查询，使用本地的term，document frequency信息进行发奋，结果添加到ftrom+size的本地有序优先队列中
    
    - 每个分片返回各自队列的所有文档id和排序值给协调节点，协调节点合并至自己的优先队列中，产生一个全局排序后的列表
  
  - fetch阶段
    
    - 协调节点只要了要获取那些信息，但是还没有具体的数据，需要执行多个get请求获取数据
    
    - 分片所在节点向协调节点返回数据
    
    - 所有数据返回完成，然会给客户端

### 文档操作

#### index操作

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

- 路由
  
  - 分片获取方式`hash(routing)%分片数量`，es默认使用_id为routing确定分片位置，然后进行写入和查询
  
  - 指定routing后搜索效率会增加，但是会导致同一分片上面_id不唯一和索引数据分布不均的问题
  
  - `index.routing_partition_size`设置索引根据routing路由shard的数量，可解决一部分数据分布不均问题
    
    ```
    curl -X POST "localhost:9200/twitter/_doc?routing=kimchy&pretty" -H 'Content-Type: application/json' -d'
    {
        "user" : "kimchy",
        "post_date" : "2009-11-15T14:12:12",
        "message" : "trying out Elasticsearch"
    }
    '
    
    ```
    
    

- 等待活动分片：索引请求返回前需要等待多少个分片写入成功，
  
  - 最小值1（默认，主分片）
  
  - 最大值 副本数量+1

- 索引自增
  
  - TODO

#### get获取文档信息

索引信息和数据如下

![](../image/curl-cao-zuo/2023-02-09-17-17-06-image.png)

![](../image/curl-cao-zuo/2023-02-09-17-17-53-image.png)

- source字段过滤
  
  - ```
    curl -X GET "localhost:9200/twitter/_doc/0?_source_includes=*.id&_source_excludes=entities&pretty"
    
    ```
  
  - ![](../image/curl-cao-zuo/2023-02-09-17-24-12-image.png)

- 只获取source字段

- ```
  curl -X GET "localhost:9200/twitter/_doc/1/_source?_source_includes=*.id&_source_excludes=entities&pretty"
  
  ```
  
  ![](../image/curl-cao-zuo/2023-02-09-17-09-29-image.png)

- stored_fields
  
  - 和source的区别是只展示已存储数据的字段(mapping中counter的store是false)
  - ```
    curl -X GET "localhost:9200/twitter/_doc/1?stored_fields=tags,counter&pretty"
    
    ```
  - ![](../image/curl-cao-zuo/2023-02-09-17-16-19-image.png)
- 指定routing
  
  ```
  curl -X GET "localhost:9200/twitter/_doc/2?routing=user1&pretty"
  
  ```

#### delete删除文档信息

- 基本用法
  
  - ```
    curl -X DELETE "localhost:9200/twitter/_doc/1?pretty"
    ```

- 路由配置

- 等待活动分片wait_for_active_shards

- 刷新

- 暂停:如果主分片不可用等待多长时间返回
  
  - ```
    curl -X DELETE "localhost:9200/twitter/_doc/1?timeout=5m&pretty"
    ```

#### 通过query去delete删除文档信息

- 基本用法
  
  - ```
    curl -X POST "localhost:9200/twitter/_delete_by_query?pretty" -H 'Content-Type: application/json' -d'
    {
      "query": { 
        "match": {
          "message": "some message"
        }
      }
    }
    '
    ```
  
  - 删除逻辑：依次执行多个搜索请求，找到所有匹配文档并删除；没找到一批文档就会执行一批的删除操作；如果某些批量操作被拒绝（重试10次），已删除的数据不会回滚
  
  - ![](../image/curl-cao-zuo/2023-02-09-18-17-06-image.png)

- conflicts=proceed因版本冲突导致的种植将不存在
  
  - ```
    curl -X POST "localhost:9200/twitter/_doc/_delete_by_query?conflicts=proceed&pretty" -H 'Content-Type: application/json' -d'
    {
      "query": {
        "match_all": {}
      }
    }
    '
    ```

- 更多支持：routing=1、croll_size=5000（每批数据的大小）、pretty、refresh、wait_for_completion（TODO）、wait_for_active_shards（活动分片数量）、timeout（等待不可用分片变成可用分片的时间)、scroll(搜索上线文保存活动状态的时间)、requests_per_second（限制批量操作速率）

- 启用任务删除  TODO

#### 更新文档

## 简单操作命令

- 创建索引和mapping
  
  - ![](../image/curl-cao-zuo/68087e750c060cfd21ac0bc04b922e0b623ddc59.jpg)

- 删除索引  
  
  ![](../image/curl-cao-zuo/0c62f8ea1cb5fa2071e26fa3a482816f7c89f0fa.jpg)

- 创建文档和修改文档
  
  - ![](../image/curl-cao-zuo/7b1a0ead1c1ffd090ae797081135040a882582b9.jpg)

- 查询文档
  
  - 通过id查询
    
    - ![](../image/curl-cao-zuo/93adabbdde12fc4bfcc21bc4f15dec0e0d47e512.jpg)
  
  - query-string查询
    
    - ![](../image/curl-cao-zuo/d3037b29b9bf4bb138db6abde06c65b2ca1fed04.jpg)
  
  - term查询
    
    - ![](../image/curl-cao-zuo/bff52aa8c4d78cba07bbd1b3dd93fd86eae3c58e.jpg)
