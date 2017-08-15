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




## 二进制包安装

------

下载对应的Prometheus Server包及其相关组件

+ [Prometheus Server](https://prometheus.io/download/)

+ [其他相关组件](https://prometheus.io/download/)

+ 解压缩对应的包

+ 执行`./prometheus -version`查看相应版本

+ 启动`./prometheus` 
![prometheus](https://aaron-13.github.io/images/prometheus-02.png)



## Docker安装

------

+ 先安装Docker

+ 编译安装prometheus，需要go语言环境的支持

```
yum install go(epel源)

编译，还需要设置GOPATH
```

+ 下载docker中对应的prometheus镜像

+ 运行docker
```
docker run --name prometheus -d -p 127.0.0.0:9090:9090 prom/prometheus

```




## Prometheus基础概念

------

+ 数据模型

+ 四种Metric Type

+ 作业与实例


**数据模型**

Prometheus存储的是时序数据，即按照相同时序(相同的名字和标签)，以时间维度存储连续的数据的集合。


时序索引
时序(time series)是由名字(Metirc),以及一组key/value标签定义的，具有相同的名字以及标签属于相同时序。
时序的名字由ASCII字符，数字，下划线，以及冒号组成，
必须满足正则表达式\[a-zA-Z_:\]\[a-zA-Z0-9_:\]*,其名字应该具有语义化，一般表示一个可以度量的指标，如http_request_total，表示http请求的总数。

时序的标签可以使Prometheus的数据更加丰富，能够区分具体不同的实例，例如http_request_total{method="POST"}可以表示所有http中的POST请求。

标签名称由ASCII字符，数字，以及下划线组成，其中__开头属于Prometueus保留，标签的值可以是任何Unicode字符，支持中文。


时序样本
按照某个时序一时间维度采集的数据，称之为样本，其值包含
+ 一个float64值

+ 一个毫秒级的unix时间戳


格式
```
<metric name>{<lable name>=<label value>,...}
```
包含时序的名字和标签



## 时序的四种类型

Prometheus时序数据分为Counter，Gauge，Histogram，Summary四种类型

**Counter**
Counter表示收集的数据是按照某个趋势(增加/减少)一直变化的，往往用来记录服务请求总数，错误总数。


**Gauge**
Gauge表示搜集的数据是一个瞬时的，与时间没有关系，可以任意变高变低，往往可以用来记录内存使用率，磁盘使用率等。


**Histogram**
Histogram由<basename>_bucket{le="<upper inclusive bound>"}, <basename>_bucket{le="+Inf"},<basename>_sum,<basename>_count组成，主要用于表示一段时间范围内对数据进行采样，(通常是请求持续时间或响应大小)，并能够对其指定区间以及总数进行统计，通过用它计算分位数的直方图。
例如Prometheus server中prometheus_local_storage_series_chunks_persisted，表示Prometheus中每个时序需要存储的chunks数量，可以用它来计算持久化的数据的分位数。


**Summary**
Summary和Histogram类似，由<basename>{quantile="<φ>"}，<basename>_sum,<basename>_count组成，主要用于表示一段时间内数据采样结果，(通常是请求持续时间或响应大小)，它直接存储了quantile数据，而不是根据统计区间计算出来的。
例如Prometheus server中prometheus_target_interval_length_seconds.


**Histogram vs Summary**
+ 都包含<basename>_sum,<basename>_count

+ Histogram 需要通过<basename>_bucket计算quantile，而summary直接存储了quantile的值



## 作业和实例

------

Prometheus中，将任意一个独立的数据源(target)称之为实例(instance).包含相同类型的实例的集合称之为作业(job),如下是一个含有四个重复实例的作业:

```
- job: api-server
	- instance 1: 1.2.3.4:5670
	- instance 2: 1.2.3.4:5671
	- instance 3: 1.2.3.4:5672
	- instance 4: 1.2.3.4:5673

```


**自生成标签和时序**

Prometheus在采集数据的同时，会自动在时序的基础上添加标签，作为数据源(target)的标识，以便区分:
```
job: The configured job name that the target belongs to.
instance: The <host>:<port> part of the target's URL that was scraped.

```

如果其中任一标签已经在此前采集的数据中存在，那么将会根据honor_labels设置选项来决定新标签。

对于每个实例而言，prometheus按照以下时序来存储所采集的数据样本
```
up{job="<job-name>",instance="<instance-id>"}:1 表示该实例正常工作
up{job="<job-name>",instance="<instance-id>"}:0 表示该实例故障

scrape_duration_seconds{job="<job-name>",instance="<instance-id>"} 表示拉取数据的时间间隔

scrape_samples_post_metric_relabeling{job="<job-name>",instance="<instance-id>"} 表示采用重定义标签(relabeling)操作后仍然剩余的样本数

scrape_samples_scraped{job="<job-name>",instance="<instance-id>"} 表示从该数据源获取的样本数

```
其中up时序可以有效应用于监控该实例是否正常工作



## PromQL基本使用

------

PromQL是Prometheus开发的数据查询DSL语言，语言表现力丰富，内置函数很多，在日常数据可视化，rule告警中都会使用到它。

如在graph页面中，查询下面的语句:
```
http_request_total{code="200"}
```

**字符串和数字**

字符串: 在查询语句中，字符串往往作为查询条件labels的值，和Golang字符串语法一致，可以使用"",'',或者``，格式如:

```
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t`
```

正数，浮点数:表达式中可以使用正数或浮点数，例如:
```
3
-2.3
```


**查询结果类型**
PromQL查询结果主要有三种类型:
+ 瞬时数据(Instant vector): 包含一组时序，每个时序只有一个点，例如: http_requests_total

+ 区间数据(Range vector): 包含一组时序，每个时序有多个点，例如 http_requests_total[5m]

+ 纯量数据(Scalar): 纯量只有一个数字，没有时序，如：count(http_requests_total)


**查询条件**
Prometheus存储的是时序数据，而它的时序是由名字和一组标签构成的，其实名字也可以写出标签的形式，如http_requests_total等价于{name="http_requests_total"}

查询条件支持正则匹配，如:
```
http_requests_total{code!="200"} //表示查询code不为200的数据
http_requests_total{code=~"2.."} //表示查询code为2xx的数据
http_requests_total{code!~"2.."} //表示查询code不为2xx的数据
```


**操作符**

查询语句支持常见的各种表达式操作符，例如:

算术运算符:
支持的算术运算符`+,-,*,/,%.^`,例如 http_requests_total * 2，表示将http_requests_total的所有数据double一倍

比较运算符
支持的比较运算符有`==,!=,<,>,<=,>=`,例如`http_requests_total > 100`表示http_requests_total结果中大于100的数据

逻辑运算符
支持的逻辑运算符有`and,or,unless`,例如 `http_requests_total ==5 or http_requests_total ==2` 

聚合运算符
支持的聚合运算符有`sum,min,max,,avg,stddev,stdvar,count,count_values,bottomk,topk,quantile`,例如max(http_requests_total)表示http_requests_total结果中最大的数据

注意: 和四则运算类型，Prometheus的运算符也有优先级，遵从`(^) > (*,/,%) > (+,-) > (==,!=,<=,>=,<,>) > (and,unless) > (or)`的原则


**内置详情**
Prometheus内置不少函数，方便查询以及数据格式化，例如将结果由浮点数转为整数的floor和ceil
```
floor(avg(http_requests_total{code="200"}))
ceil(avg(http_requests_total{code="200"}))

```

查看http_requests_total5分钟内，平均每秒数据
```
rate(http_requests_total[5m])
```


## 数据可视化

------

Prometheus自带了Web Console，访问http://localhost:9090/graph页面，用它可以进行任何PromQL查询和调试工作。



## Grafana使用

------

Grafana是一套开源的分析监视平台，支持Graphite，InfluxDB，OpenTSDB，Prometheus，Elasticsearch，CouldWatch，Zabbix等数据源。



## 配置

------

Prometheus启动的时候，可以加载运行参数-config.file指定配置文件，默认为prometheus.yml。

在配置文件中，可以指定global，alerting，rule_files，scrape_configs，remote_write,remote_read等属性。

其代码结构定义为:
![prometheus-03.png](https://aaron-13.github.io/images/prometheus-03.png)

配置文件结构:

![prometheus-04.png](https://aaron-13.github.io/images/prometheus-04.png)


**全局配置**

global属于全局的默认配置，主要包含4个属性:

+ scrape_interval: 拉取targets的默认时间间隔

+ scrape_timeout: 拉取一个target的超时时间

+ evaluation_interval: 执行rules的时间间隔

+ external_labels: 额外的属性，会添加到拉取的数据并存到数据库中

其代码结构体定义为:
![prometheus-05.png](https://aaron-13.github.io/images/prometheus-05.png)



**告警配置**

通常使用运行参数-altermanager.xxx来配置Alertmanager但是不够灵活，没有办法做到动态更新加载，以及动态定义告警属性。

所以alerting配置主要用来解决这个问题，能够很好地管理Alertmanager，主要包含两个参数:
+ alert_relable_configs: 动态修改alert属性的规则配置

+ alertmanagers: 用于动态发现Alertmanager的配置

其代码结构大概为:

![prometheus-06.png](https://aaron-13.github.io/images/prometheus-06.png)

配置文件结构大概为:
```
# Alerting specifies settings related to the Alertmanager
alerting:
	alert_relable_configs:
		[ - <relabel_config> ... ]
	alertmanagers:
		[ - <alertmanager_config> ... ]
```

其中alertmanagers为alertmanager_config数组，而alertmanager_config的代码结构体为:

![prometheus-07.png](https://aaron-13.github.io/images/prometheus-07.png)

其配置文件为:
```
# Per-target Alertmanager timeout when pushing alerts.
[ timeout: <duration> | default = 10s ]

# Prefix for the HTTP path alerts are pushed to.
[ path_prefix: <path> | default = / ]

# Configures the protocol scheme used for requests.
[ scheme: <scheme> | default = http ]

# Sets the `Authorization` header on every request with the
# configured username and password.
basic_auth:
  [ username: <string> ]
  [ password: <string> ]

# Sets the `Authorization` header on every request with
# the configured bearer token. It is mutually exclusive with `bearer_token_file`.
[ bearer_token: <string> ]

# Sets the `Authorization` header on every request with the bearer token
# read from the configured file. It is mutually exclusive with `bearer_token`.
[ bearer_token_file: /path/to/bearer/token/file ]

# Configures the scrape request's TLS settings.
tls_config:
  [ <tls_config> ]

# Optional proxy URL.
[ proxy_url: <string> ]

# List of Azure service discovery configurations.
azure_sd_configs:
  [ - <azure_sd_config> ... ]

# List of Consul service discovery configurations.
consul_sd_configs:
  [ - <consul_sd_config> ... ]

# List of DNS service discovery configurations.
dns_sd_configs:
  [ - <dns_sd_config> ... ]

# List of EC2 service discovery configurations.
ec2_sd_configs:
  [ - <ec2_sd_config> ... ]

# List of file service discovery configurations.
file_sd_configs:
  [ - <file_sd_config> ... ]

# List of GCE service discovery configurations.
gce_sd_configs:
  [ - <gce_sd_config> ... ]

# List of Kubernetes service discovery configurations.
kubernetes_sd_configs:
  [ - <kubernetes_sd_config> ... ]

# List of Marathon service discovery configurations.
marathon_sd_configs:
  [ - <marathon_sd_config> ... ]

# List of AirBnB's Nerve service discovery configurations.
nerve_sd_configs:
  [ - <nerve_sd_config> ... ]

# List of Zookeeper Serverset service discovery configurations.
serverset_sd_configs:
  [ - <serverset_sd_config> ... ]

# List of Triton service discovery configurations.
triton_sd_configs:
  [ - <triton_sd_config> ... ]

# List of labeled statically configured Alertmanagers.
static_configs:
  [ - <static_config> ... ]

# List of Alertmanager relabel configurations.
relabel_configs:
  [ - <relabel_config> ... ]
```



**配置规则**

rule_files主要用于配置rules文件，支持多个文件以及文件目录

其代码结构定义为:
```
RuleFiles []string  `yaml:"rule_files,omitempty"`
```

配置文件结构大致为:
```
rule_files:
	- "rules/node.rules"
	- "rules2/*.rules"

```


**数据拉取配置**

scrape_configs主要用于配置拉取数据节点，每一个拉取配置主要包含以下参数:

+ job_name: 任务名称

+ honor_labels: 用于解决拉取数据标签有冲突，当设置为true，以拉取数据为准，否则以服务配置为准

+ params: 数据拉去访问时带的请求参数

+ scrape_interval: 拉取时间间隔

+ scrape_timeout: 拉取超时时间

+ metrics_path: 拉取节点的metirc路径 

+ scheme: 拉取数据访问协议

+ sample_limit: 存储的数据标签个数限制，如果超过限制，该数据将被忽略，不入存储，默认值为0，表示没有限制

+ relabel_configs: 拉取数据重置标签配置

+ metric_relabel_configs: metric重置标签配置


ServiceDiscoveryConfig主要用于target发现，大体分为两类，静态配置和动态发现。所以，一份完整的scrape_configs配置大致为:
```
# The job name assigned to scraped metrics by default.
job_name: <job_name>

# How frequently to scrape targets from this job.
[ scrape_interval: <duration> | default = <global_config.scrape_interval> ]

# Per-scrape timeout when scraping this job.
[ scrape_timeout: <duration> | default = <global_config.scrape_timeout> ]

# The HTTP resource path on which to fetch metrics from targets.
[ metrics_path: <path> | default = /metrics ]

# honor_labels controls how Prometheus handles conflicts between labels that are
# already present in scraped data and labels that Prometheus would attach
# server-side ("job" and "instance" labels, manually configured target
# labels, and labels generated by service discovery implementations).
#
# If honor_labels is set to "true", label conflicts are resolved by keeping label
# values from the scraped data and ignoring the conflicting server-side labels.
#
# If honor_labels is set to "false", label conflicts are resolved by renaming
# conflicting labels in the scraped data to "exported_<original-label>" (for
# example "exported_instance", "exported_job") and then attaching server-side
# labels. This is useful for use cases such as federation, where all labels
# specified in the target should be preserved.
#
# Note that any globally configured "external_labels" are unaffected by this
# setting. In communication with external systems, they are always applied only
# when a time series does not have a given label yet and are ignored otherwise.
[ honor_labels: <boolean> | default = false ]

# Configures the protocol scheme used for requests.
[ scheme: <scheme> | default = http ]

# Optional HTTP URL parameters.
params:
  [ <string>: [<string>, ...] ]

# Sets the `Authorization` header on every scrape request with the
# configured username and password.
basic_auth:
  [ username: <string> ]
  [ password: <string> ]

# Sets the `Authorization` header on every scrape request with
# the configured bearer token. It is mutually exclusive with `bearer_token_file`.
[ bearer_token: <string> ]

# Sets the `Authorization` header on every scrape request with the bearer token
# read from the configured file. It is mutually exclusive with `bearer_token`.
[ bearer_token_file: /path/to/bearer/token/file ]

# Configures the scrape request's TLS settings.
tls_config:
  [ <tls_config> ]

# Optional proxy URL.
[ proxy_url: <string> ]

# List of Azure service discovery configurations.
azure_sd_configs:
  [ - <azure_sd_config> ... ]

# List of Consul service discovery configurations.
consul_sd_configs:
  [ - <consul_sd_config> ... ]

# List of DNS service discovery configurations.
dns_sd_configs:
  [ - <dns_sd_config> ... ]

# List of EC2 service discovery configurations.
ec2_sd_configs:
  [ - <ec2_sd_config> ... ]

# List of OpenStack service discovery configurations.
openstack_sd_configs:
  [ - <openstack_sd_config> ... ]

# List of file service discovery configurations.
file_sd_configs:
  [ - <file_sd_config> ... ]

# List of GCE service discovery configurations.
gce_sd_configs:
  [ - <gce_sd_config> ... ]

# List of Kubernetes service discovery configurations.
kubernetes_sd_configs:
  [ - <kubernetes_sd_config> ... ]

# List of Marathon service discovery configurations.
marathon_sd_configs:
  [ - <marathon_sd_config> ... ]

# List of AirBnB's Nerve service discovery configurations.
nerve_sd_configs:
  [ - <nerve_sd_config> ... ]

# List of Zookeeper Serverset service discovery configurations.
serverset_sd_configs:
  [ - <serverset_sd_config> ... ]

# List of Triton service discovery configurations.
triton_sd_configs:
  [ - <triton_sd_config> ... ]

# List of labeled statically configured targets for this job.
static_configs:
  [ - <static_config> ... ]

# List of target relabel configurations.
relabel_configs:
  [ - <relabel_config> ... ]

# List of metric relabel configurations.
metric_relabel_configs:
  [ - <relabel_config> ... ]

# Per-scrape limit on number of scraped samples that will be accepted.
# If more than this number of samples are present after metric relabelling
# the entire scrape will be treated as failed. 0 means no limit.
[ sample_limit: <int> | default = 0 ]
```



## 远程可写存储

------

remote_write主要用于可写远程存储配置，主要包含以下参数:
+ url: 访问地址

+ remote_timeout: 请求超时时间

+ write_relabel_configs: 标签重置配置，拉取到的数据，经过重置处理后，发送给远程存储

一份完整的配置:
```
# The URL of the endpoint to send samples to.
url: <string>

# Timeout for requests to the remote write endpoint.
[ remote_timeout: <duration> | default = 30s ]

# List of remote write relabel configurations.
write_relabel_configs:
  [ - <relabel_config> ... ]

# Sets the `Authorization` header on every remote write request with the
# configured username and password.
basic_auth:
  [ username: <string> ]
  [ password: <string> ]

# Sets the `Authorization` header on every remote write request with
# the configured bearer token. It is mutually exclusive with `bearer_token_file`.
[ bearer_token: <string> ]

# Sets the `Authorization` header on every remote write request with the bearer token
# read from the configured file. It is mutually exclusive with `bearer_token`.
[ bearer_token_file: /path/to/bearer/token/file ]

# Configures the remote write request's TLS settings.
tls_config:
  [ <tls_config> ]

# Optional proxy URL.
[ proxy_url: <string> ]
```

remote_write属于试验阶段，可能会在以后版本中改变。



## 远程可读存储

------

remote_read主要用于可读远程存储配置，主要包含以下参数:
+ url: 访问地址

+ remote_timeout: 请求超时时间

一份完整的配置:
```
# The URL of the endpoint to query from.
url: <string>

# Timeout for requests to the remote read endpoint.
[ remote_timeout: <duration> | default = 30s ]

# Sets the `Authorization` header on every remote read request with the
# configured username and password.
basic_auth:
  [ username: <string> ]
  [ password: <string> ]

# Sets the `Authorization` header on every remote read request with
# the configured bearer token. It is mutually exclusive with `bearer_token_file`.
[ bearer_token: <string> ]

# Sets the `Authorization` header on every remote read request with the bearer token
# read from the configured file. It is mutually exclusive with `bearer_token`.
[ bearer_token_file: /path/to/bearer/token/file ]

# Configures the remote read request's TLS settings.
tls_config:
  [ <tls_config> ]

# Optional proxy URL.
[ proxy_url: <string> ]
```

remote_read处于试验阶段，可能在以后版本中改变。



## 服务发现

------

在Prometheus的配置中，一个最重要的概念就是数据源target，而数据源的配置主要分为静态配置和动态配置，大致分以下几类:

+ static_configs: 静态服务发现

+ dns_sd_configs: DNS服务发现

+ file_sd_configs: 文件服务发现

+ consul_sd_configs: Consul服务发现

+ serverset_sd_configs: Serverset服务发现

+ nerve_sd_configs: Nerve服务发现

+ marathon_sd_configs: Marathon服务发现

+ kubernetes_sd_configs: Kubernetes服务发现

+ gce_sd_configs: EC2服务发现

+ gce_sd_configs: GCE服务发现

+ openstack_sd_configs: OpenStack服务发现

+ azure_sd_configs: Azure服务发现

+ triton_sd_configs: Triton服务发现

其中使用最广泛的应该是static_configs，其实那些动态类型都可以看成是某些通用业务使用静态服务封装的结果。



**配置样例**

```
global:
	scrape_interval:	15s
	evaluation_interval: 15s

rule_files:
	- "rules/node.rules"

scrape_configs:
	- job_name: 'prometheus'
	  scrape_interval: 5s
	  static_configs:
	  	- targets: ['localhost:9090']

	- job_name: 'node'
	  scrape_interval: 5s
	  static_configs:
	  	- targets: ['localhost:9100','127.0.0.1:9100']

	- job_name: 'mysqld'
	  static_configs:
	  	- targets: ['localhost:9101']

	- job_name: 'memcached'
	  static_configs:
	  	- targets: ['localhost:9102']
```




## Exporter

------

在Prometheus中负责数据汇报的程序统一叫做exporter，而不同的exporter负责不同的业务，它们具有统一命名格式，即xx_exporter，例如负责主机信息收集的node_exporter。


### 文本格式

exporter收集的数据转化的文本内容以行(\n)为单位，空行将被忽略，文本内容最后一行为空行。


**注释**

文本内容，如果以#开头通常表示注释。
+ 以 # HELP 开头表示metric帮助说明

+ 以# TYPE开头表示定义metric类型。包含counter，gauge，Histogram，summary和untyped类型

+ 其他表示一般注释，将被prometheus忽略


**采样数据**

内容如果不以#开头，表示采样数据，通常紧挨着类型定义行，满足以下格式:

```
# HELP http_requests_total The total number of HTTP requests.
# TYPE http_requests_total counter
http_requests_total{method="post",code="200"} 1027 1395066363000
http_requests_total{method="post",code="400"}    3 1395066363000

# Escaping in label values:
msdos_file_access_time_seconds{path="C:\\DIR\\FILE.TXT",error="Cannot find file:\n\"FILE.TXT\""} 1.458255915e9

# Minimalistic line:
metric_without_timestamp_and_labels 12.47

# A weird metric from before the epoch:
something_weird{problem="division by zero"} +Inf -3982045

# A histogram, which has a pretty complex representation in the text format:
# HELP http_request_duration_seconds A histogram of the request duration.
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 24054
http_request_duration_seconds_bucket{le="0.1"} 33444
http_request_duration_seconds_bucket{le="0.2"} 100392
http_request_duration_seconds_bucket{le="0.5"} 129389
http_request_duration_seconds_bucket{le="1"} 133988
http_request_duration_seconds_bucket{le="+Inf"} 144320
http_request_duration_seconds_sum 53423
http_request_duration_seconds_count 144320

# Finally a summary, which has a complex representation, too:
# HELP rpc_duration_seconds A summary of the RPC duration in seconds.
# TYPE rpc_duration_seconds summary
rpc_duration_seconds{quantile="0.01"} 3102
rpc_duration_seconds{quantile="0.05"} 3272
rpc_duration_seconds{quantile="0.5"} 4773
rpc_duration_seconds{quantile="0.9"} 9001
rpc_duration_seconds{quantile="0.99"} 76656
rpc_duration_seconds_sum 1.7560473e+07
rpc_duration_seconds_count 2693
```

需要注意的是，假设采样数据metric叫做x，如果x是histogram或summary类型必须满足以下条件:
+ 采样数据的综合应表示为x_sum

+ 采样数据的总量应表示为x_count

+ summary类型的采样数据的quantile应表示为x{quantile="y"}

+ histogram类型的采样分区统计数据将表示为x_bucket{le="y"}

+ histogram类型的采样必须包含x_bucket{le="+Inf"},它的值等于x_count的值

+ summary和histogram中quantile和le必须按照从小到大顺序排列


**Sample Exporter**

使用golang实现的一个简单sample_exporter：
```
package main

import (
	"fmt"
	"net/http"
	)

func handler(w http.ResponseWriter,r *http.Request) {
	fmt.Fprint(w,exportData)
}

func main(){
	http.HandlerFunc("/",handler)
	http.ListenAndServer(":8080",nil)
}

var exportData string=`
# HELP http_requests_total The total number of HTTP requests.
# TYPE http_requests_total counter
http_requests_total{method="post",code="200"} 1027 1395066363000
http_requests_total{method="post",code="400"}    3 1395066363000

# Escaping in label values:
msdos_file_access_time_seconds{path="C:\\DIR\\FILE.TXT",error="Cannot find file:\n\"FILE.TXT\""} 1.458255915e9

# Minimalistic line:
metric_without_timestamp_and_labels 12.47

# A weird metric from before the epoch:
something_weird{problem="division by zero"} +Inf -3982045

# A histogram, which has a pretty complex representation in the text format:
# HELP http_request_duration_seconds A histogram of the request duration.
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 24054
http_request_duration_seconds_bucket{le="0.1"} 33444
http_request_duration_seconds_bucket{le="0.2"} 100392
http_request_duration_seconds_bucket{le="0.5"} 129389
http_request_duration_seconds_bucket{le="1"} 133988
http_request_duration_seconds_bucket{le="+Inf"} 144320
http_request_duration_seconds_sum 53423
http_request_duration_seconds_count 144320

# Finally a summary, which has a complex representation, too:
# HELP rpc_duration_seconds A summary of the RPC duration in seconds.
# TYPE rpc_duration_seconds summary
rpc_duration_seconds{quantile="0.01"} 3102
rpc_duration_seconds{quantile="0.05"} 3272
rpc_duration_seconds{quantile="0.5"} 4773
rpc_duration_seconds{quantile="0.9"} 9001
rpc_duration_seconds{quantile="0.99"} 76656
rpc_duration_seconds_sum 1.7560473e+07
rpc_duration_seconds_count 2693
`


**与Prometheus集成**

可以使用Prometheus的static_configs来收集sample_exporter的数据
```
- job_name: "sample"
	static_configs:
	- targets:["127.0.0.1:8080"]

```
重启加载配置，在console查询，就可以看到相关数据。



## Node Exporter

------

node_exporter主要用于*NIX系统监控，用golang编写 


默认开启的功能:

|名称	|说明	|系统|
| :---: | :--- | :--- |
|arp	|从 /proc/net/arp 中收集 ARP 统计信息	|Linux|
|conntrack	|从 /proc/sys/net/netfilter/ 中收集 conntrack 统计信息	|Linux|
|cpu	|收集 cpu 统计信息	|Darwin, Dragonfly, FreeBSD, Linux|
|diskstats	|从 /proc/diskstats 中收集磁盘 I/O 统计信息	|Linux|
|edac	|错误检测与纠正统计信息	|Linux|
|entropy	|可用内核熵信息	|Linux|
|exec	|execution 统计信息	|Dragonfly, FreeBSD|
|filefd	|从 /proc/sys/fs/file-nr 中收集文件描述符统计信息	|Linux|
|filesystem	|文件系统统计信息，例如磁盘已使用空间	|Darwin, Dragonfly, FreeBSD, Linux, OpenBSD|
|hwmon	|从 /sys/class/hwmon/ 中收集监控器或传感器数据信息	|Linux|
|infiniband	|从 InfiniBand 配置中收集网络统计信息	|Linux|
|loadavg	|收集系统负载信息	|Darwin, Dragonfly, FreeBSD, Linux, NetBSD, OpenBSD, Solaris|
|mdadm	|从 /proc/mdstat 中获取设备统计信息	|Linux|
|meminfo	|内存统计信息	|Darwin, Dragonfly, FreeBSD, Linux|
|netdev	|网口流量统计信息，单位 bytes	|Darwin, Dragonfly, FreeBSD, Linux, OpenBSD|
|netstat	|从 /proc/net/netstat 收集网络统计数据，等同于 netstat -s	|Linux|
|sockstat	|从 /proc/net/sockstat 中收集 socket 统计信息	|Linux|
|stat	|从 /proc/stat 中收集各种统计信息，包含系统启动时间，forks, 中断等	|Linux|
|textfile	|通过 --collector.textfile.directory 参数指定本地文本收集路径，收集文本信息	|any|
|time	|系统当前时间	|any|
|uname	|通过 uname 系统调用, 获取系统信息	|any|
|vmstat	|从 /proc/vmstat 中收集统计信息	|Linux|
|wifi	|收集 wifi 设备相关统计数据	|Linux|
|xfs	|收集 xfs 运行时统计信息	|Linux (kernel 4.4+)|
|zfs	|收集 zfs 性能统计信息	|Linux|


默认关闭的功能:

|名称	|说明	|系统|
| :---: | :--- | :--- |
|bonding	|收集系统配置以及激活的绑定网卡数量	|Linux|
|buddyinfo	|从 /proc/buddyinfo 中收集内存碎片统计信息	|Linux|
|devstat	|收集设备统计信息	|Dragonfly, FreeBSD|
|drbd	|收集远程镜像块设备（DRBD）统计信息	|Linux|
|interrupts	|收集更具体的中断统计信息	|Linux，OpenBSD|
|ipvs	|从 /proc/net/ip_vs 中收集 IPVS 状态信息，从 /proc/net/ip_vs_stats 获取统计信息	|Linux|
|ksmd	|从 /sys/kernel/mm/ksm 中获取内核和系统统计信息	|Linux|
|logind	|从 logind 中收集会话统计信息	|Linux|
|meminfo_numa	|从 /proc/meminfo_numa 中收集内存统计信息	|Linux|
|mountstats	|从 /proc/self/mountstat 中收集文件系统统计信息，包括 NFS 客户端统计信息	|Linux|
|nfs	|从 /proc/net/rpc/nfs 中收集 NFS 统计信息，等同于 nfsstat -c	|Linux|
|qdisc	|收集队列推定统计信息	|Linux|
|runit	|收集 runit 状态信息	|any|
|supervisord	|收集 supervisord 状态信息	|any|
|systemd	|从 systemd 中收集设备系统状态信息	|Linux
|tcpstat	|从 /proc/net/tcp 和 /proc/net/tcp6 收集 TCP 连接状态信息	|Linux|

使用--collectors.enabled运行参数指定node_exporter收集的功能模块，不指定则使用默认模块


**程序安装和启动**

+ 下载Node Exporter

+ 启动Node Exporter
```
./node_exporter ...
```


**Docker安装**

```
docker run -d -p 9100:9100 \ 
	-v "/proc:/host/proc:ro" \
	-v "/sys:/host/sys:ro" \
	-v "/:/rootfs:ro" \
	--net="host" \
	quay.io/prometheus/node-exporter \ 
	--collector.procfs /host/proc \
	--collector.sysfs /host/sys \ 
	--collector.filesystem.ignored-mount-points "^/(sys|proc|dev|host|etc)($|/)"
```


**数据存储**

利用Prometheus的static_configs来拉取node_exporter的数据。
```
- job_name: 'node'
  static_configs:
  - targets: ["localhost:9100"]
```
重启加载配置，即可看到node_exporter的数据



## Node Exporter常用语句

------

收集到node_exporter的数据后，可以使用PromQL进行一些业务查询和监控

CPU使用率
```
100 - (avg by (instance) (irate(node_cpu{instance="xxx", mode="idle"}[5m])) * 100)
```

CPU各mode占比率
```
avg by (instance, mode) (irate(node_cpu{instance="xxx"}[5m])) * 100
```

内存使用率
```
100 - ((node_memory_MemFree{instance="xxx"}+node_memory_Cached{instance="xxx"}+node_memory_Buffers{instance="xxx"})/node_memory_MemTotal) * 100
```

磁盘使用率
```
100 - node_filesystem_free{instance="xxx",fstype!~"rootfs|selinuxfs|autofs|rpc_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse.*"} / node_filesystem_size{instance="xxx",fstype!~"rootfs|selinuxfs|autofs|rpc_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse.*"} * 100
```


网络IO
```
//上线带宽
sum by (instance) (irate(node_network_receive_bytes{instance="192.168.1.152:9100",device!~"bond.*?|lo"}[5m])/128)

//下行带宽
sum by (instance) (irate(node_network_transmit_bytes{instance="192.168.1.152:9100",device!~"bond.*?|lo"}[5m])/128)
```


网卡出/入包
```
//入包量
sum by (instance) (rate(node_network_re
ceive_bytes{instance="xxx",device!="lo"}[5m]))

//出包量
sum by (instance) (rate(node_network_transmit_bytes{instance="xxx",device!="lo"}[5m]))
```


### 其他Exporter

+ Memcached exporter负责收集Memcached信息

+ MySQL server exporter负责收集MySQL Server信息

+ MongoDB exporter负责收集MongoDB信息

+ InfluxDB exporter负责收集InfluxDB信息

+ JMX exporter负责收集Java虚拟机信息




## Pushgateway简介

------

Pushgateway是Prometheus生态中一个重要工具，使用它的主要原因是:
+ Prometheus采用pull模式，可能由于不在一个子网或者防火墙内，导致Prometheus无法直接取各个target数据

+ 在监控业务数据的时候，需要将不同数据汇总，由Prometheus统一收集

由于以上原因，不得不使用pushgateway，但在使用之前，有必要了解它的一些弊端:

+ 将多个节点数据汇总到pushgateway，如果挂了，受影响比多个target大

+ Prometheus拉取状态up只针对pushgateway，无法做到对每个节点有效

+ Pushgateway可以持久化推送它的所有监控数据

需要手动清理pushgateway中的数据。


##　Pushgateway安装和使用

**二进制安装**

+ [下载包](https://github.com/prometheus/pushgateway/releases/tag/v0.4.0)

+ 编译，安装

+ 启动服务
cat <<EOF | curl --data-binary @- http://pushgateway.example.org:9091/metrics/job/some_job/instance/some_instance


**Docker安装**

```
docker pull prom/pushgateway
docker run -d -p 9091:9091 prom/pushgateway
```


**数据管理**

正常情况下会使用Client SDK推送数据到pushgateway，还可以通过API来管理，例如：
+ 向{job="some_job"}添加单条数据

```
echo "some_metric 3.14" | curl --data-binary @- http://localhost:9091/metrics/job/some_job
```

+ 添加更多更复杂的数据，通常数据会带上instance，表示来源位置:
```
cat <<EOF | curl --data-binary @- http://pushgateway.example.org:9091/metrics/job/some_job/instance/some_instance
# TYPE some_metric counter
some_metric{label="val1"} 42
# TYPE another_metric gauge
# HELP another_metric Just an example.
another_metric 2398.283
EOF
```

+ 删除某个组下的某实例的所有数据:

```
 curl -X DELETE http://pushgateway.example.org:9091/metrics/job/some_job/instance/some_instance

 ```

+ 删除某个组下的所有数据:
```
curl -X DELETE http://pushgateway.example.org:9091/metrics/job/some_job

```

可以发现pushgateway中的数据通常按照job和instance分组分类，所以这两个参数不可缺少。

因为prometheus配置pushgateway的时候，也会指定job和instance，但是它只表示pushgateway实例，不能真正表达收集数据的含义，所以在prometheus中配置pushgateway的时候，需要添加honor_labels: true,从而避免收集数据本身的job和instance被覆盖。

注意: 为了防止pushgateway重启或意外挂掉，导致数据丢失，可以通过-persistence.file和-persistence.interval参数将数据持久化下来。





## 定义rule规则

Prometheus支持两种类型的规则: recording rules和alerting rules.这些规则可以写在一个文件中，然后在Prometheus的配置文件中指定相应文件位置。

**语法检查**
可以使用promtool来检查rules配置文件中的语法
```
go get github.com/prometheus/prometheus/cmd/promtool
promtool check-rules /path/to/example.rules
```

语法检查正确返回0，标准错误返回1，非法的错误输入参数返回2.

**Recording rules**
Recording rules允许你预先计算所需要的数据或者计算复杂表达式并且把他们的结果作为一个时间序列集合存储，查询预处理结果往往比执行原始的语句更快速，尤其是doshboard，在每次刷新是都要查询同样的语句。

添加一条recording rules的格式:
```
<new time series name[{<label overrides>}] = <expression to record>

// example
# saving the per-job http in process request count as a new set of time series:
job:http_inprocess_requests:sum= sum(http_inprocess_requests) by (job)

# drop or rewrite labels in the result time series
new_time_series{label_to_change="new_value",label_to_drop=""} = old_time_series

```

