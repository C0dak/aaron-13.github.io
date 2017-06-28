# Visualize功能

------

一个可视化可以基于以下几种数据源类型

+ 一个新的交互式搜索

+ 一个已保存的搜索

+ 一个已保存的可视化


### 创建一个可视化

**第一步：选择可视化类型**

![visualize-01.png](https://aaron-13.github.io/images/visualize-01.png)

已存可视化选择器包括一个文本框用来过滤可视化名称，以及一个指向**对象编辑器(Object Editor)**的链接，可以通过**Settings>Edit saved Objects**来管理已存的可视化。

如果新的可视化是一个Markdown或Timeseries挂件，会直接进入配置界面，其他的可视化类型，选择后都会转到数据源选择


**第二步：选择数据源**

可以选择新建或者读取一个已保存的搜索，作为可视化的数据源，搜索适合一个或者一系列索引相关联的。如果选择了在一个配置了多个索引的系统上开始新搜索，从可视化编辑器的下拉菜单选择一个索引模式。

当从一个已保存的搜索开始创建并保存好了可视化，这个搜索就绑定在这个可视化上，如果修改了搜索，对应的可视化也会自动更新


**第三步：可视化编辑器**

可视化编辑器用来配置编辑可视化，主要有以下几个元素:
1. 工具栏(Toolbar)
2. 聚合构建器(Aggregation Builder)
3. 预览画布(Preview Canvas)

![kibana-02.png](https://aaron-13.github.io/images/kibana-02.png)


**工具栏**
工具栏上有一个用户交互式数据搜索的搜索框，用来保存和加载可视化，因为可视化是基于保存好的搜索，搜索栏会变成灰色。


**聚合构建器**

用页面左侧的聚合构建器配置你的可视化要用的metric和bucket。桶(buckets)的效果类似于SQL group by语句。

在条带图或折线图可视化里，用metric做Y轴，然后buckets做X轴，条带颜色，以及行/列的区分，在饼图里，metrics用来做分片的大小，bucket做分片的数量。

为Y轴选一个metric聚合，包括count，average，sum，min，max or cardinality(unique count),为X轴，条带颜色，以及行/列的区分选一个bucket聚合，常见的有date，histogram，range，terms，filters和significant terms。

可以设置buckets执行的顺序，在Elasticsearch里，第一个聚合决定了后续聚合的数据集。

要看所有相同后缀名的，设置顺序如下：

1. Color: 后缀名的Terms聚合
2. X-Axis: @timestamp的时间条带图

Elasticsearch收集记录，算出前5名后缀名，然后为每个后缀名创建一个时间条带图
要看每个小时的前5名后缀名情况，设置顺序如下:
1. X-Axis: @timestamp的时间条带图
2. Color: 后缀名的Terms集合



### 区块图

这个图的Y轴是数值维度，该维度有以下聚合可用：

+ Count count聚合返回选中索引模式中元素的原始计数

+ Average 这个聚合返回一个数值字段的总和的average，从下拉菜案选择一个字段

+ Sum sum聚合返回一个数值字段的总和

+ Min min聚合返回一个数值字段的最小值

+ Max max聚合返回一个数值字段的最大值

+ Unique Count cardinality聚合返回一个字段的去重数据值

+ Standard Deviation extended stats聚合返回一个数值字段数据的标准差

+ Percentile percentile聚合返回一个数值字段中值的百分比分布

+ Percentile Rank  percentile ranks聚合返回一个数值字段中指定值的百分比排名

可以点击+Add Aggregation按钮添加一个聚合

buckets聚合指明从数据集中将要检索什么信息

图形的X轴是buckets维度，可以为X轴定义buckets，同样还可以为图片上的分片区域，或者分割的图片定义buckets

该图形的X轴支持以下聚合：

+ Date Histogram 基于数值字段创建，由时间组织起来，可以指定时间片的间隔

+ Histogram 标准histogram基于数值字段创建，为这个字段指定一个整数间隔。勾选`Shown empty buckets`让直方图中包含空的间隔。

+ Range 通过range聚合，可以为一个数值字段指定一系列区间。点击Add Range添加一对区间端点

+ Date Range 聚合计算指定的时间区间内的值。可以使用date math表达式指定区间。

+ IPv4 Range 聚合用来指定IPv4地址的 区间

+ Terms 聚合允许指定展示一个字段的首尾几个元素，排序方式可以是计数或者其他自定义的metric

+ Filters 为数值指定一组filters。可以用query string，也可以用JSON格式来指定过滤器

+ Significant Terms展示实验性的significant terms聚合的结果


一旦定义好一个X轴聚合，可以继续定义自聚合来完善可视化效果。点击+ Add Sub Aggregation添加自聚合，选择Split Area或者Split Chart，然后从类型菜单中选择一个子聚合

当一个图形中定义了多个聚合，可以使用聚合类型右侧上上下箭头来改变聚合的优先级。

Kibana4.5以后新增了两个功能: 在Custom Label里填写自定义字符串和颜色编辑器

点击Advanced链接显示更多有关聚合的自定义参数:
+ Exclude Pattern指定一个从结果集中排除掉的模式

+ Exclude Pattern Flags排除模式的Java flags标准集

+ Include Pattern 指定一个从结果集中要包含的模式

+ Include Pattern Flags 包含模式的Java flags标准集

+ JSON Input一个用来添加JSON格式属性的文本框，内容会合并进聚合的定义中
```
{"script":"doc['grade'].value * 1.2"}
```

**注意**
Elasticsearch 1.4.3及以后版本，这个函数需要开启 [dynamic Groovy scripting](http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)

这个参数是否可用，依赖于选择的聚合函数:

选择view options更改表格中如下方面:

+ Chart Mode当你为图形定义了多个Y轴，可以用该下拉菜单定义聚合如何显示在图形上

	+ stacked聚合结果一次叠加在顶部

	+ overlap聚合 结果重叠的地方采用半透明效果

	+ wiggle聚合结果显示成streamgraph效果

	+ percentage显示每个聚合在总数中的百分值

	+ silhouette显示每个聚合距离中间线的方差


多选框可以用来控制以下行为：

+ Smooth Lines勾选该项平滑数据点之间的折线成曲线

+ Set Y-Axis Extents勾选选项，然后在y-max和y-min框里输入数值限定Y轴为指定数值

+ Scale Y-Axis to Data Bounds默认的Y轴长度为0到数据集的最大值

+ Show Tooltip勾选选项显示工具栏

+ Show Legend勾选该项在图形右侧显示图例




### 数据表格

+ Count count聚合返回选中索引模式中元素的原始计数

+ Average 这个聚合返回一个数值字段的总和的average，从下拉菜案选择一个字段

+ Sum sum聚合返回一个数值字段的总和

+ Min min聚合返回一个数值字段的最小值

+ Max max聚合返回一个数值字段的最大值

+ Unique Count cardinality聚合返回一个字段的去重数据值

+ Standard Deviation extended stats聚合返回一个数值字段数据的标准差

+ Percentile percentile聚合返回一个数值字段中值的百分比分布

+ Percentile Rank  percentile ranks聚合返回一个数值字段中指定值的百分比排名

可以点击+Add Aggregation按钮添加一个聚合

buckets聚合指明从数据集中将要检索什么信息

数据表格每行，叫做buckets，可以定义buckets来切割表格成行，或者切割表格成另一个表格。


每个bucket类型都支持以下聚合：

+ Date Histogram 基于数值字段创建，由时间组织起来，可以指定时间片的间隔

+ Histogram 标准histogram基于数值字段创建，为这个字段指定一个整数间隔。勾选`Shown empty buckets`让直方图中包含空的间隔。

+ Range 通过range聚合，可以为一个数值字段指定一系列区间。点击Add Range添加一对区间端点

+ Date Range 聚合计算指定的时间区间内的值。可以使用date math表达式指定区间。

+ IPv4 Range 聚合用来指定IPv4地址的 区间

+ Terms 聚合允许指定展示一个字段的首尾几个元素，排序方式可以是计数或者其他自定义的metric

+ Filters 为数值指定一组filters。可以用query string，也可以用JSON格式来指定过滤器

+ Significant Terms展示实验性的significant terms聚合的结果

+ Geohash 聚合显示基于地理坐标的点


一旦定义好一个X轴聚合，可以继续定义自聚合来完善可视化效果。点击+ Add Sub Aggregation添加自聚合，选择Split Area或者Split Chart，然后从类型菜单中选择一个子聚合

当一个图形中定义了多个聚合，可以使用聚合类型右侧上上下箭头来改变聚合的优先级。

Kibana4.5以后新增了两个功能: 在Custom Label里填写自定义字符串和颜色编辑器

点击Advanced链接显示更多有关聚合的自定义参数:
+ Exclude Pattern指定一个从结果集中排除掉的模式

+ Exclude Pattern Flags排除模式的Java flags标准集

+ Include Pattern 指定一个从结果集中要包含的模式

+ Include Pattern Flags 包含模式的Java flags标准集

+ JSON Input一个用来添加JSON格式属性的文本框，内容会合并进聚合的定义中
```
{"script":"doc['grade'].value * 1.2"}
```

选择view options更改表格中如下方面:

+ Per Page这个输入框控制表格的翻页，默认值是每页10行

多选框用来控制以下行为

+ Show metrics for every bucket/level勾选此项用以显示每个bucket聚合的中间结果

+ Show partial rows勾选此项显示没有数据的行




### Lines Charts

...

**气泡图(Bubble Charts)**

通过以下步骤，可以转换折线图成气泡图
1. 为Y轴点击Add metircs，选择Dot size
2. 从下拉框选择一个metric聚合函数
3. 在Options标签里，去掉Show Connection Lines的勾选
4. 点击Apply changes按钮

![line-option.png](https://aaron-13.github.io/images/line-option.png)



### Markdown挂件


### Metric

metric可视化为你选择的聚合显示一个单独的数字
...


### 饼图

饼图的分片大小通过metrics聚合定义，这个维度可以支持以下聚合

+ Count 聚合返回选中索引模式中的元素的原始计数

+ Sum 聚合返回一个数值字段的总和

+ Unique Count cardinality 聚合返回一个字段的去重数据值



### 瓦片地图

瓦片地图显示一个圆圈覆盖的地理区域，这些圆圈是由指定的buckets控制的
瓦片地图的默认metrics聚合是count聚合。

...


## 竖条图

这个图的Y轴是数值维度。



## dashboard

一个Kibana dashboard能自由排列一组已保存的可视化，保存这个仪表板，用来分享或者重载。

![dashboard-02.png](https://aaron-13.github.io/images/dashboard-02.png)



## timelion介绍

在5.0版本中，timelion称为kibana5默认分支的一个插件

![timelion.png](https://aaron-13.github.io/images/timelion.png)


**可视化效果**

+ `.bars($width)`: 用柱状图展示数组

+ `.lines($width,$fill,$show,$steps)`: 用折线图展示数组

+ `.points()`: 用散点图展示数组

+ `.color('#c6c6c6')`: 改变颜色

+ `.hide()`: 隐藏数组

+ `.label("change from %s")`: 标签

+ `.legend($position,$column)`: 图例位置

+ `.static(value=1024,label="1k",offset="-1d",fit="scale")`: 在图形上绘制一个固定值

+ `.value()`: .static()的简写

+ `.title(title="qps")`: 图标标题

+ `.trend(mode="linear",start=0,end=-10)`: 采用linear或log回归算法绘制趋势图

+ `.yaxis($yaxis_number,$min,$max,$position)`: 设置Y轴属性，.Yaxis(2)表示第二根Y轴



**数据运算类**

+ .abs(): 对整个数组元素求绝对值

+ .precision($number): 浮点数精度

+ .cusum($base): 数组元素之和，再加上$base

+ .derivative(): 对数组求导数

+ .divide($divisor): 数组元素除法

+ .multiply($multiplier): 数组元素乘法

+ .subtract($term): 数组元素减法

+ .sum($term): 数组元素加法

+ .add(): 同.sum()

+ .plus(): 同.sum()

+ .first(): 返回第一个元素

+ .movingaverage($window): 用指定的窗口大小计算移动平均值

+ .mvavg(): .movingaverage()的简写

+ .movingstd($window): 用指定的窗口大小计算移动标准差

+ .mvstd(): .movingstd()的简写

+ .fit($mode): 使用指定的fit函数填充空值，average，carry，nearest，none，scale

+ .hot(alpha=0.5,beat=0.5,gamma=0.5,seaon="1w",sample=2): 即Elasticsearch的pipeline aggregation所支持的holt-winters算法

+ .log(base=10): 对数

+ .max(): 最大值

+ .min(): 最小值

+ .props(): 附加额外属性，比如.props(label=bears!)

+ .range(max=10,min=1): 保持形状的前提下，修改最大值和最小值

+ .scale_interval(interval="1s"): 在新间隔下再次统计

+ .trim(start=1,end=-1): 裁剪序列值


**逻辑运算类**

+ .condition(operator="eq",if=100,then=200): 支持eq，ne，lt，gt，lte，gte等操作以及if，else，then赋值

+ .if(): .condition()缩写


**数据源设定类**

+ .elasticsearch(): 从ES读取数据

+ .es(q="querystring",metirc="cardinality",index="logstash-*",offset="-1d"): .elasticsearch()的简写

+ .graphite(metric="path.to.*.data",offset="-1d"): 从graphite读取数据

+ .quandl(): 从quandl.com读取quandl码

+ .worldbank_indicators(): 从worldbank.org读取国家信息

+ .wbi(): .worldbank_indicators()的简写

+ .worldbank(): 从worldbank.org读取数据

+ .wb(): .worldbank()的简写


以上所有函数，都在kibana/src/core_plugins/timelion/server/series_functions目录下实现，每个js文件实现一个TimelionFunction功能


## Console

原先归属于Marvel的Sense插件，改名为Kibana console应用，在5.0版本中默认随Kibana分发。

console应用的操作界面和原先的sense几乎一样

![console-01.png](https://aaron-13.github.io/images/console-01.png)



## setting功能

要使用kibana，就要知道elasticsearch索引是哪些，这就要配置一个或者更多的索引模式，此外，还可以:
+ 创建脚本化字段，这个字段可以实时从数据中计算出来。可以浏览这种字段，并在此基础上做可视化，但是不能搜索这种字段

+ 设置高级选项，比如表格里显示多少行，常用字段显示多少个。修改高级选项要小心，因为一个设置可能与另一个设置不兼容

+ 为生产环境配置Kibana


**创建一个连接到Elasticsearch的索引模式**

一个索引模式定义了一个或者多个打算探索的Elasticsearch索引。Kibana会查找匹配指定模式的索引名，模式中的通配符(*)匹配零到多个字符。

索引模式也可以简单的设置为一个单独的索引名字

要创建一个连接到Elasticsearch的索引模式:

1. 切换到`Settings > Indices`标签页

2. 指定一个能匹配Elasticsearch索引名的索引模式，默认的，Kibana会假设要处理Logstash导入的数据


日期格式码


设置默认索引模式


重新加载索引的字段列表


删除一个索引模式


**创建一个脚本化字段**
脚本化字段是从Elasticsearch索引数据中即时计算出来的。在Discover标签页，脚本化字段数据会作为文档数据的一部分显示。
即时计算脚本化字段非常耗资源，直接影响kibana性能。
脚本化字段使用[Lucene表达式语法]http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-scripting.html#_lucene_expressions_scripts)

要创建一个脚本化字段:

1. 进入`Settings > Indices`
2. 选择打算添加脚本化字段的索引模式
3. 进入模式的`Scriptd Field`标签
4. 点击`Add Scripted Field`
5. 输入脚本化字段的名字
6. 输入用来即时计算数据的表达式
7. 点击`Save Scripted Field`

有关Elasticsearch的脚本化字段的更多[细节](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-scripting.html)



**设置Kibana服务器属性**

Kibana服务器在启动的时候会从kibana.yml文件读取属性设置，默认设置是运行砸localhost:5601,要变更主机或端口，或者连接远端主机上的Elasticsearch，都需要更新kibana.yml文件。

kibana服务器属性：

| 属性 | 描述|
| :--- | :---: |
| port | kibana服务器运行的端口。默认 port:5601|
| host | kibana服务器监听的地址。默认 host: 0.0.0.0 |
| elasticsearch_url | 请求的索引存在哪个Elasticsearch实例上 |
| elasticsearch_preserve_host | 默认的，浏览器请求中的主机名即作为kibana发送给Elasticsearch时请求的主机名 |
| kibana_index | 保存搜索，可视化，仪表盘信息的索引名字，默认: .kibana |
| default_app_id | 进入kibana是默认显示页面。可以为discover visualize，dashboard或settings |
| request_timeout | 等待Kibana后端或Elasticsearch的响应超时时间，单位毫秒。默认: 500000 |
| shard_timeout | Elasticsearch等待分片响应的超时时间，设置为0表示关闭超时控制，默认0 |
| ssl_key_file | kibana服务器的秘钥文件路径，设置用来加密浏览器和Kibana之间的通信，默认none |
| ssl_cert_file | Kibana服务器的证书文件路径，设置用来加密浏览器和Kibana之间的通信。默认none |
| pid_file | 指定进程ID文件的位置，没有指定则存储在/var/run/kibana.pid，默认none |


