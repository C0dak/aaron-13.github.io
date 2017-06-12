#Linux网络状态监控
------
对Linux的网络状态进行监控是日常的工作之一，也是不可缺少的一环。对网络的状态监控，可以让我们了解到:

> * 当前服务的网络的通畅性
> * 网络的吞吐量
> * 网络带宽的异常情况
> * 网络设备的运作情况...

以下介绍常用的网络相关的命令:

### netstat
netstat是在内核中访问网络及相关信息的程序，能提供TCP连接，TCP和UDP监听，进程内存管理的相关报告。
netstat能够打印网络连接，路由表，接口统计信息，伪装连接和多播成员。

*格式*
netstat [-a][-e][-n][-o][-p program][-r][-s][-t][-l][-u]

	-a 显示所有的socket，包括正在监听的
	-i 显示所有网络接口的信息
	-n 以网络IP地址代替名称，显示出网络连接情形
	-r 显示内核路由表信息
	-t 显示TCP协议的连接情况
	-u 显示UDP协议的连接情况
	-s 显示每种协议的总体状态
	-l 仅显示监听的socket

*常用选项:*
#### netstat -s
	按照各个协议分别显示其统计数据。如应用程序(web浏览器)运行速度慢，不显示web页之类的数据，就可以用本选项来查看所显示的信息。
#### netstat -e
	显示扩展信息
#### netstat -r
	显示路由表信息
#### netstat -a
	显示一个所有的有效连接列表，包括已建立的连接(ESTABLISHED),也包括监听连接请求(LISTENING)
#### netstat -n
	显示所有已建立的有效连接

*常见状态*
![handshake](https://aaron-13.github.io/images/tcp-handshake.png)
**LISTEN**
	侦听TCP端口的连接请求
**SYN-SENT**
	再发送连接请求后等待匹配的连接请求
**SYN-RECEIVED**
	再收到和发送一个连接请求后等待对方对连接请求的确认
**ESTABLISHED**
	连接已建立
**FIN-WAIT-1**
	等待远程TCP连接中断请求，或先前的连接中断请求的确认
**FIN-WAIT-2**
	从远程TCP等待连接中断请求
**CLOSE-WAIT**
	等待从本地用户发来的连接中断请求
**CLOSING**
	等待远程TCP对连接中断的确认
**LAST-ACK**
	等待原来的发向远程TCP的连接中断的确认
**TIME-WAIT**
	等待足够的时间以确保远程TCP接收到连接中断请求的确认，又叫2MSL(Maximum segment lifetime最大生存时间)状态，此状态防止最后一次握手的数据没有传送到对方那里而准备的
**CLOSE**
	没有任何连接状态


### iostat
主要用于监控系统设备的IO负载情况，iostat首次运行时显示自系统启动开始的各项统计信息，之后运行iostat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息

*格式*
iostat [-c][-d][-h][-N][-k][-x][-z][interval [count]]

![iostat](https://aaron-13.github.io/images/iostat-01.png)

`iostat -d -k 2`

![iostat-02](https://aaron-13.github.io/images/iostat-02.png)

参数-d表示，显示设备(磁盘)使用状态；-k某些使用block为单位的列强制使用kilobytes为单位;2表示，数据每隔2秒刷新一次

输出的信息：

	tps:该设备每秒的传输次数(transfer per second)。"一次传输"即一次I/O请求。多个逻辑请求可能被合并为"一次I/O请求"
	kB_read/s：每秒从设备读取的数据量
	kB_wrtn/s：每秒从设备写入的数据量
	kB_read:   读取的数据总量		
	kB_wrtn：  写入的数据总量


因为是瞬间值，总TPS并不严格等于各个分区TPS的总和

iostat还有一个比较常用的选项-x，该选项用于显示和io相关的扩展数据
![iostat-03](https://aaron-13.github.io/images/iostat-03.png)

	rrqm/s:每秒这个设备相关的读取请求有多少被Merge了(当系统调用需要读取数据的时候，VFS将请求发到各个FS，如果FS发现不同的读取请求读取相同block的数据，FS会将这个请求合并Merge)
	wrqm/s:每秒这个设备相关的写入请求有多少被Merge了
	rsec/s:每秒读取的扇区数
	wsec/s:每秒写入的扇区数
	rKB/s: 每秒被发送到设备上的读请求数
	wKB/s: 每秒被写到设备上的写请求数
	avgrq-sz:平均请求扇区的大小
	avgqu-sz:平均请求队列的长度。长度越短越好
	await: 每个IO请求的处理的平均时间(单位微秒毫秒)，可以理解为IO的响应时间，系统IO响应时间应该低于5ms，如果大于10ms就比较大了
	svctm: 平均每次设备I/O操作的服务时间(毫秒时间)。如果svctm和await的值很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，系统上运行的应用程序将变慢
	%util: 在统计时间内所有处理I/O时间，除以总共统计时间。如果统计间隔1秒，该设备有0.8秒在处理IO，而0.2秒闲置，那么该设备的%util = 0.8/1 = 80%。如果是多设备，即使%util是100%，因为磁盘的并发能力，所以磁盘的并发能力，所以磁盘使用未必到瓶颈


iostat还可以用来获取cpu部分的状态值:
![iostat-04](https://aaron-13.github.io/images/iostat-04.png)

**TIPS:**iostat此命令被封装在sysstat包中，可通过`yum install sysstat -y` 进行安装


### iotop
iotop命令是用来监视磁盘I/O使用状况的top类工具。它以表的形式显示当前系统中的进程或线程I/O使用情况，至少CONFIG_TASK_DELAY_ACCT和CONFIG_TASK_IO_ACCOUNTING这两个选项需要在内核编译配置中开启，这两个选项都依赖CONFIGTASKS。

`iotop`
![iotop](https://aaron-13.github.io/images/iotop-01.png)
**选项**

	-o: 只显示有io操作的进程
	-b: 批量显示，无交互，主要用作记录到文件
	-n NUM: 显示NUM次，主要用于非交互操作
	-d SEC: 间隔sec秒显示一次
	-p PID: 监控的进程pid
	-u USER: 监控的进程用户


**常用快捷键**

- 左右箭头: 改变排序方式，默认是按IO排序
- r: 改变排序顺序
- o: 只显示有IO输出的进程
- p: 进程/线程的显示方式切换
- a: 显示累积使用量
- q: 退出	



### iftop
查看实时的网络流量，监控TCCP/IP连接等。监控网卡的实时流量(可以指定网段)，反向解析IP，显示端口信息等
![iftop](https://aaron-13.github.io/images/iftop-01.png)

**界面说明**

	<= =>: 表示流量的方向
	TX: 发送流量
	RX: 接收流量
	TOTAL: 总流量
	Cumm: 运行iftop到目前的总流量
	peak: 流量峰值
	rates: 分别表示2s 10s 40s的平均流量


**相关参数说明**

	-i: 设定检测的网卡
	-B: 以bytes为单位显示流量(默认为bits)
	-n: 使host信息默认直接都显示IP
	-N: 使端口信息默认直接都使用端口号
	-F: 显示特定网段的进出流量
	-h:	帮助，显示参数信息
	-p: 运行于混杂模式
	-P: 显示端口信息
	-f: 过滤计算包
	-m: 设置界面最上边的刻度的最大值，刻度分五个大段显示



