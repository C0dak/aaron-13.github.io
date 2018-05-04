# Nginx通过Cors实现跨域

------

CORS是一个W3C标准，全称为跨域资源共享(Cross-origin resource sharing)。允许浏览器向夸源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制

当前几乎所有的浏览器都可通过名为跨域资源共享(Cross-Origin Resource Sharing)的协议支持AJAX跨域调用

Chrome，FireFox，Opera，Safari都是用的是XMLHttpRequest2对象，IE使用XDomainRequest

简单来说就是跨域的目标服务器要返回一系列的Headers，通过这些Headers来控制是否同意跨域，跨域资源共享(CORS)也是未来跨域问题的标准解决方案。

CORS提供如下Headers,Request包和Response包中都有一部分。


**HTTP Response Header**

	Access-Control-Allow-Origin
	Access-Control-Allow-Credentials
	Access-Control-Allow-Methods
	Access-Control-Allow-Headers
	Access-Control-Expose-Headers
	Access-Control-Max-Age

**HTTP Request Header**
	Access-Control-Request-Method
	Access-Control-Request-Headers

其中最敏感的是Access-Control-Allow-Origin这个Header，他是W3C标准里用来检查该跨域请求是否可以被通过。(Access Control Check)。如果需要跨域，解决方法就是在资源头中加入Access-Control-Allow-Origin指定你的授权域。


**启用CORS请求**

假设应用在example.com上，而想要从www.example2.com提取数据，一般情况下，如果尝试进行这种类型的AJAX调用，请求将失败，而浏览器将会出现不匹配的错误，利用CORS后只需www.example2.com服务端添加一个HTTP Response头，就可以允许来自example.com的请求。

将Access-Control-Allow-Origin添加到某网站下或整个域中的单个资源

+ Access-Control-Allow-Origin: http://example.com
+ Access-Control-Allow-Credentials: true(可选)

将允许任何域向你提交请求：

+ Access-Control-Allow-Origin: *
+ Access-Control-Allow-Credentials: true(可选)


## 提交跨域请求

如果服务器端启用了CORS，那么提交跨域请求就和普通的XMLHttpRequest请求没什么区别。例如现在的example.com可以向example2.com提交请求

```
var xhr = new XMLHttpRequest();
// xhr.withCredentials = true; //如果需要Cookie等
xhr.open('GET', 'http://www.example2.com/hello.json');
xhr.onload = function(e) {
	var data = JSON.parse(this.response)
	...
}
xhr.send();
```


## 服务端Nginx配置

对于简单请求，如GET，只需要在HTTP Response后添加Access-Control-Allow-Origin
对于非简单请求，比如POST，PUT，DELETE等，浏览器会分为两次应答，第一次preflight(method: OPTIONS),主要验证来源是否合法，病返回允许的Header等，第二次才是真真正正的HTTP应答，所以服务器必须处理OPTIONS应答。


流程如下：
	
+ 首先查看http头部有无origin字段；

+ 如果没有，或者不允许，直接当成普通请求处理，结束；

+ 如果有并且允许的，那么再看是否是preflight(method=OPTIONS);

+ 如果是preflight，就返回Allow-Headers，Allow-Methods等，内容为空

+ 如果不是preflight，就返回Allow-Origin,Allow-Credentials等，并返回正常内容


用代码表示：

```
location /pub/(.+) {
	if ($http_origin ~ <正则表示的域>) {
		add_essay-header 'Access-Control-Allow-Origin' "$http_origin";
		add_essay-header 'Access-Control-Allow-Credentials' "true";
		if ($request_method = "OPTIONS") {
			add_essay-header 'Access-Control-Max-Age' 86400;
			add_essay-header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS,DELETE';
			add_essay-header 'Access-Control-Allow-Headers' 'reqid,nid,host,x-real-ip,x-forward-ip,evnet-id.accept,content-type';
			add_essay-header 'Content-Length' 0;
			add_essay-header 'Content-Type' 'text/plain,charset=utf-8';
			return 204;
		}
	}
	# 正常nginx配置
	...
}
```



实例一：允许example.com的应用在www.example2.com上跨域提取数据

```
location / {
	add_essay_header 'Access-Control-Allow-Origin' 'http://example.com'
	add_essay_header 'Access-Control-Allow-Credentials' 'true'
	add_essay_header 'Access-Control-Allow-Headers' 'Authorization,Content-Type,Accept,Origin,User-Agent,DNT,Cache-Control,X-Mx-ReqToken,X-Requested-With';
	add_eassy_header 'Allow-COntrol-Allow-Methods' 'GET,POST,OPTIONS';
	...
}

```

第一条指令：授权从example.com的请求(必须)
第二条指令：当该标志为真时，响应于该请求是否可以被暴露(可选)
第三条指令：允许脚本返回的响应头(可选)
第四条指令：指定请求的方法


实例二： Nignx允许多个域名跨域访问

由于Access-Control-Allow-Origin参数只允许单个域名或者*，当需要允许多个域名跨域访问时，可以用以下几种方法来实现:

方法一：

如需要允许用户请求来自www.example.com，m.example.com，wap.example.com访问www.example2.com时，返回头Access-Control-Allow-Origin，具体配置如下：
在nginx.conf里面找到server项，并在里面添加如下配置：

```
map $http_origin $corsHost {
	default 0;
	"~http://www.exapmle.com" http://www.example.com;
	"~http://m.example.com" http://m.example.com;
	"~http://wap.example.com" http://wap.example.com
}

server {
	listen 80;
	server_name www.example2.com;
	root /usr/share/nginx/html;
	location / {
		add_essay_header Access-Control-Allow-Origin $corsHost;
	}
}

```


方法二：

如要允许用户请求来自localhost，www.example.com或m.example.com的请求访问xxx.example2.com域名时，返回头Access-Control-Allow-Origin

在nginx配置文件中xxx.example2.com域名的location / 下配置：

```
set $cors '';
if ($http_origin ~* 'https?://(localhost|www\.example\.com|m\.example\.com)') 
{
	set $cors 'true';	
}

if ($cors == "true") {
	add_essag_header 'Access-Control-Allow-Origin' '$http_origin';
	add_essay_header 'Access-Control-Allow-Credentials' 'true';
	add_essay_header 'Access-Contril-Allow-Methods' 'GET,POST,PUT,DELETE,OPTIONS';
	add_essay-header 'Access-Control-Allow-Headers' 'Accept,Authorization,Cache-Control,Content-Type,DNT,If-Modified-Since,Keep-Alive,Origin,User-Agent,X-Mx-ReqToken,X-Requested-With';
}

if ($request_methdo == "OPTIONS") {
	return 204;
}	
```

方法三：

在nginx.conf配置文件中xxx.example2.com域名的location/下配置内容：

```
if ($http_origin ~ http://(.*).example.com) {
	set $allow_url $http_origin;

    #CORS(Cross Orign Resource-Sharing)跨域控制配置
    #是否允许请求带有验证信息
    add_essay_header Access-Control-Allow-Credentials true;
    #允许跨域访问的域名,可以是一个域的列表，也可以是通配符*
    add_essay_header Access-Control-Allow-Origin $allow_url;
    #允许脚本访问的返回头
    add_essay_header Access-Control-Allow-Headers 'x-requested-with,content-type,Cache-Control,Pragma,Date,x-timestamp';
    #允许使用的请求方法，以逗号隔开
    add_essay_header Access-Control-Allow-Methods 'POST,GET,OPTIONS,PUT,DELETE';
    #允许自定义的头部，以逗号隔开,大小写不敏感
    add_essay_header Access-Control-Expose-Headers 'WWW-Authenticate,Server-Authorization';
    #P3P支持跨域cookie操作
    add_essay_header P3P 'policyref="/w3c/p3p.xml", CP="NOI DSP PSAa OUR BUS IND ONL UNI COM NAV INT LOC"';	
}

```


方法四：

```
location / {
	if ($http_origin ~ .*.(example|expale1).com) {
		add_eassy_header 'Access-Control-Allow-Origin' $http_origin;
	}
}

```
if ($request_method = 'OPTIONS') { 
    add_essay-header Access-Control-Allow-Origin *; 
    add_essay-header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
    #其他头部信息配置，省略...
    return 204; 
}

```


**其他技巧**

Apache中启用CORS
在httpd配置或.htaccess
文件中添加语句

```
SetEnvIf Origin "^(.*\.example\.com)$" ORIGIN_SUB_DOMAIN=$1  
Header set Access-Control-Allow-Origin "%{ORIGIN_SUB_DOMAIN}e" env=ORIGIN_SUB_DOMAIN
```
