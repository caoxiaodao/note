# 问题记录

## 数据无法入库

### 现象

ES的集群状态正常，但是数据无法正常入库，运行的tomcat无法连接

### 排查过程：

 1.报错信息

![](E:\project\gitcode\notenew\note\readme\问题记录\1数据无法数据\arkbase报错1.png)

![arkbase报错2](E:\project\gitcode\notenew\note\readme\问题记录\1数据无法数据\arkbase报错2.png)

![tomcat报错](E:\project\gitcode\notenew\note\readme\问题记录\1数据无法数据\tomcat报错.png)

### 原因

  1.查看内存分配32G 

```
ps -ef|grep elasticsearch
```


  2.查看内存占用：21G 

```
curl 'http://localhost:31600/_cat/nodes?v&h=segments.count,segments.memory,segments.index_writer_memory,segments.version_map_memory,segments.fixed_bitset_memory'
```

3. es内存占用说明 ：如果超过分配内存的70%无法保证数据入库
    https://www.elastic.co/guide/en/elasticsearch/guide/master/heap-sizing.html#_just_how_far_under_32gb_should_i_set_the_jvm  
4. es断路器说明  https://www.elastic.co/guide/en/elasticsearch/reference/5.3/circuit-breaker.html

### 解决方案

#### 临时解决：关闭部分在线索引

#### 永久解决：

1.解决方案

 [海燕部署优化方案.pdf](问题记录\1数据无法数据\海燕部署优化方案.pdf) 

2.注意事项

- 扩容后所有索引为green状态

- 每个ES节点的data目录应该在不同的磁盘：可能会受到IO的限制，读取还是有问题es入门
