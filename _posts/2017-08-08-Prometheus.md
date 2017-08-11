# Prometheus

------

Prometheus是SoundCloud开源监控解决方案。作为新一代开源解决方案，很多理念和Google SRE运维之道不谋而合。

主要功能：
+ 多维数据模型(时序有metric名字和k/v的labels构成)
+ 灵活的查询语句(PromQL)
+ 无依赖存储，支持local和remote不同模型
+ 采用http协议，使用pull模式，拉取数据
+ 监控目标，可以采用服务发现或静态配置的方式
+ 支持多种统计数据模型，图形化友好

核心组件
+ Prometheus Server: 主要用于抓取数据和存储时序数据，另外还提供查询和Alert Rule配置管理

+ client libraries: 用于对接PrometheusServer，可以查询和上报数据

+ push gateway: 用于批量，短期的监控数据的汇总节点，主要用于业务数据汇报等

+ 各种汇报数据的exporters，例如汇报机器数据的node_exporter，汇报MongoDB信息的MongoDB exporter等

+ 用于告警通知管理的alertmanager

![prometheus-01.png](https://aaron-13.github.io/images/prometheus-01.png)

主要模块包含: Server,Exporters,Pushgateway,PromQL,Alertmanager,WebUI等

大致使用逻辑为:
1. Prometheus Server定期从静态配置的targets或者服务发现的targets拉取数据
2. 当新拉取的数据大于配置内存缓存区的时候，Prometheus会将数据持久化到磁盘(如果使用remote storage将持久化到云端)
3. Prometheus可以配置rules，然后定时查询数据，当条件触发的时候，会将alert推送到配置的Alertmanager
4. Alertmanager收到警告的时候，可以根据配置，聚合，去重，降噪，最后发送警告
5. 可以使用API，Prometheus Console或者Grafana查询和聚合数据


**注意**
+ Prometheus的数据是基于时序的float64的值，如果数据值有更多类型，无法满足

+ Prometheus不适合做审计计费，因为数据是按一定时间采集的，关注的更多的是系统的运行瞬间状态以及趋势，即使有少量没有采集也能容忍，但审计计费则需要记录每个请求，并且长期存储


**总结**
+ Prometheus属于一站式监控告警平台，依赖少，功能齐全
+ Prometheus支持对云的或容器的监控，其他系统主要对主机监控
+ Prometheus数据查询语句表现力更强大，内置更强大的统计函数
+ Prometheus在数据存储扩展性以及持久性上没有InfluxDB，OpenTSDB，Sensu好

