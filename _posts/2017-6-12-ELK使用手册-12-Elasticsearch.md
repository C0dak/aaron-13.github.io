# Elasticsearch

------

## 架构原理

**segment，buffer和translog对实时性的影响**

写入的数据是如何变成Elasticsearch里可以被检索和聚合的索引内容的?
从单文件的静态层面看，每个全文检索都是一个词元的倒排索引，具体涉及到全文索引的通用知识，可阅读《Lucene in Action》等书籍

**动态更新的Lucene索引**

以在线动态服务的层面看，要做到实时更新条件下数据的可用和可靠，就需要在倒排索引的基础上，再做一系列更高级的处理。

Lucene的处理办法：**新收到的数据写到新的索引文件里**

Lucene把每次生成的倒排索引，叫做一个段(segment),然后另外使用一个commit文件，记录索引内所有的segment。而生成segment的数据来源，则是内存中的buffer，也就是说，动态更新过程如下:

1. 当前索引有3个segment可用，索引状态如图：

![segment-01.png](https://aaron-13.github.io/images/segment-01.png)

2. 新接收的数据进入内存buffer，索引状态:

![segment-02.png](https://aaron-13.github.io/images/segment-02.png)

3. 内存buffer刷到磁盘，生成一个新的segment，commit文件同步更新，索引状态如图：

![segment-03.png](https://aaron-13.github.io/images/segment-03.png)


**利用磁盘缓存实现的准实时索引**

涉及到磁盘，会有个不可避免的问题：磁盘太慢了！对要求实时性很高的服务来说，这种处理是不够的，所以在第三步的处理中，还有一个中间状态:

1. 内存buffer生成一个新的segment，刷到文件系统缓存中，Lucene即可检索这个新的segment，状态如下:

![segment-04.png](https://aaron-13.github.io/images/segment-04.png)

2. 文件系统缓存真正同步到磁盘上，commit文件更新

这一步刷到文件系统缓存的步骤，在Elasticsearch中，是默认设置为1秒间隔的，对于大多数应用来说，几乎就相当于实时可搜索了。Elasticsearch也提供了单独的`/_refresh`接口，可主动调用该接口保证搜索可见

*5.0中还提供了一个新的请求参数;`?refresh=wait_for`可以在写入数据后不强制刷新但一直等到刷新才返回*

不过对于Elastic Stack的日志场景来说，恰恰相反，并不需要如此高的实时性，而是需要更快的写入性能，一般会通过`/_settings`接口或者定制`template`的方式，加大`refresh_interval`参数:

```
curl -XPOST http://127.0.0.1:9200/logstash-2017-06-21/_settings -d '
{"refresh_interval": "10s"}
'
```

如果是导入历史数据的场合，甚至可以先完全关闭掉:

```
curl -XPUT http://127.0.0.1:9200/logstash-2017-06-21 -d '
{
    "settings": {
        "refresh_interval": "-1"
    }
}'
```


在导入完成后，修改回来或者手动调试一次即可:

```
curl -XPOST http://127.0.0.1:9200/logstash-2017-06-21/_refresh
```


**translog提供的磁盘同步控制**

既然refresh只是写到文件系统缓存，那么第四步写到实际磁盘又是有什么来控制的？如果这期间发生主机错误，硬件故障等，数据会不会丢失？

这里，其实有另一个机制来控制，Elasticsearch在把数据写入到内存buffer的同时，其实还另外记录了一个translog日志

![translog-01.png](https://aaron-13.github.io/images/translog-01.png)

refresh发生的时候，translog日志文件依旧保持原样

![translog-02.png](https://aaron-13.github.io/images/translog-02.png)

在这段时间发生异常，Elasticsearch会从commit位置开始，恢复整个translog文件中的记录，保证数据一致性

等到真正把segment刷到磁盘，且commit文件进行更新的时候，translog文件才清空，这一步，叫做flush。同样，Elasticsearch也提供了`/_flush`接口

对于flush操作，Elasticsearch默认设置为：每30分钟主动进行一次flush，或者当translog文件大小阿玉512MB时，主动进行一次flush，这两个行为，可以分别通过`index.translog.flush_threshold_period`和`index_translog.flush_threshold_size`参数修改

如果对两种控制方式都不满意，Elasticsearch还可以通过`index.translog.flush_threshold_ops`参数，控制每收到多少条数据后flush一次


**translog的一致性**

索引数据的一致性通过translog保证，那么translog文件自己呢？

默认情况下，Elasticsearch每5秒，或者每次请求操作结束前，会强制刷新translog日志到磁盘上。

后者是Elasticsearch2.0新加入的特性，为了保证不丢数据，每次index,bulk,delete,update完成的时候，一定会触发新translog到磁盘上，才给请求返回200 OK。这个改变在提高数据安全性的同时降低了一点性能。

如果不在意这点可能性，还是希望性能优先，可以在index template里设置如下参数:

```
{
    "index.translog.durability": "async"
}
```

## Elasticsearch分布式索引
Elasticsearch为了分布式系统，对一些名词概念做了变动。索引成为了整个集群级别的命名，而在单个主机上的Lucene索引，则被命名为分片(shard)。


## segment merge对写入性能的影响

一句话概括Lucene的设计思路就是"开新文件"。从另一方面，开新文件也会给服务器带来负载压力，因为默认每1秒，都会有一个新文件产生，每个文件都需要文件句柄，内存，CPU使用等各种资源，一天有86400秒，每次请求扫描所有文件，响应性能会很差。

为了解决这个问题，ES会不断在后天运行任务，主动将这些零散的segment做数据归并，尽量让索引内只保有少量的，每个都比较大的segment文件，这个过程是有独立的线程来进行的，并不影响新segment的产生。归并过程中，索引状态如图，尚未完成的较大segment是被排除在检索可见范围之外的:

![segment-05.png](https://aaron-13.github.io/images/segment-05.png)

当归并完成，较大的这个segment刷新到磁盘后，commit文件做出相应变更，删除之前几个小segment，改成新的大segment，等检索请求都从小segment转到大segment上以后，删除没用的小的segment，这时候，索引里的segment数量就下降了。

![segment-06.png](https://aaron-13.github.io/images/segment-06.png)


**归并线程配置**

segment归并的过程，需要先读取segment，归并计算，再写一遍segment，最后要保证刷到磁盘。这是很消耗IO和CPU的任务，所以，ES提供了对归并线程的限速机制，确保这个任务不会过分影响到其他业务。

在5.0之前，归并线程的限速配置`indices.store.throttle.max_bytes_per_sec`是20MB，对于写入量较大，磁盘转速较高，或者使用SSD盘的服务器，这个限速过低了。对于Elastic Stask应用，社区广泛的建议是可以适当调大到100MB或者更高。

```
curl -XPUT http://127.0.0.1:9200/_cluster/settings -d '
{
    "persistent" : {
        "indices.store.throttle.max_bytes_per_sec" : "100mb"
    }
}'
```

5.0开始，ES对此作了大幅度改进，使用了Lucene的CMS(ConcurrentMergeScheduler)的auto throttle机制，正常情况下已经不再需要手动配置，从源码中看到，这个值默认设置是10240MB

归并线程的数目，ES也是有所控制的，默认数目的计算公式是: `Math.min(3,Runtime.getRuntime().availableProcessors() / 2)`。即服务器CPU核数的一半大于3时，启动3个归并线程，否则启动跟CPU核数的一半相等的线程数。

如果磁盘性能跟不上，可以降低`index.merge.scheduler.max_thread_count`配置，免得IO情况恶化。


**归并策略**

归并线程是按照一定的运行策略来挑选segment进行归并的。主要有以下几条:

+ index.merge.policy.floor_segment默认2MB，小于这个大小的segment，优先被归并

+ index.merge.policy.max_merge_at_once默认一次最多归并10个segment

+ index.merge.policy.max_merge_at_once_explicit默认forcemerge时一次最多归并30个segment

+ index.merge.policy.max_merged_segment默认5GB，大于这个大小的segment不用参与归并。forcemerge除外

根据这段策略，可以从另一个角度考虑如何减少segment归并的消耗以及提高响应的办法，加大flush间隔，尽量让每次生成的segment本身大小就比较大。


**forcemerge接口**


默认的最大segment大小是5GB，那么一个比较庞大的数据索引，就必须会有为数不多的segment永远存在，这对文件句柄，内存等资源都是浪费。但是由于归并任务太消耗资源，所以一般不太选择加大`index.merge.policy.max_merged_segment`配置，而是在负载较低的时间段，通过forcemerge接口，强制归并segment

```
curl -XPOST http://127.0.0.1:9200/logstash-2017-06-21/_forcemerge?max_num_segment=1
```

由于forcemerge线程对资源的消耗比普通的归并线程大得多，所以，绝不建议对还在写入数据的热索引执行这个操作。这个问题对于Elastic Stack来说非常好办，一般索引都是按天分割。



## routing和replica的读写过程

------

**路由计算**

作为一个没有额外依赖的简单的分布式方案，ES在这个问题上同样选择了一个非常简洁的处理方式，对任一条数据计算其对应分片的方式如下:

```
shard = hash(routing) % number_of_primary_shards
```

每个数据都有一个routing参数，默认情况下，就是用其`_id`值，将其`_id`值计算哈希后，对索引的主分片数取余，就是数据实际应该存储到的分片ID。


由于取余这个计算，完全依赖分母，所以导致ES索引有一个限制，索引的主分片数，不可以随意修改，因为一旦主分片数不一样，所有数据的存储位置计算结果都会发生变化，索引数据就完全不可读了


**副本一致性**

作为分布式系统，数据副本可算是一个标配。ES数据写入流程，自然也涉及到副本。在有副本配置的情况下，数据从发向ES节点，到接到ES节点响应返回，流向如下:

![routing-01.png](https://aaron-13.github.io/images/routing-01.png)

1. 客户端请求发送给Node1节点，注意图中Node 1是Master节点，实际完全可以不是。

2. Node 1用数据的`_id`取余计算得到应该讲数据存储到shard 0上。通过cluster state信息发现shard 0的主分片已经分配到了Node 3上。Node 1转发请求数据给Node 3

3. node 3完成请求数据的索引过程，存入主分片0，然后并行转发数据给分配有shard 0的副本分片的Node 1和node 2，当收到任一节点汇报副本分片数据写入成功。Node 3即返回给初始的接收节点Node 1，宣布数据写入成功。Node 1返回成欧共响应给客户端。


这个过程中，有几个参数可以用来控制或变更其行为:

+ wait_for_active_shards上面示例中，2个副本分片只要有一个成功，就可以返回给客户端。这点也是有配置项的。其默认值的计算来源:
```
int ((primary + number_of_replicas) / 2) + 1
```

根据需要，可以将参数设置为one，表示仅写完主分片就返回，等同于async；还可以设置为all，表示等所有副本分片写完才返回。

+ timeout如果集群出现异常，有些分片不可用，ES默认会等待1分钟看分片能否恢复，可以使用`?timeout=30s`参数来缩短这个等待时间

副本配置和分片配置不一样，是可以随时调整的。有些较大的索引，甚至可以在做forcemerge前，先把副本全部取消掉，等optimize完后，再重新开启副本，节约单个segment的重复归并消耗。

```
curl -XPUT http://127.0.0.1:9200/lgostash-2017.06.23/_settings -d '{
    "index" : "{"number_of_replicas": 0}"
}'
```


## shard的allocate控制

------

某个shard分配在哪个节点上，一般来说，是由ES自动决定的。以下几种情况会触发分配动作：

1. 新索引生成

2. 索引的删除

3. 新增副本分片

4. 节点增减引发的数据均衡


ES提供了一系列参数详细控制这部分逻辑:

+ `cluster.routing.allocation.enable`该参数用来控制允许分配哪种分片。默认是all，可选项还包括`primaries`和`new_primaries`。none则彻底拒绝分片

+ `cluster.routing.allocation.allow_rebalance`该参数用来控制什么时候允许数据均衡。默认是`indices_all_active`，即要求所有分片都能正常启动成功以后，才可以进行数据均衡操作，否则，在集群重启阶段，会浪费流量。

+ `cluster.routing.allocation.cluster_concurrent_rebalance`该参数用来控制集群内同时运行的数据均衡任务个数，默认是2个，如果有节点增减，且集群负载压力不高，可以适当增大

+ `cluster.routing.allocation.node_initial_primaries_recoveries`该参数用来控制节点重启时，允许同时恢复几个主分片，默认是4个，如果节点是多磁盘，却IO压力不大，可适当加大

+ `cluster.routing.allocation.node_concurrent_recoveries`该参数用来控制节点除了主分片重启恢复以外其他情况下，允许同时运行的数据恢复任务，默认是2个，所以，节点重启时，可以看到主分片迅速恢复完成，副本分片的恢复却很慢，除了副本分片本身数据要通过网络复制以外，并发线程本身也减少一半，当然，这种设置也是有道理的，主分片一定是本地恢复，副本分片要走网络，带宽有限。从ES1.6开始，冷索引的副本分片可以本地恢复，这个参数也就是可以适当加大了。

+ `indices.recovery.concurrent_streams`该参数用来控制节点从网络复制恢复副本切片时的数据流个数，默认是3个，可以配合上一条配置一起加大

+ `indices.recovery.max_bytes_per_sec`该参数用来控制节点恢复时的速率，默认是40MB，建议加大



此外，ES还有一些其他的分片分配控制 策略。比如以tag和rack_id作为区分等。一般来说，Elastic Stack场景中使用不多。比较常见的两种策略有两种:

1. 磁盘限额  为了保护节点数据安全。ES会定时(`cluster.info.update.interval`,默认30秒)检查各节点数据目录磁盘使用情况。在达到`cluster.routing.allocation.disk.watermark.low`(默认85%)的时候，新索引分片就不会再分配到这个节点上。在达到`cluster.routing.allocation.disk.watermark.high`(默认90%)的时候，就会触发该节点现存分片的数据均衡 ，把数据挪到其他节点上去。这两个值不但可以写百分比，还可以写具体字节数。

```
curl -s -XPUT localhost:9200/_cluster/settings -d '{
    "transient" : {
        "cluster.routing.allocation.disk.watermark.low" : "85%",
        "cluster.routing.allocation.disk.watermark.high" : "10gb",
        "cluster.info.update.interval" : "1m"
    }
}'
```

1.热索引分片不均，默认情况下，ES集群的数据均衡策略是以各节点的分片总数(indices_all_active)作为基准的。这对于搜索服务来说无疑是均衡搜索压力提高性能的好方法。
但对于Elastic Stack场景，一般压力集中在新索引的数据写入方面。正常运行的时候，也没有问题，但是当集群扩容的时候，新加入