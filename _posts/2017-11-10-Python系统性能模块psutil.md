# Python系统性能模块--psutil

------

psutil是一个跨平台库，能够轻松实现获取系统运行的进程和系统利用率信息。主要应用于系统监控，分析和限制系统资源及进程的管理。实现了如ps,top,lsof,netstat,ifconfig,who,df,kill,free,nice,ionice,iostat,iotop,uptime,pidof,tty,taskset,pmap等


## 获取系统性能

(1) CPU信息
psutil.cpu_frey() CPU频率
psutil.cpu_percent() CPU使用百分比
psutil.cpu_times() CPU完整信息，如要显示所有逻辑CPU信息，添加percpu=True参数即可
psutil.cpu_times().user 获取指定用户CPU时间比
psutil.cpu_count() CPU的逻辑个数，添加logical=False，获取物理个数

(2) 内存信息
psutil.virtual_memory() 获取内存完整信息
psutil.virtual_memory().total 获取内存总数
psutil.virtual_memory().free 获取空闲内存
psutil.swap_memory() 获取swap分区信息

(3) 磁盘信息
psutil.disk_usage() 获取磁盘利用率信息
psutil.disk_partitions() 获取磁盘完整信息
psutil.disk_usage('/') 获取分区('/')使用情况
psutil.disk_io_counters() 获取硬盘总数的IO个数，读写信息，使用perdisk=True获取单个分区IO个数

(4) 网络信息
psutil.net_io_counters() 获取网络总的IO信息，使用pernic=True输出每个网络接口的IO信息

(5) 其他系统信息
psutil.user() 返回当前登录系统的用户信息
psutil.boot_time() 获取开机时间，以Linux时间戳格式返回
datetime.datetime.fromtimestamp(psutil.boot_time()).strftime("%Y-%m-%d %H:%M:%S") 时间格式转换


## 系统进程管理

(1) 进程信息
psutil.pids() 获取所有PID信息
psutil.Process() 获取单个进程的名称，路径，状态，系统资源利用率等信息
psutil.Process().status() 获取进程状态信息
psutil.Process().cwd() 获取进程工作目录的绝对路径
psutil.Process().memory_percent() 获取进程内存利用率
psutil.Process().io_counters() 获取进程IO信息，包括读写IO数及字节数
psutil.Process().connections() 获取打开进程socket的namedutples列表，包括fs，family，laddr等信息
psutil.Process().num_threads() 获取进程开启的线程数


(2) popen类的使用
获取用户启动的应用程序进程信息，以便跟踪程序进程的运行状态。
psutil.Popen(["/usr/bin/python","-c","print('hello')"], stdout=PIPE)


## IP地址处理模块IPy

### IP地址网段的基本处理
```
from IPy import IP
ip = IP('192.168.1.1')
ip.reverseNames()  	# 反向解析地址格式
ip.iptype() 		# ip类型private
IP('8.8.8.8').int()	# 转换成整型格式
IP('8.8.8.8').strHex()	# 十六进制
IP('8.8.8.8').strBin()	# 二进制
IP('192.168.1.0').make_net('255.255.255.0')	# 192.168.1.0/24
IP('192.168.1.0/24').strNormal(0/1/2/3)	# 根据不同参数值显示不同输出类型的网段 


## DNS处理模块dnspython
可用于查询，传输并动态更新ZONE信息，同时支持TSIG(事务签名)，验证消息和EDNS0(扩展DNS)。

dnspython提供一个DNS解析器类--resolver，使用它的query方法来实现域名的查询功能。
```
query(self, qname, rdtype=1, rdclass=1, tcp=False, source=None, raise_on_no_answer=True, source_port=0)
```

rdtype参数用来指定RR资源的类型。rdclass参数用于指定网络类型IN/CH和HS，IN为默认。tcp参数用于指定是否启用TCP协议。source和source_port参数作为指定查询源地址和端口。raise_on_no_answer参数用于指定当查询无应答时是否触发异常。

```
#!/usr/bin/python3
import dns.resolver

domain = input("input an domain")
a = dns.resolver.query(domain, 'A')		# 指定查询类型为A记录
for i in a.response.answer: 			# 通过response.answer方法获取查询回应信息
	for j in i.items: 					遍历回应信息
		print(j.address)
```

### DNS轮询业务监控

```
import urllib.request
import os 
import dns.resolver

iplist = []
domain = "www.56pingtai.net"

def get_iplist(domain=""):
	try:
		ip = dns.resolver.query(domian, 'A') 	# 通过domain查询其ip地址
	except Exception:
		print("dns resolve failed")
		return

	# 遍历结果，获取其ip值
	for i in ip.response.answer: 
		for j in i.items:
			# iplist.append(j.address)
			if isinstance(j, rdtypes.IN.A.A): # 现在A记录中可能存在CNAME，所以不可直接添加至列表中，需要先判断其类型
				iplist.append(j.address) 	
	return True 	# 如果正确执行，则返回True

def CheckIp(ip):
	url_f = ip + ":80"
	agent = 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)'
	try:
		# 通过urllib这个模块获取其请求页面信息
		url_s = urllib.request.Request(url_f, headers = {'Host': domain, 'User-Agent': agent}, method = "GET") 	# 添加修改headers中的值，自定义请求头信息
		response = urllib.request.urlopen(url_s, timeout=5)
	except Exception:
		print("url check failed")
		return
	finally:
		html = response.read(16).decode('utf-8') 	#将页面的返回结果转换成utf-8编码

		if re.match("<Doctype htm\w+", html, re.I|re.M): 	# 使用正则进行判断页面是否正常
			print("this ip %s is [OK]" % ip)
		else:
			print("this ip %s is [malfunction]" % ip)

if __name__ == "__main__":
	if get_iplist(domain) and len(iplist) > 0:
		for ip in iplist:
			CheckIp(ip)
	else:
		print("dns resolver error")		
```




## 业务服务监控

### 文件内容差异对比方法

difflib模块可用来对比代码，配置文件的差别，与diff命令相似

```
import difflib

test01 = """test01:
This module provides classes and functions foor comparing sequences
======
"""

test02 = """test02:
This module provides classes and function for comparing sequences
++++++
"""

test01_lines = test01.splitlines()
test02_lines = test02.splitlines()

d = difflib.Differ()
diff = d.compare(test01_lines, test02_lines)
print("\n".join(list(diff)))

```

Differ()类对两个字符串进行比较，difflib的SequenceMatcher()类支持任意类型序列的比较，HtmlDiff()支持将比较结果输出为HTML格式

![differ-01.png](https://aaron-13.github.io/images/difflib-01.png)

|符号|含义|
|:---:| :---:|
|'-'|包含在第一个序列中，但不包含在第二个序列中|
|'+'|包含在第二个序列中，但不包含在第一个序列中|
|''|两个序列行一致|
|'?'|标志两个序列行存在增量差异|
|'^'|标志两个序列行存在的 差异字符|


### 生成HTML格式的文档

```
d = difflib.HtmlDiff()
print d.make_file(test01_lines, test02_lines)
```

将输出内容重定向到文件中，在浏览器中打开即可


### 对比nginx配置文件差异

```
import difflib
import sys

try:
	file01 = sys.argv[1]
	file02 = sys.argv[2]
except Exception:
	print("please input two file to compare,like diff.py file01 file02")
	sys.exit()


def readfile(filename):
	try:
		file = open(filename, 'rb')
		content = file.read().splitlines()
		file.close()
		return content
	except IOError as error:
		print("Read file Error: %s" % error)
		sys.exit()

if file01 == "" or file02 == "":
	print("useage: diff.py filename1 filename2")
	sys.exit()

content01 = readfile(file01)
content02 = readfile(file02)

d = difflib.HtmlDiff()
print d.make_file(content01, content02)
```


## 文件与目录差异对比方法
模块filecmp可以实现文件，目录，遍历子目录的差异对比功能。

filecmp提供了三种操作方法，分别是cmp(单文件对比)，cmpfiles(多文件对比)，dircmp(目录对比)

+ 单文件对比，采用filecmp.cmp(f1,f2[,shallow])方法。相同返回True，不同返回False，shallow默认为True。意思是只根据os.stat()方法返回的文件基本信息进行对比，忽略文件内容的对比

+ 对文件对比，采用filecmp.cmpfiles(dir1,dir2,common[,shallow])方法，对于dir1与dir2目录给定的文件清单。该方法返回文件名的三个列表，分别为匹配，不匹配，错误。

+ 目录对比，通过dircmp(a,b[,ignore[,hide]])类创建一个目录比较对象。a,b是比较的目录名，ignore代表文件名忽略的列表，并默认为['RCS','CVS','tags'];hide代表隐藏的列表，默认为[os.curdir, os.pardir]

dircmp提供了三种输出报告的方法
	+ report() 	比较当前指定目录中的内容
	+ report_partial_closure() 	比较当前指定目录及第一级子目录中的内容
	+ report_full_closure() 	递归比较所有指定目录的内容



## 发送电子邮件
smtplib模块实现邮件的发送功能，模拟一个smtp客户端，通过与smtp服务器交互来实现邮件发送的功能。

SMTP类定义: smtplib.SMTP([host[,port[,local_hostname[,timeout]]]]),作为SMTP的构造函数。host参数为远程smtp主机地址，port为端口，默认25，local_hostname的作用是在本地主机的FQDN发送HELO/EHLO(标识用户身份)指令，timeout超时时间。

+ SMTP.connect([host[,port]])

+ SMTP.login(user,password)

+ SMTP.sendmail(from_addr, to_addrs, msg[,mail_options, rcpt_options])

+ SMTP.starttls(keyfile[,cerfile]) 

+ SMTP.quit()

使用gmail发送邮件
```
import smtplib

host = "smtp.gmail.com"
port = 25
f_addr = "test@gmail.com"
t_addr = "to@gmail.com"
text = "this is text info"
body = "\r\n".join(( 		# 自定义body，以'\r\n'即换行符进行分割
	"From: %s" % f_addr,
	"To: %s" % t_addr,
	"Subject: %s" % subject,
	"",
	text
))

# 创建SMTP实例
server = smtplib.SMTP()
server.connect(host,port)
server.login(f_addr,"password")
server.sendmail(f_addr, [t_addr], body) 	# 收件人为列表，可有多个
server.quit()

```

### MIME邮件格式
MIME(Multipurpose Internet Mail Extensions,多用途互联网邮件扩展)

+ email.mime.multipart.MIMEMultipart([_subtype[,boundary[,_subparts[,_params]]]]),作用是生成包含多个部分的邮件体的MIME对象，参数_subtype指定要添加到"Content-type:multipart/subtype"报头的可选的三种子类型，分别是mixed，related，alternative，默认值mixed。定义mixed实现构建一个带附件的邮件体，定义related实现构建内嵌资源的邮件体，定义alternative则实现构建纯文本与超文本共存的邮件体。

+ email.mime.audio.MIMEAudio(_audiodata[,_subtype[,_encoder[,**_params]]]),创建包含音频数据的邮件体，_audiodata包含原始二进制音频数据的字节字符串

+ email.mime.image.MIMEImage(_imagedata[,_subtype[,_encoder[,**_params]]])，创建包含图片数据的邮件

+ email.mime.text.MIMEText(_text[,_subtype[,_charset]])，创建包含文件数据的邮件体

```
#coding: utf-8
import smtplib
from email.mime.text import MIMEText
...
msg = MIMEText("""<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>

</body>
</html>
""")
```

### 通过MIMEText和MIMEImage类的组合实现图文格式的服务器性能报表邮件

```
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage

host = "smtp.gmail.com"
port = 25
subject = "report"
t_addr = "to@gmail.com"
f_addr = "from@gmail.com"

def add(src,imgid):
	fp = open(src,'rb')
	msgImage = MIMEImage(fp.read()) 	# 创建MIMEImage对象，读取图片内容作为参数
	fp.close()
	msgImage.add_header("Content-ID", imgid) 	# 指定图片文件的Content-ID 
	return msgImage

msg = MIMEMultipart('related') 	# 创建MIMEMultipart对象，采用related定义内嵌资源的邮件体

msgtext= MIMEText("""
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
<src <img src="cid:mem"> 	# <img>标签的src属性是通过Content-ID来引用的
<src <img src="cid:swap">
</body>
</html>
""","html","utf-8")

msg.attach(msgtext) 	# MIMEMultipart对象附加MIMEText的内容
msg.attach(add("img/os_mem.png", "mem")) 	# 使用MIMEMultipart对象附加MIMEImage的内容
msg.attach(add("img/os_swap.png", "swap"))
msg["Subject"] = subject
msg["From"] = f_addr
msg["To"] = t_addr

...
```

### 带附件格式的邮件

```
...

# 创建一个MIMEText对象，附加report.xlsx文档
attach = MIMEText(open("doc/report.xlsx",'r').read(), "base64", "utf-8")

attach["Content-Type"] = "application/octet-stream" 	#指定文件格式类型
#指定Content-Disposition值为attachment，则出现下载保存对话框，保存的文件名使用filename指定
attach["Content-Disposition"] = "attachment; filename=\"report.xlsx\""
...
```


## 探测Web服务质量方法
pycurl模块，支持FTP，HTTP，HTTPS，TELNET等。

+ close() 	对应libcurl包中的curl_easy_cleanup方法
+ perform() 对应libcurl包中的curl_easy_perform方法，无参数，实现Curl对象请求的提交
+ setopt(option, value) 	对应libcurl包中的curl_easy_setopt方法，option通过libcurl的常量来指定的参数value的值会依赖option
```
c = pycurl.Curl() 创建一个curl对象
c.setopt(pycurl.CONNECTTIMEOUT, 5) 连接的等待时间，0为不等待
c.setopt(pycurl.TIMEOUT, 5) 请求超时时间
c.setopt(pycurl.NOPROGRESS, 0) 是否屏蔽进度条，非0屏蔽
c.setopt(pycurl.MAXREDIRS,1) 指定HTTP重定向的最大数
c.setopt(pycurl.FORBID_REUSE, 1) 完成交互后强制断开连接，不重用
c.setopt(pycurl.FRESH_CONNECT, 1) 强制获取新的连接，即替代缓存中的连接
c.setopt(pycurl.DNS_CACHE_TIMEOUT, 60) 保存DNS信息的时间，默认120s
c.setopt(pycurl.URL, "https://www.google.com") 指定请求的URL
c.setopt(pycurl.USERAGENT, "chrome/1.10") 指定请求http头的user-agent
c.setopt(pycurl.HEADERFUNCTION, getheader) 将返回http header定向到回调函数getheader
c.setopt(pycurl.WRITEFUNCTION, getbody) 将返回的内容定向到回调函数getbody
c.setopt(pycurl.WRITEHEADER, fileobj) 将返回的HTTP header定向到fileobj文件对象
c.setopt(pycurl.WRITEDATA, fileobj) 将返回的html内容定向到fileobj文件对象
```

+ getinfo(option) 对应libcurl包中的curl_easy_getinfo方法
```
c.getinfo(pycurl.HTTP_CODE) 返回HTTP状态码
TOTAL_TIME  传输结束所消耗的总时间
NAMELOOKIP_TIME  DNS解析消耗的时间
CONNECT_TIME  建立连接消耗的时间
PRETRANSFER_TIME 建立连接到准备传输消耗的时间
STARTTRANSFER_TIME  建立连接到传输开始消耗的时间
REDIRECT_TIME  重定向消耗的时间
SIZE_UPLOAD  上传数据包大小
SIZE_DOWNLOAD  下载数据包大小
SPEED_DOWNLOAD  下载平均速度
SPEED_UPLOAD    上传平均速度
HEADER_SIZE     HTTP头部大小
```


## 数据报表Excel模块
XlsxWriter模块可以操作多个工作表的文字，数字，公式，图表等

```
import xlsxwriter

workbook = xlsxwriter.Workbook('demo.xlsx')  # 创建一个Excel文件
worksheet = workbook.add_worksheet() 		# 创建一个工作表对象

worksheet.set_column('A:A', 20) 	# 设定第一列(A)宽度为20像素
bold = workbook.add_format({'bold': True})  # 定义一个加粗的格式对象

worksheet.write('A1': 'hello') 	# A1单元格写入hello
worksheet.write('A2': "world", bold) 	# A2单元格写入加粗的world
worksheet.write('B2', 'test', bold) 	# B2写入加粗的test

worksheet.write(2, 0, 32) 	# 用行列表示法写入数字32和35.5
worksheet.write(3, 0, 35.5) # 行列下标表示法的单元格下标以0开始，'3, 0'相当于A3
worksheet.write(4, 0, '=SUM(A3:A4)') # 求A3:A4的和，并将结果写入'4, 0'即A5

worksheet.insert_image('B5', 'img/logo.png') # 在B5单元格插入图片
worksheet.insert_image(6, 2, 'img/logo1.png')
worksheet.close() # 关闭Excel文件
```

### 模块常用方法
1. Workbook类
Workbook(filename[,options]),该类实现创建一个XlsxWriter的Workbook对象。参数filename(String类型)为创建的Excel文件存储路径；参数options(Dict类型)为可选的Workbook参数，一般作为初始化工作表内容格式。

	+ add_worksheet([sheetname]), 添加一个新的工作表，参数sheetname(String类型)为可选的工作表名称，默认Sheet1

	+ add_format([properties])，在工作表中创建一个新的格式对象来格式化单元格。参数properities(dict类型)为指定一个格式属性的字典。

	+ add_chart(options)，在工作表内创建一个图标图像，内部是通过insert_chart()方法来实现。参数options(dict类型)为图表指定一个字典属性，如设置一个线条类型的图表对象 chart = workbook.add_chart({'type': 'line'})

	+ close()，关闭工作表文件

2. Worksheet类
Worksheet类代表了一个Excel工作表，是最核心的一个类。Worksheet对象不能直接实例化，取而代之的是通过Workbook对象调用add_worksheet()方法来创建。
	
	+ write(row, col, *args)

	+ set_row(row, height, cell_format, options), 设置行单元格的属性。cell_format(format类型)指定格式对象，参数options(dict类型)设置行hidden，level(组合分级)，collapsed(折叠)

	+ set_column(first_col, last_col width, cell_format, options), 设置一列或多列单元格属性。

	+ insert_image(row, col, image[,options]), 插入图片到指定单元格，支持PNG，JPEG，BMP等格式。worksheet.insert_image('B5', 'img/2.png',{'url': 'https://google.com'})


3. Chart类
Chart类实现在XlsxWriter模块中图表组件的基类，支持的图表类型包括面积，条形图，柱形图，折线图，饼图，散点图，股票，雷达等。
```
	area: 面积样式的图表
	bar: 条形样式的图表
	column: 柱形样式的图表
	line: 线条样式的图表
	pie: 饼图样式的图表
	scatter: 散点样式的图表
	stock: 股票样式的图表
	radar: 雷达样式的图表
```

	+ chart.add_series(options), 添加一个数据系列到图表，参数options(dict类型)设置图表系列选项的字典，其最常用的三个选项为categories，values，lines。categories作为设置图表类别标签范围。

	+ set_x_axis(options)，设置图表X轴选项

	+ set_size(options), 设置图表大小

	+ set_title(options)，设置图表标题

	+ set_style(style_id)， 设置图表样式

	+ set_table(options)， 设置X轴为数据表格形式


```
#!/usr/bin/python3
# -*- coding: utf-8 -*-
#
import xlsxwriter

workbook = xlsxwriter.Workbook('demo2.xlsx')  # 创建一个excel文件
worksheet = workbook.add_worksheet() # 创建一个excel工作表对象
chart = workbook.add_chart({'type': 'column'}) # 创建一个图表对象

# 定义数据表头列表
title = [u'业务名称', u'星期一', u'星期二', u'星期三', u'星期四', u'星期五', u'星期六', u'星期日', u'平均流量']

buname = [u'业务官网', u'新闻中心', u'购物频道', u'体育频道', u'亲子频道']

# 定义5频道一周流量数据列表
data = [
	[150,152,158,149,155,145,148],
	[89,88,95,93,98,100,99],
	[201,200,198,175,170,198,195],
	[75,77,78,78,74,70,79],
	[88,85,87,90,93,88,84], 
]

format = workbook.add_format() # 定义format格式对象
format.set_border(1) # 边框加粗

format_title = workbook.add_format()
format_title.set_border(1)
format_title.set_bg_color('#ccccc')

format_title.set_align('center') # 定义对象单元格居中对齐的格式
format_title.set_bold()

format_ave = workbook.add_format()
format_ave.set_border(1)
format_ave.set_num_format('0.00') # 定义对象单元格数字类别显示格式

# 分别以行或列写入方式将标题，业务名称，流量数据写入起初单元格，同时引用不同格式对象
worksheet.write_row('A1', title,format_title)
worksheet.write_column('A2', buname,format)
worksheet.write_row('B2',data[0],format)
worksheet.write_row('B3',data[1],format)
worksheet.write_row('B4',data[2],format)
worksheet.write_row('B5',data[3],format)
worksheet.write_row('B6',data[4],format)


# 定义图表数据系列函数
def chart_series(cur_row):
	# 计算(AVERAGE函数)频道周平均流量
	worksheet.write_formula('I'+cur_row, '=AVERAGE(B'+cur_row+':H'+cur_row+')',format)
	chart.add_series({
		# 将"星期一至星期日"作为图表数据标签
		'categories': "=Sheet1!$B$1:$H$1",
		# 频道一周所有数据作为数据区域
		'values': '=Sheet1!$B$'+cur_row+':$H$'+cur_row,
		'line': {'color': 'black'},
		# 引用业务名称为图例项
		'name': '=Sheet1!$A$'+cur_row,
})

# 数据域以第2-6行进行图表数据系列函数调用
for row in range(2,7):
	chart_series(str(row))


chart.set_size({'width': 577, 'height': 287})
chart.set_title({'name': u'流量周报'})
chart.set_y_axis({'name': 'Mb/s'})

# 在A8出插入图像
worksheet.insert_chart('A8', chart)
workbook.close()
```


## 环状数据库存储模块
rrdtool(round robin database)工具为环状数据库的存储格式，是一种处理定量数据以及当前元素指针的技术。

最新包2012年更新


## 生成动态路由轨迹图
scapy是一个强大的交互式数据包处理程序，能够对数据包进行伪造或解包，发送数据包，包嗅探，应答和反馈匹配等功能。
scapy提供众多网络数据包操作方法，包括发包send(),SYN\ACK扫描，嗅探sniff(),抓包wrpcap(),TCP路由追踪traceroute()等。

traceroute(target, dport=80, minttl=1, maxttl=30, sport=<RandShort>, 14=None, filter=None, timeout=2, verbose=None, **kargs)

	+ target: 跟踪的目标，可以是域名或ip，类型为列表，支持指定多个目标
	+ dport: 目标端口，类型为列表
	+ minttl: 路由跟踪的最小跳数
	+ maxttl: 路由跟踪的最大跳数

实现TCP探测目标服务路由轨迹
![scapy-01.png](https://aaron-13.github.io/images/scapy-01.png)

```
import os, sys, time, subprocess
import warnings, logging
warnings.filterwarnings("ignore", category=DeprecationWarning) 	# 屏蔽scapy无用告警信息
logging.getLogger("scapy.runtime").setLevel(logging.ERROR) 	# 屏蔽模块IPv6多余告警
from scapy.all import traceroute

domain = input("Please input one or more IP/domain")
target = domain.split(' ')
dport = [80] 	# 扫描的端口列表

if len(target) >= 1 and target[0] != '';
	res, unans = traceroute(target, dport=dprot, retry=-2) 	# 自动路由跟踪
	res.graph(target="test.svg")
	time.sleep(1)
	subprocess.Popen("/usr/bin/convert test.svg test.png", shell=True) 	# svg转png格式
else:
	print("IP/domain number of errors,exit")
```	
如执行失败，根据报错信息排查，dot(graphviz), convert(ImageMagick), lncurses(ncurses), readline



# 系统安全
Clam AntiVirus(ClamAV)是一款开放源代码的防毒软件。pyClamad模块可以让Python直接使用ClamAV病毒扫描守护进程clamd，来实现一个高效的病毒检测功能。

pyClamd提供了两个关键类: 一个为ClamdNetworkSocket()类，实现使用网络套接字操作clamd，另一个为ClamdUnixSocket()类，实现使用Unix套接字操作clamd。
	+ __init__(self, host='127.0.0.1', port=3310, timeout=None)方法，是ClamdNetworkSocket类的初始化方法。host与/etc/clamd.conf配置文件中的TCPSocket参数一致

	+ contscan(self, file)，实现扫描指定的文件或目录，发生错误不会终止

	+ multiscan_file(self, file), 实现多线程扫描指定的文件或目录

	+ scan_file(self, file), 实现扫描指定文件或目录，发生错误会终止

	+ shutdown(self) 强制关闭clamd进程并退出

	+ stats(self) 获取Clamscan的当前状态

	+ reload(self) 强制重载clamd病毒特征库

	+ EICAR(self) 返回EICAR测试字符串


```
#!/usr/bin/python3
# -*- coding:utf-8 -*-
#
import pyclamd
from threading import Thread
import time

class Scan(Thread):
	
	def __init__(self,IP,scan_type,file):
		"""构造方法，参数初始化"""
		Thread.__init__(self)
		self.IP = IP
		self.scan_type = scan_type
		self.file = file
		self.scanresult = ""

	def run(self):
		"""多进程run方法"""
		try:
			cd = pyclamd.ClamdNetworkSocket(self.IP, 3310) 	# 创建网络套接字连接对象
			if cd.ping(): 	# 测试连通性
				self.connstr = self.IP+" connection [OK]"
				cd.reload() 	# 重载clamd病毒特征库
				if self.scan_type == "contscan_file": 	# 选择不同的扫描方式
					self.scanresult = "{0}\n".format{cd.contscan_file(self.file)}
				elif self.scan_type == "multiscan_file":
					self.scanresult = "{0}\n".format(cd.multiscan_file(self.file))
				else self.scan_type == 	"scan_file":
					self.scanresult = "{0}\n".format(cd.scan_file(self.file))		
				time.sleep(1)
			else:
				self.connstr = self.IP+ "ping error,exit"
		except Exception as error:
			self.connstr = self.IP + " " + str(error)

IPs = ['192.168.71.215', '192.168.71.216'] 	# 扫描主机列表
scantype = "multiscan_file" 	# 指定扫描模式
scanfile = "/etc/sysconfig" 	# 指定扫描路径
i = 1

threadnum = 3	# 指定启动线程数
scanlist = []	# 存储扫描Scan类线程对象列表

for ip in IPs:
	curip = Scan(ip, scantype, scanfile) 	# 创建扫描Scan类对象，参数(IP,扫描模式，扫描路径)
	scanlist.append(curip) 	# 追加对象到列表

	if i%threadnum == 0 or i==len(IPs): 	# 当达到指定的线程数或IP列表数后启动，退出线程
		for task in scanlist:
			task.start() 	# 启动线程

		for task in scanlist:
			task.join() 	# 等待所有子线程退出，并输出扫描结果
			print(task.connstr) 	# 打印服务器连接信息
			print(task.scanresult) 	# 打印扫描结果
		scanlist = []
	i += 1
```

通过EICAR()方法生成一个带有病毒特征的文件/tmp/EICAR
```
file = open('/tmp/EICAR', 'w').write(cd.EICAR())
```


### 端口扫描Nmap

nmap模块中常用的两个类: 一个是PortScanner()类，实现nmap工具的端口扫描功能封装，另一个是PortScannerHostDict()类，实现存储与访问主机的扫描结果。

PostScanner()类:
	+ scan(self, hosts='127.0.0.1', ports=None, arguments='-sV')方法，实现指定主机，端口，nmap命令行参数的扫描。

	+ command_line(self)方法，返回的扫描方法映射到具体的命令行

	+ scaninfo(self)方法，返回nmap扫描结果，格式为字典类型

	+ all_hosts(self)方法，返回nmap扫描的主机清单


PostScannerHostDict()类常用方法：
	+ hostname(self)方法，返回扫描对象的主机名

	+ state(self)方法，返回扫描对象的状态，(up,down,unknown,skipped)

	+ all_protocols(self)方法，返回扫描的协议

	+ all_tcp(self)方法，返回TCP协议扫描的端口

	+ tcp(self, port)方法，返回扫描TCP协议port信息


```
#!usr/bin/python3
# -*- coding:utf-8 -*-
#

import sys
import nmap

scan_row = []
data = input("please input ip and port:  ")
scan_row.split(" ")

hosts = scan_row[0]
ports = scan_row[1]
if len(scan_row) != 2:
	print("input error: ")
	sys.exit(0)

try:
	nm = nmap.PortScanner() 	# 创建端口扫描对象
except Exception:
	print("nmap not found: ", sys.exc_info()[0]
	sys.exit(0)
except Exception:
	print("nmap error: ", sys.exc_info()[0])
	sys.exit(0)

try:
	nm.scan(hosts=hosts, arguments=' -v -sS -p'+ports) 	# 进行扫描
except Exception as e:
	print("nmap scan error: %s" % str(e) )

for host in  nm.all_hosts():
	print("=========================")
	print("Host: %s %s" % (host, nm[host].hostname()))
	print('State: %s' % nm[host].state())

	for proto in nm[host].all_protocols():
		print("------------------------")
		print("protocol: %s" % proto)

		lport = nm[host][proto].keys()
		lport.sort()
	
		for port in lport:
			print("port: %s \tstate: %s" % (port, nm[host][proto][port]['state']))

```



# 系统批量运维管理器pexpect

通过pexpect可以实现对ssh，ftp，passwd，telnet等命令行进行自动交互。

```
import pexpect
child = pexpect.spawn('scp foo user@example.com:.') 	# spawn启动scp程序
child.expect('Password:') 	# expect方法等待子进程产生的输出，判断是否匹配定义的字符串
child.sendline(mypassword) 	# 匹配后则发送密码串进行回应


## pexpect的核心组件

### spawn类
spawn是pexpect的主要类接口，功能是启动和控制子应用程序
```
class pexpect.spawn(command, args=[], timeout=30, maxread=2000, searchwindowsize=None, logfile=None, cwd=None, env=None, ignore_sighup=True)

当子程序需要参数时，还可以使用Python列表来代替参数项:
```
child = pexpect.spawn('/usr/bin/ssh', ['user@example.com'])
child = pexpect.spawn('ls', ['-lh', '/tmp'])
```

参数maxread是pexpect从终端控制台一次读取的最大字节数，searchwindowsize参数为匹配缓冲区字符串的位置，默认是从开始位置匹配。

pexpect不会解析shell命令当中的字符串，包括重定向">",管道"|"，,通配符"*"，可使用bash命令带参数的形式调用
```
child = pexpect.spawn('/bin/bash -c "ls -l | grep LOG > logs.txt"')
chiid = pexpect.spawn('/bin/bash', ['-c','ls -l | grep LOG > logs.txt'])
```

pexpect提供了两种途径，一种为写到日志文件，另一种为输出到标准输出。

写到日志文件:
```
child = pexpect.spawn('ls -l /')
fout = file('log.txt', 'w')
child.logfile = fout
```

输出到终端
```
child = pexpect.spawn('ls -l /')
child.logfile = sys.stdout

```

远程登录ssh，并记录于日志
```
import pexpect
import sys

child = pexpect.spawn("ssh www@192.168.71.216")
file = open('/tmp/ssh.log', 'wb') 	# 这里文件要以二进制格式写入
child.logfile = file	# logfile是二进制格式存储文件的

child.expect('password')
child.sendline("mypass")
child.expect('\$')
child.sendline('lscpu')
child.expect('\$')
```

(1) expect方法
expect定义了一个子程序输出的匹配规则
expect(pattern, timeout=-1, searchwindowsize=-1)
参数pattern表示字符串，expect.EOF(指向缓冲区尾部，无匹配项)，expect.TIMEOUT(匹配等待超时)，正则表达式或者前面四种类型组成的列表，当pattern为一个列表时，返回最先出现的元素，或者列表最左边的元素(最小索引ID)

expect方法有两个成员: before和after。before成员保存了最近匹配成功之前的内容，after成员保存了最近匹配成功之后的内容。

(2) read相关方法
send(self, s) 	# 发送命令，不回车
sendline(self, s='') 	# 发送命令，回车
sendcontrol(self, char) 	# 发送控制字符 child.sendcontrol('c') 等价于"ctrl+c"
sendeof() 	# 发送eof


### run函数
run是使用pexpect进行封装的调用外部命令的函数，类似于os.system或os.popen方法，不同的是，使用run()可以同时获得命令的输出结果及命令的退出状态。`
pexpect.run(command, timeout=-1, withexitstatus=False, events=None, extra_args=None, logfile=None, cwd=None, env=None)` events是一个字典，定义了expect及sendline方法的对应关系

```
from pexpect import *
run('scp foo user@example.com:.', events={'(?!password)': 'mypassword'})
# 等同于
# child.expect('(?!)password')
# child.sendline('mypassword')
```


### pxssh类
pxssh是pexpect的派生类，针对在ssh会话操作上再一层封装，提供与基类更加直接的操作方法
`class pexpect.pxssh.pxssh(timeout=30 maxread=2000, searchwindowsize=None, logfile=None, cwd=None, env=None)`
	+ login() 建立ssh连接
	+ logout() 断开连接
	+ prompt() 等待系统提示符，用于等待命令执行结束

```
#!/usr/bin/python3
#
from pexpect import pxssh
import getpass


try:
	s = pxssh.pxssh()
	hostname = input("input hostname: ")
	user = input("input user: ")
	password = getpass.getpass("input password: ")
	s.login(hostname, user, password)
	s.sendline('uptime')
	s.prompt()
	print(s.before)
	
	s.sendline('ls -l')
	s.prompt()
	print(s.before)
	
	s.sendline("df -h ")
	s.prompt()
	print(s.before)

except pxssh.ExceptionPxssh as e:
	print("pxssh failed")
	print(str(e))
```


# 系统批量运维管理器paramiko详解
paramiko是基于Python实现的SSH2远程安全连接，支持认证及密钥方式。实现远程命令执行 ，文件传输，中间SSH代理等功能，相对于Pexpect，封装的层次更高。

```
import paramiko

hostname = '192.168.1.1'
username = 'root'
password = 'mypass'
paramiko.util.log_to_file('syslogin.log') 	# 发送paramiko日志到syslogin.log文件

ssh = paramiko.SSHClient() 	#创建一个ssh客户端client对象
ssh.load_system_host_keys() 	#获取客户端host_keys，默认~/.ssh/known_hosts，非默认路径需指定

ssh.connect(hostname=hostname,username=username,password=password) 	# 创建ssh连接
stdin,stdout,stderr = ssh.exec_command('free -m') 	# 调用远程执行命令方法exec_command()
print(stdout.read()) 	#打印命令执行结果，得到Python列表形式，可使用stdout.readlines()
ssh.close() 	# 关闭ssh连接
```

### paramiko的核心组件
+ SSHClient类
	SSHClient类是SSH服务会话的高级表示，该类封装了传输(transport),通道(channel)及SFTPClient的校验，建立的方法，通常用于执行远程命令
```
client = SSHClient()
client.load_system_host_keys()
client.connect('ssh.example.com')
stdin,stdout,stderr = client.exec_command('ls -;')

	+ connect方法
	connect(self, hostname, port=22, username=None, password=None, pkey=None, key_filename=None, timeout=None, allow_agent=True, look_for_keys=True, compress=False)
	pkey(PKey类型)，秘钥方式用于身份验证
	key_filename(str or list(str)类型)，一个文件名或文件名的列表，用于私钥的身份验证
	allow_agent(bool类型)，设置为False时用于禁用连接到SSH代理

	+ exec_command方法
	远程命令执行方法，该命令的输入和输出流为标准输入(stdin),输出(stdout),错误输出(stderr)的Python文件对象
	exec_command(self,command,bufsize=-1)
	bufsize(int类型)，文件缓冲区大小，默认为-1(不限制)

	+ load_system_host_keys方法
	加载本地公钥校验文件，默认为~/.ssh/known_hosts,非默认路径需手工指定
	load_system_host_keys(self, filename=None)

	+ set_missing_host_key_policy方法
	设置连接的远程主机没有本地秘钥或HostKeys对象时的策略，目前支持三种，分别为AutoAddPolicy,RejectPolicy(默认),WarningPolicy,仅限于SSHClient
		+ AutoAddPolicy，自动添加主机名及主机密钥到本地HostKeys对象，并将其保存，不依赖load_system_host_keys()的配置

		+ RejectPolicy，自动拒绝未知的主机名和密钥，依赖load_system_host_keys()配置

		+ WarningPolicy，用于记录一个位置的主机密钥的Python警告


+ SFTPClient类
1. from_transport
创建一个已连通的SFTP客户端通道
from_transport(cls, t)
t(Transport)，一个已通过验证的传输对象

```
t = paramiko.Transport("192.168.71.216",22)
t.connect(username="root", password="mypass")
sftp = paramiko.SFTPClient.from_transport(t)
```

2. put
上传本地文件到远程SFTP服务器
put(self, localpath, remotepath, callback=None, confirm=True)

callback(functioon(int, int)),获取已接收的字节数及总传输字节数，以便回调函数调用，默认为None

3. get
从远程SFTP服务器上下载文件到本地
get(self, remotepath, localpath, callback=None)

4 其他
	+ Mkdir 创建目录
	+ remove 删除SFTP服务器指定目录
	+ rename 重命名
	+ stat 获取远程指定文件信息
	+ listdir 获取远程指定目录列表

私钥文件不存放在指定目录
```
import paramiko
import os

hostname = "192.168.71.216" 
username = "root"

paramiko.util.log_to_file('sys.log')

ssh = paramiko.SSHClient()
ssh.load_system_host_keys()
privatekey = os.path.expanduser("/home/key/id_rsa")
key = paramiko.RSAkey.from_private_key_file(privatekey)

ssh.connect(hostname=hostname, username=username, pkey=key)
stdin,stdout,stderr = ssh.exec_command("df -h")
print(stdout.read())
ssh.close()
```


# 系统批量运维管理器Fabic
Fabric是基于Python实现的SSH命令行工具，简化了SSH的应用程序部署及系统管理任务，提供了系统基础的操作组件，可实现本地或远程shell命令，包括命令执行，文件上传，下载及完整执行日志输出等功能。

创建fabfile.py文件
```
from fabric.api import run

def host_type():
	run("uname -s")
```

使用fab -H localhost,ip1,ip2... host_type 即可查看指定ip的hostname。
fab命令引用默认文件名为fabfile.py，如果使用非默认文件名称，需通过"-f"来指定。如果管理机与；目标主机未配置密钥认证信任，将会提示输入目标主机对应账号登录密码。


## fab常用参数
fab [options] <command>[:arg1, arg2=val2, host=foo, hosts='h1:h2',...]...
+ -i, 显示定义好的任务函数名
+ -f, 指定fab入口文件，默认入口文件名为fabfile.py
+ -g, 指定网关设备，比如堡垒机
+ -H, 指定目标主机，多台主机用","分隔
+ -P, 以异步并行方式运行多主机任务，默认为串行运行
+ -R, 指定role，以角色名区分不同业务组设备
+ -t, 设置设备超时时间
+ -T, 设定远程主机命令执行超时时间
+ -w, 当命令执行失败，发出告警，而非默认中止任务

命令行形式：
`fab -p mypass -H 192.168.71.216 -- 'uname -s'`


## fabfile的编写
fabfile的主体由多个自定义的任务函数组成，不同任务函数实现不同的操作逻辑。

### 全局属性设定
+ env.hosts 定义目标主机，可以用IP或主机名表示
+ env.exclude_hosts 排除指定主机
+ env.user 定义用户名
+ env.port 定义目标主机端口
+ env.password 定义密码
+ env.passwords 与password功能一样，区别在于不同主机不同密码的应用场景。如:
env.passwords = {
	'root@192.168.1.1:22': 'mypass1',
	'root@192.168.1.2:22': 'mypass2',
	'root@192.168.1.3:22': 'mypass3'
}
+ env.gateway 定义网关IP
+ env.deploy_release_dir 自定义全局变量
+ env.roledefs 定义角色分组
env.roledefs = {
	'webserver': ['192.168.1.1', '192.168.1.2'],
	'dbserver': ['192.168.1.3', '192.168.1.4']
}
引用时使用Python修饰符的形式进行，角色修饰符下面的任务函数为其作用域
```
@roles('webserver')
def webtask():
	run('/etc/init.d/nginx start')

@roles('dbserver')
def dbtask():
	run('/etc/init.d/mysql start')

@roles('webserver', 'dbserver')
def task():
	run('uptime')

def deploy():
	execute(webtask)
	execute(dbtask)
	execute(task)
```

### 常用api
fabric.api命令集
+ local 执行本地命令
+ lcd 切换本地目录
+ cd 切换远程目录
+ run 执行远程命令
+ sudo sudo方式执行远程命令
+ put 上传本地文件到远程主机
+ get 从远程主机下载文件
+ prompt 获取用户输入信息
+ confirm 获得提示信息确认
+ reboot 重启远程主机
+ @task 函数修饰符，表示的函数对fab可调用,非标记对fab不可见，纯业务逻辑
+ @runs_once 函数修饰符，标识的函数只会执行一次，不受多台主机影响


文件打包，上传和校验
```
from fabric.api import *
from fabric.context_managers import *
from fabric.contrib.console import confirm
from fabric.color import color
import time

env.user = "root"
env.hosts = ['192.168.1.1', '192.168.1.2']
env.password = 'mypass'

env.project_dev_source = '/data/dev/source'
env.project_tar_source = '/data/dev/release'
env.project_pack_name = 'release'

env.deploy_project_root = '/data/www/project/' 
env.deploy_release_dir = 'release'
env.deploy_current_dir = 'current'
env.deploy_version = time.strftime("%Y%m%d") + "v2"

@runs_once
def input_version():
	return prompt("please input project rollback versionID: ", default="")

@task
@runs_once
def  tar_source():
	print(yellow("Creating source package..."))
	with lcd(env.project_dev_source):
		local("tar czf %s.tar.gz ." % (env.project_dev_source + env.project_pack_name))
	print(green("creating source package success"))

@task
def put_package():
	print(yellow("put packing ..."))
	with settings(warn_only=True):
		with cd(env.deploy_project_root+env.deploy_release_dir):
			run("mkdir %s" %(env.deploy_version))
	env.deploy_full_path = env.deploy_project_rot + env.deploy_release_dir + "/" + env.deploy_version
		with settings(warn_only=True):
			result = put(env.project_tar_source + env.project_pack_name + ".tar.gz", env.deploy_full_path)
		if result.failed and no("put file failed, Continue [Y/N]?"):
			abort("Aborting file put task")

		with cd(env.deploy_full_path):
			run("tar -zxvf %s.tar.gz" % env.project_pack_name)
			run("rm -rf %s.tar.gz" % env.project_pack_name)

		print(green("put && untar package success"))


@task
def make_symlink():
	print(yellow("update current symlink"))
	env.deploy_full_path = env.deploy_project_root + env.deploy_release_dir + "/" + env.deploy_version
	with settings(warn_only=True):
		run("rm -rf %s" % (env.deploy_project_root + env.deploy_current_dir))
		run("ln -s %s %s" % (env.deploy_full_path, env.deploy_project_root + env.deploy_current_dir))
	print(green("make symlink success"))


@task
def rollbask():
	print(yellow("rollback  project version"))
	versionid = input_versionid()
	if versionid == '':
		abort("project version ID error,abort")
	
	env.deploy_full_path = env.deploy_projct_root + env.deploy_release_dir + "/" + versionid
	run("rm -rf %s" % env.deploy_project_root + env.deploy_current_dir)
	run("ln -s %s %s" %(env.deploy_full_path, env.deploy_project_root + env.deploy_current_dir))
	print(green("rollback success"))

@task
	def go():
		tar_source()
		put_package()
		make_symlink()
```


# 集中化管理平台Ansible
Ansible是一种集成IP系统的配置管理，应用部署，执行特性任务的开源平台。Ansible基于Python语言实现，有Paramiko和PyYAML两个关键模块构建。
Ansible提供了一个在线Playbook分享平台[https://galaxy.ansible.com/list#/roles?page=1&page_size=100](https://galaxy.ansible.com/list#/roles?page=1&page_size=100),可以使用`ansible-galaxy install username.rolename`

## YAML语言
YAML是一种用来表达数据序列的编程语言，主要特点是:可读性强，语法简单明了，支持丰富的语言解析库，通用性强等。Ansible和SaltStack环境中配置文件都以YAML格式存在。

```
file_roots:
  base:
  - /srv/salt/
  dev:
  - /srv/salt/dev
  prod:
  - /srv/salt/prod
```

PyYAML模块
pip install pyyaml

### 块序列描述
块序列就是将描述的元素序列到Python的列表(list)中。
```
import yaml
obj = yaml.load(
"""
  - Hesperiidae
  - Papilionidae
  - Apatelodidae
  - Epiplemidae
  - 
    - a
    - b
    - c
    - d
"""
print(obj)
)
```

['H...','P...','A...','E...',['a','b','c','d']]


### 块映射描述
块映射就是将描述的元素序列到Python的字典(Dictionary)中，格式为"键(key): 值(value)"。
```
hero:
  hp: 34
  sp: 8
  level: 4
orc:
  hp: 12
  sp: 0
  level: 2
```

{'hero': {'hp': 34, 'sp': 8, 'level': 4}, 'orc': {'hp': 12, 'sp': 0, 'level': 2}}


*配置linux主机SSH无密码访问*
1. ssh-keygen -t rsa -P ''
2. ssh-copy-id -i ~/.ssh/id_rsa.pub user@ip


### ansible的hosts文件
+ 定义主机和组
[webserv]
192.168.1.1
192.168.1.2

host1 ansible_ssh_port=22 ansible_ssh_host=192.168.1.1

+ 定义主机变量
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=800 maxRequestsPerChild=909

+ 定义组变量
组变量的作用域是覆盖组所有成员，通过定义一个新块，块名由组名+":vars"组成
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com


## 匹配目标
ansible <pattern_goes_here> -m <module_name> -a <arguments>

`ansible hosts -m service -a "name=nginx state=restarted"`

在playbooks中运行远程命令格式如下:
```
- name: restart the service
  command: service nginx restart
```

ansible提供了非常丰富的模块：	
1. 远程命令模块
模块包括command,script,shell，都可以实现远程shell命令运行。command作为ansible的默认模块。script功能是在远程主机执行主控端存储的shell脚本文件，相当于scp+shell。

2. copy模块
实现主控端向目标主机拷贝文件。
ansible host1 -m copy -a "src=/home/test.sh dest=/tmp/ owner=root group=root mode=0755"

3. stat模块
获取远程文件状态信息
ansible host1 -m stat -a "path=/etc/fstab"

4. get_url模块
实现在远程主机下载指定URL到本地，支持sha256sum文件校验
ansible host1 -m get_url -a "url=http://www.zhihu.com dest=/tmp/index.html mode=0644 force=yes"

5. yum模块
软件包管理操作
ansible host1 -m yum -a "name=curl state=latest"

6. cron模块
远程主机crontab配置
ansible host1 -m cron -a "name='check dirs' hour='5,2' job='ls -alh > /dev/null'"

7. mount模块
远程主机分区挂载
ansible host1 -m mount -a "name=/mnt/data src=/dev/sr0 fstype=ext3 opts=ro state=present"

8. service模块
远程主机系统服务管理
ansible host1 -m service -a "name=nginx state=stopped/restarted/reloaded"

9. sysctl包管理模块
远程Linux主机sysctl配置

10. user服务模块
远程主机系统用户管理
ansible host1 -m user -a "name=aaron comment='aaron'"
ansible host1 -m user -a "name=aaron state=absent remove=yes"


### playbook
playbook是一个非常简单的配置管理和多主机部署系统，不同于任何已经存在的模式，可作为一个适合部署复杂应用程序的基础。

nginx.yml
```
---
- hosts: webserver
  vars:
  	worker_processes: 4
  	num_cpus: 4
  	max_open_file: 65535
  	root: /data
  remote_user: root
  tasks:
  - name: ensure nginx is at the latest version
    yum: name=nginx state=latest
  - name: write the nginx config file
  	template: src=/home/test/ansible/nginx/nginx2.conf dest=/etc/nginx/nginx.conf
  	notify:
  	- restart nginx
  - name: ensure nginx is running
  	service: name=nginx state=started
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted
```

hosts参数的作用为定义操作的对象，可以是主机或组。vars参数定义了四个变量(配置模板用到)。remote_user为指定远程操作的用户名，默认为root，支持sudo方式运行，通过添加sudo: yes即可。

所有定义的任务列表(task list),playbook将按定义的配置文件自上而下的顺序执行，定义的主机豆浆得到相同的任务。

在playbook可通过template模块对本地配置模板文件进行渲染并同步到目标主机。
nginx2.conf
```
user	nginx;
worker_processes {{ worker_processes }};
{% if num_cpus == 2 %}
worker_cpu_affinity 01 10;
{% elif num_cpus == 4 %}
worker_cpu_affinity 1000 0100 0010 0001;
{% elif num_cpus >= 8 %}
worker_cpu_affinity 10000000 ...;
{% else %}
worker_cpu_affinity 1000 0100 0010 0001;
worker_rlimit_nofile {{ max_open_file }};
......
```

当目标主机配置文件发生变化后，通知处理程序(Handlers)来触发后续的动作，比如重启nginx服务。Handlers中定义的处理程序在没有通知触发时是不会执行的，出发后也只会运行一次。触发是通过Handlers定义的name标签来识别的。


### 执行ansible-playbook
`ansible-playbook /home/test/nginx.yml -f 10`
+ -u REMOTE_USER: 手工指定远程执行playbook的系统用户
+ --syntax-check: 检查playbook的语法
+ --list-hosts playbooks: 匹配到的主机列表
+ -T TIMEOUT: 定义playbook执行超时时间
+ --step: 以单人舞分步骤运行


当多个playbook涉及复用的任务列表时，可以将复用的内容剥离出，写到独立的文件中，最后在需要的地方include进来即可

tasks/foo.yml
```
- name: placeholder foo
  command: /bin/foo
- name: placeholder bar
  command: /bin/bar
```
然后在使用的playbook中include进来:
```
tasks:
  - include: tasks/foo.yml
```

### 获取远程主机系统信息: Facts
```
ansible host1 -m setup
```

在Ansible中使用变量的目的是方便处理系统之间的差异，变量名的命名规则由字母，下划线和数字组合而成，变量必须以字母开头。


### Jinja2过滤器
使用格式: {{ 变量名|过滤方法 }}

获取文件所处的目录名:
{{ path | dirname }}

获取一个文件路径变量过滤出文件名:
{{ path | basename}}


### 本地Facts
当通过Facts获取目标主机的系统信息不能满足功能需求时，可以通过编写自定义的Facts模块来实现。在目标主机/etc/ansible/facts.d目录定义JSON，INI或可执行文件的JSON输出，文件扩展名要求使用".fact"，这些文件都可以作为ansible的本地Facts。

在目标主机/etc/ansible/facts.d/preferences.fact文件中添加如下内容:
```
[general]
max_memory_size=4096
max_user_processes=65535
open_files=65535
```
然后在主控端运行ansible host1 -m setup -a "filter=ansible_local"即可看到自定义的结果

![ansible-01.png](https://aaron-13.github.io/images/ansible-01.png)

返回JSON的层次结构，preferences(facts文件名前缀) ---> general(INI的节名) ---> key:value(INI的键值)。
在模板或playbook中通过以下方式调用:
`{{ ansible_local.preferences.general.open_files}}`


### 注册变量
变量的另一个用途是将一条命令的运行结果保存到变量中，供后面的playbook使用。
```
- hosts: webserver
  tasks:
  	- shell: /usr/bin/foo
  	  register: foo_result
  	  ignore_errors: True

  	- shell: /usr/bin/bar
  	  when: foo_result.rc == 5 (rc:returncode)


### 条件语句
```
tasks:
  - name: "shutdown Debian flavored systems"
    command: /sbin/shutdown -t now
    when: ansible_os_family == "Debian"
  - command: /bin/something
    when: result|failed
  - command: /bin/something_else
    when: result| success
```
"when: result|success"的意思为当变量result执行结果为成功状态时，将执行/bin/something_else命令


### 循环
批量化创建用户
```
- name: add several users
  users: name={{ item.name }} state=present groups={{ item.groups }}
  with_items:
  - { name: 'testuser1', groups: 'wheel'}
  - { name: 'testuser2', groups: 'root' }
```

循环支持字典和列表的形式，字典通过with_items，列表通过with_flattened。列表中变量用item表示



# 网络统一控制器Func