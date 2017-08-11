# Burp Suite

------

## Burp Suite安装和环境配置

Burp Suite是一个集成化的渗透测试工具，集合了多种渗透测试组件，能更好的完成对web应用的渗透测试和攻击。

[免费版下载地址](https://portswigger.net/burp/downloadfree.html)

Burp Suite的基本配置，包含以下内容:
+ 如何从命令行启动Burp Suite

+ 如何设置JVM内存大小

+ IPv6问题调试


**如何从命令行启动Burp Suite**
1.配置Java环境

2.设置JVM内存大小
java -jar Xmx2048M /path/burpsuite.jar
Xmx:指定JVM可用的最大内存


**IPv6问题调试**

Burp Suite不支持IPv6地址进行数据通信，修改启动脚本，添加对IPv4的制定后，重启Burp Suite即可:

```
java -Xmx2048M -Djava.net.perferIPv4Stack=true -jar /path/burpsuite.jar
```


## Burp Suite代理和浏览器设置
Burp Suite代理工具是以拦截代理的方式，拦截所有通过代理的网络流量，如客户端的请求数据，服务器端的返回信息等。Burp Suite主要拦截http和https协议的流量，通过拦截，Burp Suite以中间人的方式，可以对客户端请求数据，服务端返回做各种处理，达到安全评估的目的。

当Burp Suite启动之后，默认分配的代理地址是127.0.0.1:8080,可以在Burp Suite的proxy选项中的options上查看

设置对应浏览器的代理服务器地址



## 使用Burp Suite代理

Burp Suite是BurpSuite以用户驱动测试流程功能的核心，通过代理模式，可以拦截，查看，修改所有在客户端和服务端之间传输的数据。

+ Burp Proxy的基本使用

+ 数据拦截与控制

+ 可选项配置Options

+ 历史记录History


**Burp Proxy基本使用**

一般使用Burp Proxy时，大体涉及环节如下:
1.首先，确认JRE环境，Burp Suite可以启动并正常运行，切已经完成浏览器的代理服务器配置

2.打开Proxy功能中的Intercept选项卡，确认拦截功能为"Interception is on"状态

3.打开浏览器，输入URL，就会看到数据流量经过Burp Proxy并暂停，直到点击【Forward】，才会继续传输下去。

4.点击【Forward】就能看到这此请求返回的所有数据

5.当Burp Suite拦截的客户端和服务器交互之后，可以在Burp Suite的消息分析选项卡中查看这次请求的实体信息，消息头，请求参数等信息。消息分析选项主要包括:

+ Raw: 显示web请求的raw格式

+ Params: 显示客户端请求的参数信息，包括GET，POST，Cookie参数

+ Headers: 显示和Raw的信息类似

+ Hex: 显示Raw的二进制内容


默认情况下，Burp Proxy只拦截请求的消息，普通文件请求如css，js，图片是不会被拦截的，可以通过修改默认拦截选项来进行拦截。


**数据拦截与控制**
Burp Proxy的拦截功能主要由Intercept选项卡中的Forward，Drop，Interception is on/off，Action
+ Forward: 当你查看过消息或者重新编辑消息之后，点此按钮，将发送消息至服务器端。

+ Drop: 丢失当前拦截的消息，不再forward到服务端

+ Interception is on表示拦截功能打开

+ Action: 除了将当前请求的消息传递到Spider，Scanner，Repeater，Intruder，Sequencer，Decoder，Comparer组件外，还可以做一些请求消息的修改，如改变GET或者POST请求方式...



**可选项配置Options**

+ 客户端请求消息拦截

+ 服务端返回消息拦截

+ 服务端返回消息修改

+ 正则表达式配置

+ 其他配置项


客户端请求消息拦截

![burp-01.png](https://aaron-13.github.io/images/burp-01.png)

主要包含拦截规则配置，错误消息自动修复，自动更新Content-Length消息头三部分：
1.Intercept requests based on the following rules:对请求消息拦截时，对规则的过滤是自上而下的。
![burp-02.png](https://aaron-13.github.io/images/burp-02.png)

2.automatically fixmissing:会自动修复丢失或者多余的新行

3.automatically update Content-Length:当请求的消息被修改后，Content-Length消息头部也会自动被修改，替换为与之相对应的值。


服务器端返回消息拦截

![burp-03.png](https://aaron-13.github.io/images/burp-03.png)

![burp-04.png](https://aaron-13.github.io/images/burp-04.png)
+ 显示form表单中隐藏字段
+ 高亮显示form表单中隐藏字段
+ 使form表单中的disable字段生效，变成可输入域
+ 移除输入域长度限制
+ 移除JavaScript闫恒
+ 移除所有的JavaScript
+ 移除标签
+ 转换https超链接为http链接
+ 移除所有cookies中的安全标志


正则表达式配置

![burp-05.png](https://aaron-13.github.io/images/burp-05.png)


其他配置项 
![burp-06.png](https://aaron-13.github.io/images/burp-06.png)

自上而下依次的功能是:
+ 指定使用HTTP/1.0协议与服务器进行通信，，这项设置强制客户端使用HTTP/1.0协议与服务器进行通信。

+ 指定使用HTTP/1.0协议反馈消息给客户端，目前所有浏览器均支持HTTP/1.0协议和HTTP/1.1协议，强制指定HTTP/1.0协议主要用于显示浏览器的某些方面特征，比如，阻止HTTP管道攻击。

+ 设置返回头中的"Connection:close"可用于某些情况下的阻止HTTP管道攻击。

+ 在获取到的请求中设置Connection:close

+ 去除reqeusts中的"Proxy-*"头

+ 去除requests中的"Accetp-Encoding"头部

+ 去除"Sec-WebSocket-Extensions"头部

+ 解压请求消息中的压缩文件

+ 解压响应消息中的压缩文件

+ 禁用http://burp

+ 在浏览器中隐藏Burp Suite的错误信息

+ 不发送关于代理历史的项目或者其他burp工具

+ 如果超出了范围，不发送关于代理历史的项目，或者其他burp工具


**历史记录History**

Burp History的历史记录由HTTP历史和WebSockets历史两个部分组成。

历史消息列表中主要包含请求序列号，请求协议和主机名，请求的方式，URL路径，请求参数，Cookie，是否用户编辑过消息，服务端返回的HTTP状态码等消息。

![burp-07.png](https://aaron-13.github.io/images/burp-07.png)
按照过滤条件的不同，筛选过滤器划分出7个子版块，分别是:
+ 按照请求类型过滤，可以选择仅显示当前作用域的，仅显示有服务端响应和仅显示带有请求参数的消息。当勾选了"仅显示当前作用域"时，此作用域需要在Burp Suite的Scope选项中进行配置。

+ 按照MIME类型过滤，可以控制是否显示服务端安徽的不同的文件类型消息

+ 按照服务端返回的HTTP状态码过滤

+ 按照查找条件过滤，此过滤器是针对服务器端返回的消息内容，与输入的关键字进行匹配

+ 按照文件类型过滤

+ 按照注解过滤，功能是：根据每一个消息拦截的时候的备注或是否高亮作为筛选条件控制哪些消息在历史列表中显示

+ 按照监听端口过滤



## SSL和Proxy的高级使用

HTTPS协议是为了数据传输安全的需要，在HTTP原有的基础上，加入了安全套接字层SSL协议，通过CA证书来验证服务器的身份，并对通信消息进行加密。基于HTTPS协议这些特征，在使用Burp Proxy代理时，需要增加设置，才能拦截HTTPS的消息。

**CA证书的安装**

1.配置好Burp Proxy监听的端口和代理服务器设置。卸载已经安装的Burp Suite的CA证书。

2.以管理员身份，启动IE浏览器，在地址栏输入http://burp，进入证书下载页

3.下载证书到本地

4.将证书导入到对应浏览器的证书管理器中


**CA证书卸载**

1.可以通过浏览器的证书管理器进行卸载

2.win + r --> mmc --> 文件菜单 --> 打开/删除管理单元 --> 添加证书 


**Proxy监听设置**
当测试非浏览器应用时，无法使用浏览器代理的方式去拦截客户端与服务端通信的数据流量，这种情况下，使用Proxy监听设置，而不是默认的设置。

![burp-08.png](https://aaron-13.github.io/images/burp-08.png)

Proxy监听设置主要包含3个模块:
1.端口和IP绑定设置binding，绑定IP地址分仅本地回路，所有接口，指定地址三种模式

2.请求处理Request Handing请求处理主要是用来控制接受到Burp Proxy监听端口的请求后，如果队请求进行处理的。其配置可分为: 端口转发，主机名/域名的转发，强制使用SSL和隐形代理4部分。

3.SSL证书，这些设置控制呈现给SSL客户端的服务器SSL证书，可解决使用拦截代理时出现的一些SSL问题。有下列选项可供设置:
+ 使用自签名证书(Use a self-singed certificate)

+ 生成每个主机的CA签名证书(Generate CA-signed per-host certicates),默认选项。在安装时，Burp创造了一个独特的自签名的证书颁发机构(CA)证书

+ 生成与特定的主机名CA签发的证书(Generate a CA-signed certificate with a specific hostname)

+ 使用自定义证书(Use a custom certificate)


**SSL直连和隐形代理**
SSL直连的设置主要用于指定的目的服务器直接通过SSL连接，而通过这些连接的请求或想响应任何细节将在Burp代理拦截视图或历史日志中显示。通过SSL连接传递可以不是简单地消除在客户机上SSL错误的情况下有用。如果启动自动添加客户端SSL协商失败的选项，当客户端失败的SSL协议检测，并会自动将相关的服务器添加到SSL直通通过列表中。
有时，在拦截富客户端软件时，通常要使用隐形代理。富客户端软件通常是指运行在浏览器之外的客户端软件，这就意味着它本身不具有HTTP代理的属性。
使用隐形代理通常需要做如下设置:

1.配置hosts文件，windows操作系统目录(Windows/System32/drivers/etc/hosts),Linux/Unix目录(/etc/hosts),添加如下行: 127.0.0.1 example.com

2.添加一个新的监听来运行在HTTP默认的80端口，如果使用HTTPS协议，则端口为443。如果是HTTPS协议的通信方式，需要一个指定域名的CA证书。接着，需要把Burp拦截的流量转发给原始请求的服务器。这需要在Project Options -> Connections -> Hostname Resolution进行设置。因为已经将example.com的监听地址在127.0.0.1上，所以在Burp上，将example.com的流量转发到真是服务器那里去。

![burp-09.png](https://aaron-13.github.io/images/burp-09.png)

3.通过这样的配置，就可以欺骗富客户端，将流量发送到Burp坚挺的端口上，再由Burp将流量转发给真实的服务器。




## 使用Burp Target
Burp Target组件主要包含站点地图，目标域，Target工具三部分组成。

**目标域设置Target Scope**
可以通过域名或者主机名去限制拦截内容，可以对作用域为目录的请求进行拦截,其主要使用于以下几种场景中:
+ 限制站点地图和Proxy历史中的显示结果

+ 告诉Burp Proxy拦截哪些请求

+ Burp Spider抓取哪些内容

+ Burp Scanner自动扫描哪些作用域的安全漏洞

+ 在Burp Intruder和Burp Repeater中指定的URL


通过Target Scope，能方便控制Burp的拦截范围，操作对象，减少无效的噪音。在Target Scope的设置中，主要包含两部分功能: 允许规则和去除规则。

![burp-10.png](https://aaron-13.github.io/images/burp-10.png)


**站点地图**

![burp-11.png](https://aaron-13.github.io/images/burp-11.png)

Site Map的左边为访问的URL，按照网站层级和深度，树型展开整个应用系统的结构和关联其他域的url情况；右边显示具体的信息，可以选择某个分支，对指定的路径进行扫描和抓取,或者添加到Target Scope中:

![burp-12.png](https://aaron-13.github.io/images/burp-12.png)


**Target工具使用**
Target工具的使用主要包括以下部分:
+ 手工获取站点地图

+ 站点比较

+ 攻击面分析

手工获取站点地图时，需要遵循以下操作: 1.设置浏览器代理和Burp Proxy代理，并使之能正常工作 2.关闭Burp Proxy的拦截功能 3.手工浏览网页，这时Target会自动记录站点地图信息

站点比较是一个Burp对站点进行动态分析的利器。主要有以下3中场景:1.同一个账号，具有不同的权限，比较两次请求结果的差异 2.两个不同的账号，具有不同的权限，比较两次请求结果的差异 3.不同的账号，具有相同的权限，比较两次请求结果的差异

![burp-13.png](https://aaron-13.github.io/images/burp-13.png)

站点比较是在两个站点地图之间进行的，在配置过程中需要分别指定Site Map1和Site Map2，通常情况下，Site Map1默认为当前会话，点击Next

如果是全站点比较，选择第一项，如果仅仅比较选中的功能，选择第二项，如果全站点比较，且不想加载其他域时，可以勾选只选择当前域。

对于Site Map2的配置有两种方式，第一种是之前已经保存下来的Burp Suite站点记录，第二种是重新发生一次请求作为Site Map2.

如果选择了第二种方式，则进入请求消息设置界面，在这个界面，需要指定通信的并发线程数，失败重试次数，暂停的间隙时间。

设置完Site Map1和Site Map2之后，将进入请求消息匹配设置，在这个界面可以通过URI文件路径，Http请求的方式，请求参数，请求头，请求Body来对匹配条件进行过滤

设置请求匹配条件，接着进入应答比较设置界面。主要有响应头，form表单域，空格，MIME类型


攻的使用击面分析是Burp Suite交互工具(Engagement tools)中的功能，Analyze Target
![burp-14.png](https://aaron-13.github.io/images/burp-14.png)

在弹出的分析界面中，有概况，动态URL，静态URL，参数4个视图

在使用攻击分析功能时，需要注意，此功能主要是针对站点地图中的请求URL进行分析，如果某些URL没有记录，则不会被分析到。存在很多站点使用伪静态，如果请求的URL中不带有参数，则分析时无法区别，只能当做静态URL来分析。

