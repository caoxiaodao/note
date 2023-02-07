# 简介和安装

## ES安装

- 官网地址 [https://www.elastic.co/products/elasticsearch](https://link.zhihu.com/?target=https%3A//www.elastic.co/products/elasticsearch)

- windows版本：zip包解压缩之后在bin之下有.bat的脚本直接运行

- linux版本：TODO
  
   

## 图形化工具安装

### es-header

- 官网地址 [https://github.com/mobz/elasticsearch-head](https://link.zhihu.com/?target=https%3A//github.com/mobz/elasticsearch-head)

- 安装步骤
  
  - 解压缩zip包
  
  - 安装grunt插件：npm install -g grunt-cli
  
  - 连接不上需要修改如下配置：config/elasticsearch.yml，允许跨域访问  
    
    `http.cors.enabled: true  `
    
    `http.cors.allow‐origin: "*"`

## ES简介

 Elasticsearch 是一个分布式文档存储。支持1S以内的近乎实时入库和查询。每个索引都有一个专用的数据结构。例如：文本字段存储在到排序索引中，数字和地址字段存储在BKD树索引中；但是ES还具有无模式的能力，就是自动的根据文档的字段判断其类型

### 术语

- 文档：可以被索引的基础信息单元，就是一条数据；并且该文档必须有一个类型

- 字段：对于文档来说是有很多信息的比如书名，作者，编号，简介等等信息，每一个信息单元就是一个字段

- 映射：每一个字段都应该设定数据类型，默认值，分析器，能否索引等等

- 类型：关于文档我们应该有很多方面的书本，客户等等，所以我们就需要多个类型

- 索引：文档的集合；可以包含多个类型  
  
  - > 比如现在建立一个卖书的商城系统，是不是需要书本的信息，账户的信息，订单的信息；而当形成一个订单之后，订单中有参杂着书本信息等等，是否要将书本类型，账户类型和订单类型放在同一个索引当中这个是我们值得考虑的问题，当然上面的答案显而易见，但是遇到复杂的情况我们就要进行评估和设计了；理解仍有问题请参照如下链接
    
    > > 重点注意！！！6.0的版本不允许一个index下面有多个type，并且官方说是在接下来的7.0版本中会删掉type
  
  - [ES中index和type的区别_行走的IT-CSDN博客_es typeblog.csdn.net](https://link.zhihu.com/?target=https%3A//blog.csdn.net/tengxing007/article/details/100663530)

- 节点：一个节点是集群中的一个服务器，由一个名字来标识，并且通过集群的名字加入到一个指定的集群当中，默认的集群名称是"elasticsearch"

- 集群：一个集群由一个节点或者多个节点组成，共同持有整个数据，并一起提供索引和所有功能

- 分片和复制：索引可以建立多个分片>水平分割数据，进行分布式，并行操作

- 复制：高可用性，主从分片不要再同一个节点上
  
  - > 索引创建好之后复制数量可以更改，分片数量不可以更改  
    > 分片怎么分布，怎么聚合相应搜索请求时es管理的不用干预

### 基本功能

Elasticsearch 提供 REST API、Java、JavaScript、Go、.NET、PHP、Perl、Python 、 Ruby。

REST API 支持结构化查询、全文查询、两者结合查询；单个术语搜索，短语搜索、相似性搜索、前缀搜索、地理位置搜索

可以使用 [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/query-dsl.html "查询 DSL")也可以使用[SQL 样式的查询](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/sql-overview.html "概述")；可以使用SQL和ES进行交互

### 扩展性

- 原理
  
  - es索引是一个或者多个的物理分片的逻辑分组；索引中的文档分布在多个分片上，这些分片又分布在多个节点上
  
  - 分片为主分片和副本分片；主分片数量固定，副本分片数量可随时更改

- 应用：分片越多，维护索引的开销越大；分片大小越多，重新平衡集群，移动分片所需时间越长
  
  - 分片大小最好在几GB-几十GB之间；如果是基于时间的数据可以为20-40GB之间
  
  - 避免巨大的碎片问题。节点可以容纳的分片数量与可用堆空间成正比。作为一般规则，每 GB 堆空间的分片数应少于 20。
  
  - [使用您自己的数据和查询进行测试](https://www.elastic.co/elasticon/conf/2016/sf/quantitative-cluster-sizing)。

### 常见配置和解释

#### elasticsearch.yml

- path.data ：数据目录

- path.logs ：日志目录 

- [`cluster.name`](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/cluster.name.html) ：集群名称

- [`node.name`](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/node.name.html)：节点名称

- [`network.host`](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/network.host.html)：绑定host
  
  - 特殊地址
    
    `_[networkInterface]_  网络接口的地址，例如_en0_`
    
    `[_local_]  系统上回环地址127.0.0.1`
    
    `[_site_]  系统上任何站点本地地址 192.168.0.0.1`
    
    ` [__global__]  系统上任何全局范围的地址 8.8.8.8`

-   http.port:http请求绑定端口
  
  - 示例：`31600-31700`

- transport.tcp.port:节点通信绑定端口

- discovery.zen.ping.unicast.hosts：集群发现
  
  - 示例
    
    `192.168.1.10:9300 ` 
    
    `192.168.1.11`
    
    `seeds.mydomain.com `  

- discovery.zen.hosts_provider：file  集群发现，动态文件配置
  
  - 文件位置  config目录下unicast_hosts.txt

- discovery.zen.minimum_master_nodes:集群建议的最少主节点数量
  
  - 脑裂现象：(主节点数量/2)+1

- 

### jvm.options

- Xms 和Xmx：堆内存设置
  
  - 不超过物理RAM的50%
  
  - 不超过JVM的压缩对象指针，接近于32GB（26GB大多系统试用，有些可达到30GB）
  
  - 

#### log4j2.properties

### 搜索和聚合

- 搜索
  
  - _search查询
  
  - from，size限制
  
  - match，match_phrase，bool（`must`、`should`和`must_not`）

- 聚合
  
  - aggregations（aggs）的term聚合
