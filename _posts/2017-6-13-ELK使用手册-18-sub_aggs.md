# 常见sub aggs

------

### 函数调用链堆栈

![aggs-01.png](https://aaron-13.github.io/images/aggs-01.png)

利用Kibana4的sub aggs特性，按照分层次的函数堆栈，逐层做terms agg。千层饼图不能自动深入到函数堆栈的全部层次，需要自己手动指定聚合到第几层


### 分图统计

![aggs-02.png](https://aaron-13.github.io/images/aggs-02.png)

分图的aggs是split chart而不是split slice


### TopN的时序趋势图

TopN的时序趋势图是将Elastic Stack用于监控场景最常用的手段。
![aggs-03.png](https://aaron-13.github.io/images/aggs-03.png)


### 响应时间的百分比趋势图

在时序数据基础上，改变Y轴数据源，选择percentile方式，然后输入具体的四分卫，运行即可
![aggs-04.png](https://aaron-13.github.io/images/aggs-04.png)


### 响应时间的概率分布在不同阶段的相似对比度

采用grouped方式，排列filter aggs的结果

![aggs-05.png](https://aaron-13.github.io/images/aggs-05.png)


