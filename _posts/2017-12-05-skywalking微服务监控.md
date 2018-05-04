# PinPoint

------

[PinPoint](https://github.com/naver/pinpoint)是APM(Application Performance Managment)的一种工具。
现在APM体系，基本都是参考Google的Dapper(大规模分布式系统的跟踪系统)的体系实现的。
PinPoint
	通过JavaAgent的机制来做字节码代码植入，实现假如traceid和抓取性能数据的目的

同类工具:
[google的Dapper](http://bigbully.github.io/Dapper-translation/)

[twitter的Zipkin](https://github.com/openzipkin/zipkin)

淘宝的鹰眼(EgleEye)

[大众点评的CAT](https://github.com/dianping/cat)

[华为的Skywalking](https://github.com/OpenSkywalking/skywalking)https://github.com/OpenSkywalking/skywalking

针对PHP应用提供APM能力的工具
Xhprof/Xhgui
[Xhprof](https://github.com/preinheimer/xhprof)
[Xhgui](https://github.com/perftools/xhgui)

PinPoint:基本不用修改源码和配置文件，只要在启动命令里指定javaagent参数即可
Zipkin：需要对Spring，web.xml之类的配置文件做修改，相对麻烦一些
CAT:因为需要修改源码设置埋点，对代码有侵入性


Pinpoint跟踪模块之间的调用流并提供清晰的视图来定位问题区域和潜在瓶颈
+ 服务器地图(ServerMap)
	通过可视化分布式系统的模块和他们之间的相互联系来理解系统拓扑。点击某个节点会展示这个模块的详情，比如当前的状态和请求数量。

+ 实时活动线程图表(Realtime Active Thread Chart)
	实时监控应用内部的活动内容

+ 请求/应答分布图表(Request/Response Scatter Chart)
	长期可视化请求数量和应答模式来定位潜在问题。通过在图标上拉拽可以选择请求查看更多的详细信息
![pinpoint-01.png](https://aaron-13.github.io/images/pinpoint-01.png)	

+ 调用栈(CallStack)
	在分布式环境中为为每个调用生成代码级别的可视图，在单个视图中定位瓶颈和失败点。
![pinpoint-02.png](https://aaron-13.github.io/images/pinpoint-02.png)	

+ 巡查(Inspector)
	查看应用上的其他详细信息，比如CPU使用率，内存/垃圾回收，TPC和JVM参数。
![pinpoint-03.png](https://aaron-13.github.io/images/pinpoint-03.png)	


**架构**
![pinpoint-04.png](https://aaron-13.github.io/images/pinpoint-04.png)

**支持模块**
![pinpoint-05.png](https://aaron-13.github.io/images/pinpoint-05.png)


## 安装
Pinpoint有三个主要组件(collector,web,agent),并使用HBase作为存储。Collector和Web被打包为单个war文件，而agent被打包以便可以作为java agent附加到应用

### 要求
+ JDK1.6
+ JDK1.8
+ MAVEN3.2+
+ 环境变量JAVA_6_HOME，JAVA_7_HOME，JAVA_8_HOME

开始
```
git clone https://github.com/naver/pinpoint.git
cd pinpoint
mvn clean install -Dmaven.test.skip=true
```

安装并启动HBase
1.下载apache官网中hbase相关包解压并启动服务
	hbase中自带zookeeper服务，启动完成后可通过jps进行查看

2.创建Schema，pinpoint有两个脚本可供执行: hbase-create.hbase和hbase-create-snappy.hbase,如果使用hbase-create-snappy.hbase脚本，需要snappy组件的支持，否则就使用hbase-create.hbase。
```
$HBASE_HOME/bin/hbase shell hbase-create.hbase
```

在创建过程中报错:client.ConnectionManager$HConnectionImplementation: The node /hbase is not in ZooKeeper. It should have been written by the master. Check the value configured in 'zookeeper.znode.parent'. There could be a mismatch with the one configured in the master.
这个报错是由于没有配置hbase的rootdir所导致的。
修改conf目录下的hbase-site.xml文件
![hbase-01.png](https://aaron-13.github.io/images/hbase-01.png)

可以通过http://localhost:16010页面查看hbase的状态信息



## 部署Pinpoint
1.部署Pinpoint-collector
将collector的war包放到tomcat的webapps目录下，并重命名为ROOT.war。修改ROOT/WEB-INF/classes/hbase.properities，将:
```
hbase.client.host=localhost
hbase.client.port=2181
```
修改为zk的地址，然后重启服务

2.部署Pinpoint-web
操作和collector一样

3.部署Pinpoint-agent
拷贝agent的压缩包到相应的服务器上并进行解压缩。配置pinpoint.config文件将profiler.collector.ip=127.0.0.1改为相应collector对应的地址。
安装collector启动后，自动就开启9994,9995,9996的端口，如需修改，可编辑collector的配置文件pinpoint-collector.properties，进行修改。

4.部署应用(探针)
如果是tomcat应用，修改bin/catalina.sh文件
```
AGENT_VERSION="1.6.2"
AGENT_PATH="/data/x/app/apm/pinpoint"
AGENT_ID="app_test_`date +%Y%m%d`"
APPLICATION_NAME="app_name"
CATALINA_OPTS="$CATALINA_OPTS -javaagent:$AGENT_PATH/pinpoint-bootstrap-$AGENT_VERSION.jar"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=$AGENT_ID"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=$APPLICATION_NAME"
```

如果是springboot包部署
```
nohup java -javaagent:/path/to/pinpoint-bootstrap-1.6.2.jar -Dpinpoint.agentId=app_name_01 -Dpinpoint.applicationName=app_name -jar myapp.jar &
```

**agent默认输出的日志是DEBUG级别的，通过降低级别来减少日志输出量，修改agent/lib/log4j.xml中的日志参数，将日志级别调制INFO**


github地址:[pinpoint](https://github.com/naver/pinpoint)


带外数据跟踪收集
tip1: 带外数据:传输层协议使用带外数据(out-of-band,OOB)来发送一些重要的数据，如果通信一方有重要的数据需要通知对方时，协议能够将这些数据快速发送到对方，为了发送这些数据，协议一般不使用与普通数据相同的童丹，而是使用另外的通道。

tip2: in-band策略是把跟踪数据随着调用链进行传送，out-of-band是通过其他的链路进行跟踪数据的收集，Dapper的写日志进行日志采集的方式就属于out-of-band策略

Dapper系统请求树树自身进行跟踪记录和收集带外数据。这样做为两个不相关的原因。首先，带内收集方案--这里跟踪数据会以RPC响应头的形式返回--会影响应用程序网络动态。RPC回应大小可能接近大型分布式的跟踪的根节点这种情况。在这种情况下，带内Dapper的跟踪数据会让应用程序数据和倾向于使用后续分析结果的数据量相形见绌。其次，带内收集方案假定所有的RPC是完美嵌套的，在所有后端的系统返回的最终结果之前，有很多中间件会把结果返回给他们的调用者。带内收集系统是无法解释这种非嵌套的分布式执行模式的。

Google的Dapper
[Dapper](http://bigbully.github.io/Dapper-translation/)