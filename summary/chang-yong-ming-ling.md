# 常用命令

+ 查看集群健康状态  
  ````
  curl http://localhost:31600/_cat/health?v  
  curl http://localhost:31600/_cluster/health?pretty

+ 查看索引  
  ````
  curl -XGET 'localhost:31600/_cat/indices?v&pretty'  

+ 查看段内存状态  
  ```
  curl 'http://localhost:31600/_cat/nodes?v&h=segments.count,segments.memory,segments.index_writer_memory,segments.version_map_memory,segments.fixed_bitset_memory'  

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
