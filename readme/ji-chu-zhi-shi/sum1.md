报错现象：告警可以查询，无法入库

ES基础概念

  ES：存储

  Logstash 日志采集，开源框架（java开发）

  kibana 可视化；管理监控

  beats基于go语言开发，轻量级，使用的内存，cpu都很低

- 如果 `index_current` 很高（比如几十甚至上百），说明写入压力大或刷新较慢。
- 如果 `is_throttled` 为 `true`，说明你的写入速率超过了刷新能力，需要调整配置。
- 如果 `throttle_time_in_millis` 不断增长，也说明存在写入瓶颈。
