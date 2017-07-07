# nginx并发连接数限制

------

nginx关于并发连接数限制所使用的模块是[ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)这个模块。

有五个可选的配置参数:

+ limit_conn

+ limit_conn_log_level

+ limit_conn_status

+ limit_conn_zone

+ limit_zone

并不是所有的连接都被计算。一个连接只在当它有一个被服务端处理的请求并且整个请求头已经被读取的时候，这个连接才被计数。

```
http {
	limit_conn_zone $binary_remote_addr zone=addr:10m;
	...
	server {
		...
		location /download/ {
			limit_conn addr 1;
		}
	}
}
```

```
Systax: limit_conn zone number;
Default: - 
Context: http,server,location
```

设置共享内存区域和给定键值所允许的最大连接数。当超过这个连接数，服务器会返回503错误(Server Temporarily Unavailable)

下面的配置用来限制连接到服务器上的每个IP的连接数和总的连接到服务器上的连接数

```
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;
server {
	...
	limit_conn perip 10;
	limit_conn perserver 100;
}
```

![limit-01.png](https://aaron-13.github.io/images/limit-01.png)

设置日志级别

![limit-02.png](https://aaron-13.github.io/images/limit-02.png)

设置拒绝请求返回的响应码

![limit-03.png](https://aaron-13.github.io/images/limit-03.png)

以指定的键的值大小，设置共享存储内存空间的名称和大小

limit_conn_zone $binary_remote_addr zone=addr:10m;
这里，设置客户端的IP地址作为键。注意，这里使用的是$binary_remote_addr变量，而不是$remote_addr变量。
$remote_addr变量的长度为7字节到15字节不等。而存储状态在64平台中占用64字节。而$binary_remote_addr变量的长度是固定的4字节，存储状态在64位平台占用64字节，一兆字节的共享内存空哦见可以保存32万个32位的状态，1.6万个64位的状态。

![limit-04.png](https://aaron-13.github.io/images/limit-04.png)

该指令已经在1.7.6版本被移除，使用limit_conn_zone来替代



# Tomcat

------

1. Tomcat的外部调优

Java虚拟机(JVM)性能优化，可以通过以下两个参数来设置，-Xms<size>(JVM初始化堆的大小)和 -Xmx<size>(JVM堆的最大值)。如果虚拟机启动时设置使用的内存比较小而在这种情况下有许多对象进行初始化，虚拟机就必须重复增加内存来满足使用。由于这种原因，一般把-Xms和-Xmx设为一样大，而堆得最大值受限于物理内存的大小。
Tomcat默认可以使用的内存为128MB。
另外需要考虑的是Java提供的垃圾回收机制。虚拟机的堆大小决定了虚拟机花费在收集垃圾上的时间和频度。收集垃圾可以接受的速度与应用相关，应该通过分析实际的垃圾收集的时间和频率来调整。如果堆的大小很大，那么完全垃圾收集就会很慢，但是频度会降低。如果堆的大小和内存的需要一致，完全收集就很快，但是会更加频繁。调整堆大小的目的就是最小化垃圾收集的时间，以在特定的时间内最大化处理客户的请求。当增加处理器时，记得增加内存，因为分配可以并行进行，而垃圾收集不是并行的。

2. Tomcat的内部调用

**禁止DNS查找**
记录客户端的信息，两种方式，一个是记录客户端的数字ip地址，另一个是在DNS数据中查找真是的主机名。DNS查找会增加网络通信，以致造成了网络延迟。要消除这个延迟，可以禁掉DNS查找。这时，再调用getRemoteHost()方法时，就只会得到数字IP地址。这个配置是在Tomcat的server.xml文件中，Connector对象的enableLookups属性，如下:

```
<Connector
	minSpareThreads: 制定最少运行的线程数，如果没有指定，默认是10个，如果设置为-1表示清除不使用
	maxThreads：能被该connector创建的最大请求线程数，如果没有指定，默认200个。-1表示清除不使用
	acceptCount：当所有请求线程在使用中时，还能接入连接的最大队列长度，默认是100
	redirectport: 当请求为https时，把该请求转发到8443端口
	enableLookups：可以将IP地址转换为域名，影响性能，设为false
	connectionTimeout：定义建立客户连接超时的时间，-1表示不限制建立客户连接时间
	maxConnections：服务器接收和处理的最大连接数。NIO和NIO2默认是10000，APR/原生默认是8192
/>

```

**调整线程数**


**加速JSP的编译**


