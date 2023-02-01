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

## ES术语简介

有很多文档会把他和关系型数据库的数据库，表，行进行对应，但是我觉得这样的对应反而对我们的理解以及使用产生一些不太正确的理解，所以我这里不会进行这样的对应

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

- 


