# 常用命令

+ 查看集群健康状态  
  
  ```
  curl http://localhost:31600/_cat/health?v  
  curl http://localhost:31600/_cluster/health?pretty
  ```

+ 查看索引  
  
  ```
  curl -XGET 'localhost:31600/_cat/indices?v&pretty'  
  ```

+ 查看段内存状态  
  
  ```
  curl 'http://localhost:31600/_cat/nodes?v&h=segments.count,segments.memory,segments.index_writer_memory,segments.version_map_memory,segments.fixed_bitset_memory'  
  ```

+ 删除索引数据  
  
  ```
  curl -XPOST -H "Content-Type:application/json" -d '{"query":{"match_all":{}}}' 'http://127.0.0.1:31600/ti_v0/_delete_by_query/'  
  ```

+ 查询索引数据
  
  ```
  curl -X GET "localhost:31600/alert_event_v0/_search?pretty" -H 'Content-Type: application/json' -d'
  {
  "query" : {
  "term" : { "id" : "19797792150400108" }
  }
  }
  '
  ```
- 查看node段内存占用
  
      curl -XGET 'localhost:31600/_cat/nodes?help'
      
      curl -XGET "localhost:31600/_cat/nodes?v&h=id,ip,port,v,master,name,heap.current,heap.percent,heap.max,ram.current,ram.percent,ram.max,fielddata.memory_size,fielddata.evictions,query_cache.memory_size,query_cache.evictions,request_cache.memory_size,request_cache.evictions,request_cache.hit_count,request_cache.miss_count"
      
      curl 'http://localhost:31600/_cat/nodes?v&h=segments.count,segments.memory,segments.index_writer_memory,segments.version_map_memory,segments.fixed_bitset_memory'

- 查看indces信息
  
      curl -XGET 'localhost:31600/_cat/indices?pri&v&h=index,health,pri,rep,docs.count,store.size,pri.store.siz是。e,segments.memory,segments.count,pri.segments.memory,pri.segments.count'
