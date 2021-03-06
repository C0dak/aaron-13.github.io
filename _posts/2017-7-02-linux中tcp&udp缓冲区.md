# TCP/UDP缓冲区问题

------

1. 发送缓冲区问题

TCP：每个TCP套接字都有一个发送缓冲区，可以用SO_SNDBUF套接口选项来改变这一缓冲区大小。
当某个应用进程调用write往套接字写数据时，内核从应用进程缓冲区拷贝所有数据到套接口的发送缓冲区，如果套接口发送缓冲区容不下应用程序的所有数据，或者是应用进程的缓冲区大于套接口的发送缓冲区，或者是套接口的发送缓冲区有别的数据，应用进程将被挂起。内核将不从write返回。直到应用进程缓冲区的所有数据都拷贝到套接口发送缓冲区。
所以，从写一个TCP套接口的write调用成功返回仅仅表示我们可以重新使用应用进程缓冲区，它并不是告诉我们对方接收到数据。TCP发给对方的数据，对方在收到数据时必须给予确认，只有在收到对方的确认时，本方TCP才会把TCP发送缓冲区中的数据删除。

UDP：UDP因为是不可靠连接，不必保存应用进程的数据拷贝，应用进程中的数据在延协议栈向下传递时，以某种形式拷贝到内核缓冲区，当数据链路层把数据传出去后就把内核缓冲区中的数据拷贝删除。因为它不需要一个真正意义上的发送缓冲区，但是任何UDP套接字还是有发送缓冲区的，并且可以使用SO_SNDBUF套接字选项更改它，不过它仅仅是可写到该套接字的UDP数据包的大小上限。
如果一个应用进程写一个大于套接字发送缓冲区大小的数据包，内核将返回该进程一个EMSGSIZE错误。


2.接受缓冲区

TCP: 对于TCP来说，套接字接收缓冲区中可用空间的大小限定TCP通告对端的窗口大小，TCP套接字的接收缓冲区不可能溢出，因为不允许对端发出超过本端所通告窗口大小的数据，这就是TCP的流量控制。如果对端无视窗口大小而发出超过该窗口大小的数据，本端TCP将丢弃它们。


UDP: 当接收到的数据包装不进套接字接收缓冲区时，该数据包就被丢弃。UDP是没有流量控制的，较快的发送端可以很容器地淹没较慢的接收端，导致接收端的UDP丢弃数据包。


+ 因为TCP的窗口规模选项是在建立连接时用SYN分节与对端互换得到的，所以函数的调用顺序很重要：对于客户，SO_RCVBUF选项必须在调用connect之前设置，对于服务器，该选项必须在调用listen之前给套接字设置

+ 关于缓冲区大小的设置范围是有规定的



UDP缓冲区的大小主要和一下几个值有关：

1. /proc/sys/net/core/rmem_max: UDP缓冲区的最大值，单位字节

2. /proc/sys/net/core/rmem_default: UDP缓冲区的默认值，如果不更改的话程序的udp缓冲区默认值就是这个

查看方法可以直接cat以上两个文件进程查看，也可以通过sysctl查看

sysctl -a | grep rmem_max

其实sysctl信息来源就是proc下的文件


**更改udp缓冲区大小**

程序中进行更改：
程序中可以使用setsockopt函数与SO_RCVBUF选项对udp缓冲区的值进行更改，但是要注意不管设置的值有多大，超过rmem_max的部分会被无视掉。

```
int a = value_wanted;
if (setsockopt(sockfd,SOL_SOCKET,SO_RCVBUF,$a,sizeof(int)) == -1) {
	...
}

```

**更改系统值**

如果确实要把udp缓冲区改到一个比较大的值，那就需要更改rmem_max的值，编译/etc/rc.local文件添加以下代码可使系统在每次启动的时候自动更改系统缓冲区的最大值。

`echo value_wanted > /proc/sys/net/core/rmem_default`

或者在/etc/sysctl.conf添加以下代码即可在重启后永久生效

rmem_max = MAX

不想重启的话使用命令sysctl -p 即可

可以顺便看下setsockopt在linux下的实现：

```
...
case SO_SNDBUF:
if (val > sysctl_wmem_max)
val = sysctl_wmem_max
if ((val * 2) < SOCK_MIN_SNDBUF)
sk -> sk_sndbuf = SOCK_MIN_SNDBUF;
else
sk -> sk_sndbuf = val *2;
//当然缓冲区在系统中实际值要大一点，因为upd报头以及IP报头等都是需要空间的
...
```



