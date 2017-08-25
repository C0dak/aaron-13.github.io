# OpenResty

------

为了处理大量连接请求场景，需要使用非阻塞I/O和复用。select，poll和epoll是Linux API提供的I/O复用方式。Nginx使用的是epoll模型。


**select模型**

函数接口:

```
int select (int n,fd_set *readfds,fd_set *writefds,fd_set expectfds,struct timeval *timeout);
```

select其良好跨平台支持是它的一大优点。缺点在于单个进程能够监视的文件描述符的数量存在最大限制，在Linux上一般为1024。


**poll模型***
```
int poll (struct pollfd *fds, unsigned int nfds, int timeout);
```

不同于select使用三个位图来表示fdset的方式，poll使用一个pollfd的 指针来实现

```
struct pollfd {
	int fd; /*file descripto*/
	short events; /*request events to watch*/
	short revents; /*returned events witnessed*/ 
}
```

select和poll都要在返回后，遍历文件描述符来获取已经就绪的socket。


**epoll模型**

```
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
		typedef union epoll_data{
			void *ptr;
			int fd;
			__uint32_t u32;
			__uint64_t u64;
		} epoll_data_t;

		struct epoll_event {
			__uint32_t events; /*epoll events*/
			epoll_data_t data;	/*User data variable*/
		}

int epoll_wait(int epfd, struct epoll_event * events; int maxevents, int timeout);
```

epoll优点:
1.监视的描述符数量不受限制

2.IO的效率不会随着监视fd的数量的增长而下降。不同于select和poll轮询方式，而是通过每个fd定义的回调函数来实现。

3.支持水平触发和边沿触发两种模式:
- 水平触发模式，文件描述符状态改变之后，如果没有采取行动，将后面反复通知

— 边沿触发模式，只告诉哪些文件描述符刚刚变为就绪状态，只说一遍，如果没有采取行动，将不会再次告知

4.mmap加速内核与用户空间的信息传递。epoll是通过内核与用户空间mmap同一块内存，避免了无谓的内存拷贝



## 简介

------

OpenResty(ngx_openresty)是一个全功能的web应用服务器。打包了标准的Nginx核心，很多的常用的第三方模块，以及它们的大多数依赖项。

OpenResty有两大应用目标:
1.通用目的的web应用服务器。

2.Nginx的脚本扩展编程，构建灵活的web应用网关和Web应用防火墙。


**LuaJIT**
LuaJIT是采用C和汇编语言描写的Lua解释器与即时编译器。


在Lua实现中，Lua字符串一般都会经历一个"内化"(intern)过程，即两个完全一样的Lua字符串在Lua虚拟机中只会存储一份。每个Lua字符串在创建时都会插入到Lua虚拟机内部的一个全局哈希表中。

**Lua语言中"不等于"运算符的写法为: ~=**


**字符串连接**

在Lua中连接两个字符串，可以使用操作符"..",如果其任意一个操作数是数字的话，Lua会将这个数字转换为字符串。注意，连接操作符只会创建一个新字符串，而不会改变原操作数。也可以使用string库函数`string.format`连接字符串,对于很多这样的连接操作，推荐使用table和table.concat()来进行字符串的拼接。

```
print("hello" .. "world")

str = string.format("%s-%s","hello","world")
```

**Lua函数**

使用函数的好处:
1.降低程序的复杂性
2.增加程序的可读性
3.避免重复代码
4.隐含局部变量


Lua支持变长参数，若形参为...,表示该函数可以接收不同长度的参数，访问参数的时候也要使用...

```
local function func(...) 	--形参为...,表示函数采用变长参数

	local tmp={...} 	--访问的时候也要使用...
	local ans = table.concat(tmp,"") 	--使用table.concat库函数对数
										--组内容使用" "拼接成字符串
	print(ans)
end

func(1,2)	--传递两个参数
func(1,2,3,4)	--传递四个参数

--output
1 2
1 2 3 4
```


## 模块

------

一个Lua模块的数据结构使用一个Lua值(通常是一个Lua表或者Lua函数)。一个Lua模块代码就是一个会返回这个Lua值的代码块。可以使用内建函数require()来加载和缓存模块。


**require函数**
Lua提供了一个名为require的函数用来加载模块，要加载一个模块，只需要简单地调用require "file"就可以了，file指模块所在的文件名。这个调用会返回一个由模块函数组成的table，并且还会定义一个包含该table的全局变量。


**string库**

string.byte(s[,i[,j]]) 返回字符s[i]..s[j]对应的ASCII码。

string.char(...)接收0个或多个整数(0-255),返回这些整数所对应的ASCII码字符组成的字符串

string.find(s,p[,init[,plain]]) 在s字符串中第一次匹配p字符串。若成功，则返回p字符串在s字符串中出现的开始位置和结束为止，失败返回nil。第三个参数指定索引起始位置。第四个参数默认为false，当为true时，只会把p看成一个字符串对待。

string.format(formatstring,...) 按照格式化参数formatstring，返回后面...内容的格式化版本。

string.match(s,p[,init]) 在字符串s中匹配(模式)字符串p，若匹配成功，则返回目标字符串中与模式匹配的子串；否则返回nil。

**string.match目前并不能被JIT编译，应尽量使用ngx_lua模块提供的ngx.re.match等接口**

string.gmatch(s,p) 返回一个迭代器函数，通过这个迭代器函数可以遍历到在字符串s中出现模式串p的所有地方。

**此函数目前不能被LuaJIT和JIT编译，而只能被解释执行。应尽量使用ngx_lua模块提供的ngx.re.gmatch等接口。**

string.rep(s,n) 返回字符串s的n次拷贝

string.sub(s,i[,j]) 返回字符串s中，索引i到索引j之间的子字符串。

string.gsub(s,p,r[,n]) 将目标字符串s中所有子串p替换成字符串r

**此函数目前不能被LuaJIT和JIT编译，而只能被解释执行。应尽量使用ngx_lua模块提供的ngx.re.gsub等接口。**



## 正则表达式

------

Lua中正则表达式的性能并不如ngx.re.*中的正则表达式优秀
Lua中的正则表达式并不符合POSIX规范，而ngx.re.*中实现的是标准的POSIX规范
ngx.re.*中的正则表达式可以通过参数缓存编译过后的Pattern。

ngx.re.*中的o选项，指明该参数，被编译的Pattern将会在工作进程中缓存，并且被当前工作进程的每次请求所共享。Pattern缓存的上限值通过`lua_regex_cache_max_entries`来修改。

ngx.re.*中的j选项，指明该参数，如果使用PCRE库支持JIT，OpenResty会在编译Pattern时启用JIT。如果系统自带的PCRE库不支持JIT，出于性能考虑，最好编译一份libpcre.so，然后在编译OpenResty时链接过去。验证当前PCRE库是否支持JIT，可以这样做:
1.编译OpenResty时在./configure中指定--with-debug
2.在error_log指令中指定日志级别为debug
3.运行正则匹配代码，查看日志中是否有pcre JIT compiling result: 1



## 虚变量

------

当一个方法返回多个值时，有些返回值有时候用不到，要是声明很多变量来一一接收，显然不太适合，Lua提供了一个虚变量(dummy variable),以单个下划线("_")来命名，用它来丢弃不需要的数值，仅仅起到占位作用。



## 模块调用

------ 

```
-- square.lua 长方形模块
local _M = {}           -- 局部的变量
_M._VERSION = '1.0'     -- 模块版本

local mt = { __index = _M }

function _M.new(self, width, height)
    return setmetatable({ width=width, height=height }, mt)
end

function _M.get_square(self)
    return self.width * self.height
end

function _M.get_circumference(self)
    return (self.width + self.height) * 2
end

return _M
```

引用示例:

```
local square = require "square"

local s1 = square:new(1,2)
print(s1:get_square()) 			-- output: 2
print(s1:get_circumference()) 	-- output: 6	
```

当lua_code_cache on开启时，require加载的模块是会被缓存下来的，可直接显示调用:
```
package.loaded["square"] = nil
```


## 点号与冒号操作符的区别

------

```
local str = "abcde"
print("case 1:", str:sub(1, 2))
print("case 2:", str.sub(str, 1, 2))
```

执行结果
```
case 1: ab
case 2: ab
```

冒号操作会带入一个self参数，用来代表自己。而点号操作，只是内容的展开。在函数定义时，使用冒号将默认接收一个self参数，而使用点号则需要显示传入self参数。



## Nginx

------

**特点:**

1.处理响应请求很快

2.高并发连接

3.低的内存消耗

4.具有很高的可靠性

5.高扩展性

6.热部署

7.自由的BSD许可协议


**location匹配规则**

|模式 | 含义 |
|:---:|:---:|
| location = /uri | = 表示精确匹配，只有完全匹配上才能生效 |
| location ^~ /uri | ^~开头对URL路径进行前缀匹配，并且在正则之前 |
| location ~ pattern| 开头表示区分大小写的正则匹配 |
| location ~* pattern| 开头表示不分区大小写的正则匹配|
| location /uri| 不带任何修饰符，也表示左前缀匹配，但在正则匹配之后|
| location / | 通用匹配，任何未匹配到其他location的请求都会匹配到|

匹配顺序: =,^~,~/~*,不带修饰符匹配,通用匹配/

```
# 直接匹配网站根
location = / {
	proxy_pass http://tomcat:8080/index
}

# 处理静态文件请求
location ^~ /static/ {
	root /webroot/static;
}

location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
	root /webroot/res/;
}

# 通用规则匹配
location / {
	proxy_pass http://tomcat:8080/
}

```


**rewrite语法**

+ last 基本上都是用这个Flag

+ break 中止rewrite，不再继续匹配

+ redirect 返回临时重定向的HTTP状态302

+ permanent 返回永久重定向的HTTP状态301


```
server {
	listen 80;
	server_name start.igrow.cn;
	index index.html index.php;
	root html;

	if ($http_host !~ "^start\.igrow\.cn$") {
		rewrite ^(.*) http://start.igrow.cn$1 redirect;
	}
}

```

防盗链
```
location ~* \.(gif|jpg|swf)$ {
	valid_referers none blocked start.igrow.cn sta.igrow.cn;
	if ($invalid_referers) {
		rewrite ^/ http://$host/logo.png;
	}
}
```


## Nginx静态文件服务

------

```
http {
	# 设置缓存文件数量及失效时间
	open_file_cache max=204800 inactive=20s；

	# 最少使用次数
	open_file_cache_min_uses 1；

	# 多长时间检查一次
	open_file_cache_valid 30s;

	# 压缩
	gzip on;
	gzip_min_length 1k;
	gzip_buffers 16 64k;
	gzip_http_version 1.1;
	gzip_comp_level 6;
	gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/javascript;
	gzip_vary on; //和http头有关系，加个vary头，给代理服务器使用，有的浏览器支持压缩，有的不支持，避免不支持的也压缩，根据客户端的http头来判断，是否需要压缩
}
```


nginx可以分发缓存中的陈旧内容，这种情况一般发生在关联缓存内容的原始服务器宕机或者繁忙时。

```
location / {
	proxy_cache_use_stale error timeout http_500 http_502 http__503 http_504;
}
```


nginx提供了丰富的可选项配置用于缓存性能的微调。

1.proxy_cache_revalidate: 指示Nginx在刷新来自服务器的内容时使用GET请求

2.proxy_cache_min_uses: 该指令设置同一链接请求达到几次即被缓存，默认值是1

3.proxy_cache_use_stale:告知Nginx在客户端请求的项目的更新正在原服务器中下载时发送旧内容，而不是向服务器转发重复的请求。

4.当proxy_cache_lock被启用时，当多个客户端请求一个缓冲中不存在的文件，只有这些请求中的第一个被允许发送至服务器


**upstream支持的负载均衡算法**

+ 轮询: 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响

+ ip_hash: 每个请求按访问IP的hash结果分配，这样来自同一个IP的访客访问一个后端服务器，解决一部分的session共享问题

+ url_hash: 此方法按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，进一步提高后端缓存服务器的效率

+ least_conn: 最少连接负载均衡算法

+ hash: 普通hash和一致性hash


```
location ~* \.php$ {
	fastcgi_pass backend;
}
```

每个以.php结尾的请求，都会传递给FastCGI的后台处理程序，这样做的问题是，当完整的路径未能指向文件系统里面的一个确切文件时，默认的PHP配置试图猜测想执行的是哪个文件。
如: 如果一个请求中的/forum/avatar/123.jpg/file.php文件不存在，但是/forum/avatar/123.jpg存在，那么PHP解释器就会取而代之，如果里面嵌入了PHP代码，这段代码就会被执行。

有几个避免这种情况的选择:
+ 在php.ini中设置cgi.fix_pathinfo=0，这样php解释器只尝试给定的文件路径，如果没有找到就停止处理

+ 确保Nginx只传递指定的PHP文件去执行
```
location ~* (file_a|file_b|file_c)\.php$ {
	fastcgi_pass backend;
}
```

+ 对于任何用户可以上传的目录，特别的关闭php文件的执行权限
```
location /upload {
	location ~ \.php$ { return 403;}
}
```

+ 使用try_files指令过滤文件不存在的情况
```
location ~* \.php$ {
	try_files $uri =404;
	fastcgi_pass backend;
}
```

+ 使用嵌套的location过滤出文件不存在的情况
```
location ~* \.php$ {
	location ~ \..*/.*\.php$ {return 404;}
	fastcgi_pass backend;
}
```

**在HTTPS中使用SSLv3**

由于SSLv的POODLE漏洞，建议不要在开启SSL的网站使用SSLv，可以直接禁用SSLv3，用TLS来替代
```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
```




## 安装OpenResty

------

包管理安装
```
yum-config-manager --add-repo https://openresty.org/yum/cn/centos/OpenResty.repo

yum install openresty
```

源码包安装
```
相关依赖包
yum install readline-devel pcre-devel openssl-devel perl

编译安装openrestry
```
# ./configure --prefix=/opt/openresty\
            --with-luajit\
            --without-http_redis2_module \
            --with-http_iconv_module

gmake && gmake install
```



## Hello World

------

创建一个openresty的工作目录用来测试
```
mkdir /home/openresty/{conf,log} -p

在conf目录下创建一个nginx.conf文件

worker_processes auto;

error_log log/error.log;

events {
	worker_connections 1024;
}

http {
	server {
		listen 9999;
		location / {
			default_type text/html;

		# if openresty version under 1.9.3.1 please use "content_by_lua" repalce
		content_by_lua_block {
			ngx.say("hello world")
		}	
		}
	}
}
```

使用`/usr/local/openresty/nginx/sbin/nginx -p /home/openresty`启动服务



### 内部调用

------

例如对数据库，内部公共函数的统一接口，可以把它们放到统一的location中，通常情况下，为了保护这些内部接口，都会把这些接口设置为internal，这么做的最主要的好处就是可以让这个内部接口相对独立，不受外界干扰。

```
location = /sum {
	# 只允许内部调用
	internal;

	content_by_lua_block {
		local args = ngx.req.get_uri_args()
		ngx.say(tonumber(args.a) + tonumber(args.b))
	}
}

location = /app/test {
	content_by_lua_block {
		local res = ngx.location.capture ( "/sum",{args={a=4,b=5}}
		)
	ngx.say("status: ", res.status, "response: ", res.body )
	}
}
```


```
location /sum {
	internal;

	content_by_lua_block {
		ngx.sleep(0.1)
		local args = ngx.req.get_uri_args()
		ngx.print(tonumber(args.a) + tonumber(args.b))
	}
}

location /subdection {
	internal;
	content_by_lua_block {
		ngx.sleep(o.1)
		local args = ngx.req.get_uri_args()
		ngx.print(tonumber(args.a) - tonumber(args.b))
	}
}

location /app/test_parallels {
	content_by_lua_block {
		local start_time = ngx.now()
		local res1,res2 = ngx.location.capture_multi ({
		{"/sum",{args={a=3,b=5}}},
		{"/subdection",{args={a=5,b=2}}}
		})
		ngx.say("status: ",res1.status," response: ",res1.body)
		ngx.say("status: ",res2.status," response:",res2.body)
		ngx.say("time: ",ngx.now() - start_time)
	}
}

location /app/test_queue {
	content_by_lua_block {
		ngx.sleep(0.1)
		local start_time = ngx.now()
		local res1 = ngx.location.capture_multi({{"/sum",{args={a=3,b=5}}}})
		local res2 = ngx.location.capture_multi({{"/subdection",{args={a=5,b=2}}})
		ngx.say("status: ",res1.status," response: ",res1.body)
		ngx.say("status: ",res2.status," response: ",res2.body)
		ngx.say("time: ",ngx.now() - start_time)
	}
}
```

利用ngx.location.capture_multi函数，直接完成两个子请求并行执行。当两个请求没有相互依赖，这种方法可以极大提高查询效率。


```
location  ~ ^/static/([_-a-zA-Z0-9/]+).jpg {
	set $image_name $1;
	content_by_lua_block {
		ngx.exec("/download_internal/images".. ngx.var.image_name .. ".jpg")
	}
}

location /download_internal {
	internal;
	alias ../download;
}
```

ngx.exec方法和ngx.redirect是完全不同的，前者是纯粹的内部跳转并且没有引入任何任何额外的HTTP信号。

