# Beautiful Soup文档

------

Beautiful Soup是一个可以从HTML或XML文件中提取数据的Python库，它能够通过你喜欢的转换器实现惯用的文档导航，查找，修改文档的方式。

![soup-01.png](https://aaron-13.github.io/images/soup-01.png)

使用BeautifulSoup解析这段代码，能够得到一个BeautifulSoup的对象，并能按照标准的缩进格式的结构输出：

```python

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc,"html.parser")

print(soup.prettify())
```

![soup-02.png](https://aaron-13.github.io/images/soup-02.png)


从文档中找到所有<a>标签的链接：

```python

for link in soup.find_all('a'):
	print(link.get('href'))
```
![soup-04.png](https://aaron-13.github.io/images/soup-04.png)


从文档中获取所有文字内容：

```python

print(soup.get_text())
```
![soup-03.png](https://aaron-13.github.io/images/soup-03.png)



------


## 安装Beautiful Soup

Debian或ubuntu系统：

`apt-get install Python-bs4`

使用pip或者easy_install进行安装

`easy_install beautifulsoup4`

`pip install beautifulsoup4`

通过源码进行安装
[下载BS4源码](http://www.crummy.com/software/BeautifulSoup/download/4.x/)，然后通过setup.py来安装
`python setup.py install`


**安装完成后的问题**

如果代码抛出了ImportError的异常:"No module named HTMLParser",这是因为在Python3的版本中执行了Python2版本的额代码

如果代码抛出了ImportError的异常:"No module named html.parser"，这是因为在Python2版本中执行Python3版本的代码。

如果在ROOT_TAG_NAME=u'[document]'代码处遇到SyntaxError "Invalid syntax"错误，需要将把BS4的Python代码版本从Python转换到Python3，可以重新安装BS4：

`Python3 setup.py install`

或在bs4的目录中执行Python代码版本转换脚本

`2to3-3.2 -w bs4`


## 安装解析器

BeautifulSoup支持Python标准库中的HTML解析器，还支持一些第三方的解析器，其中一个是lxml，根据操作系统不同，可以选择下列方法来安装lxml；

`apt-get install Python-lxml`

`easy_install lxml`

`pip install lxml`

另一个可供选择的解析器是纯Python实现的html5lib，html5lib的解析方式与浏览器相同，可以选择下列方法安装：

`apt-get install Python-html5lib`

`easy_install html5lib`

`pip install html5lib`


下表列出了主要的解析器，以及它们的优缺点：

| 解析器 | 使用方法 | 优势 | 劣势 |
| :---:  | :---:    | :--: | :--: |
| Python标准库| BeautifulSoup(markup,'html.parser') | Python的内置标准库，执行速度适中，文档容错能力强 | Python2.7.3or3.2.2千的版本中文档容错能力差 |
| lxml HTML解析器 | BeautifulSoup(markup,"lxml") | 速度快，文档容错能力强 | 需要安装C语言库 |
| lxml XML解析器 | BeautifulSoup(markup,"xml") | 速度快，唯一支持XML的解析器 | 需要安装C语言库 |
|html5lib | BeautifulSoup(markup,"html5lib") | 最好的容错性，以浏览器的方式解析文档，生成HTML5格式的文档 | 速度慢，不依赖外部扩展 | 


推荐使用lxml作为解析器，因为效率高，早Python2.7.3之前的版本和Python3中3.2.2之前的版本，必须安装lxml或html5lib，因为那些Python版本的标准库中内置的HTML解析方法不够稳定

提示：如果一段HTML或XML文档格式不正确的话，那么在不同的解析器中返回的结果可能是不一样的


------

## 如何使用

将一段文档传入BeautifulSoup的构造方法，就能得到一个文档的对象，可以传入一段字符串或一个文件句柄

```
from bs4 import BeautifulSoup
soup = BeautifulSoup(open("index.html"))
soup = BeautifulSoup("<html>data<html>")

```

首先，文档被装换成Unicode，并且HTML的实例都被转换成Unicode编码

```
BeautifulSoup("Sacr bleu!")
<html><head></head><body>Sacr bleu</body>
```

------

## 对象的种类

BeautifulSoup将复杂HTML文档转换成一个复杂的树形结构，每个字节都是python对象，所有对象可以归纳为4中:`Tag`,`NavigableString`,`BeautifulSoup`,`Comment`

**Tag**

Tag对象与XML或HTML原生文档中的tag相同:

```
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)

<class 'bs4.element.Tag'>
```
![soup-05.png](https://aaron-13.github.io/images/soup-05.png)


Tag有很多方法和属性，介绍下最重要的两个属性：name和attributes

```
tag.name
# b
```

如果改变了tag的name，那将影响所有通过当前BeautifulSoup对象生成的HTML文档:

```
tag.name = "block"
# <block class="boldest">Extremely bold</block>
```


**Attributes**

一个tag可能有很多个属性.tag<b class="boldest">有一个"class"属性，值为"boldest"。tag的属性的操作方法与字典相同

```
tag['class']
# ['boldest']
```

也可以直接"点"取属性

```
tag.attrs
# {'class':['boldest']}
```

tag的属性可以被添加，删除或修改

```
tag['class'] = 'verybold'
tag['id'] = 1
tag
# <block class='verybold' id=1>Extremely bold</block>

del tag['class']
del tag['id']
tag
# <block>Extremely bold</block>

tag['class']
# KeyError: 'class'

print(tag.get('class'))
# None

```


**多值属性**

HTML4定义了一系列可以包含多个值得属性，在HTML5中移除了一些，却增加更多，最常见的多值的属性是class(一个tag可以有多个CSS的class)。还有一些属性`rel`,`rev`,`accept-charset`,`headers`,`accesskey`。在BeautifulSoup中多值属性的返回类型是list:

```
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.p['class']
# ["body","strikeout"]

css_soup = BeautifulSoup('<p class="body"></p>')
css_soup.p['class']
# ['body']

```

如果某个属性看起来好像有多个值，但在任何版本的HTML定义中都没有被定义为多值属性，那么BeautifulSoup会将这个属性作为字符串返回


```
id_soup = BeautifulSoup('<p id="my id"></p>')
id_soup.p["id"]
# 'my id'
```

将tag转换成字符串时，多值属性会合并为一个值：

![soup-06.png](https://aaron-13.github.io/images/soup-06.png)


如果转换的文档是XML格式，那么tag中不包含多值属性


![soup-07.png](https://aaron-13.github.io/images/soup-07.png)


**可以遍历的字符串**

字符串常被包含在tag内。BeautifulSoup用`NavigableString`类来包装tag中的字符串：

![soup-08.png](https://aaron-13.github.io/images/soup-08.png)

一个NavigableString字符串与Python中的Unicode字符串相同，并且还支持包含遍历文档树和搜索文档树中的一些特性。通过Unicode()方法直接将NavigableString对象转换成Unicode字符串：

![soup-09.png](https://aaron-13.github.io/images/soup-09.png)


tag中包含的字符串不能编辑，但是可以被替换成其他的字符串，用replace_with()方法：

![soup-10.png](https://aaron-13.github.io/images/soup-10.png)


NavigableString对象支持遍历文档树和搜索文档树中的大部分属性，并非全部，尤其是，一个字符串不能包含其他内容(tag能够包含其他字符串或其他tag)，字符串不支持`.contents`或`.string`属性或是`find()`方法

如果想要在BeautifulSoup之外使用`NavigableString`对象，需要使用`unicode()`方法，将该对象转换成普通的Unicode字符串，，否则就算BeautifulSoup方法已经执行结束，该对象的输出也会带有对象的引用地址，这样会浪费内存。


**BeautifulSoup**

BeautifulSoup对象表示的是一个文档的全部内容，大部分时候，可以把他当做Tag对象，它支持遍历文档树和搜索文档树种描述的大部分方法。
因为BeautifulSoup对象并不是真正的HTML或XML的tag，所以他没有name和attribute属性，但有时查看它的`.name`属性是很方便的，所以BeautifulSoup对象包含了一个值为"[document]"的特殊属性`.name`

![soup-11.png](https://aaron-13.github.io/images/soup-11.png)


**注释及特殊字符串**

`Tag`，`NavigableString`，`BeautifulSoup`几乎覆盖了html和xml中的所有内容，但是还有一些特殊对象，容易让人担心的内容是文档的注释部分：

![soup-12.png](https://aaron-13.github.io/images/soup-12.png)

Comment对象是一个特殊类型的NavigableString对象：

```
comment
# u'hello'

```

当它出现在HTML文档中时，Comment对象会使用特殊的格式输出：

```
print(soup.b.prettify())
# <b>
# <!--hello-->
# </b>
```

BeautifulSoup中定义的其他类型都可能会出现在XML的文档中：`CData`,`ProcessiongInstruction`,`Declaration`,`Doctype`与`Comment`对象类似，这些类都是`NavigableString`的子类，只是添加一些额外的方法的字符串独享，下面是用CDATA来代替注释的例子：

![soup-14.png](https://aaron-13.github.io/images/soup-14.png)


------


## 遍历文档树

```
html_doc = """<html><head><title>The Dormouse's story</title><li>loop</li></head>
    <body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')
```

**子节点**

一个Tag可能包含多个字符串或其他的Tag，这些都是这个Tag的子节点。BeautifulSoup提供了许多操作和遍历子节点的属性。

注意: BeautifulSoup中字符串节点不支持这些属性，因为字符串没有子节点


**tag的名字**

操作文档树最简单的方法就是告诉它你想获取的tag的name，如果想获取<head>标签，只要用soup.head:

```
soup.head
# <head><title>aaron</title></head>

soup.title
# <title>aaron</title>

```

这是获取tag的小窍门，可以在文档树的tag中多次调用这个方法。下面的代码可以获取<body>标签中的第一个<b>标签：

```
soup.body.b
# <b>aaron</b>
```

通过点取属性的方式只能 获得当前名字的第一个tag：

```
soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```

如果想要得到所有的<a>标签，或是通过名字得到比一个tag更多的内容的时候，就需要用到Searching the tree中描述的方法，比如find_all()

```
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

**.contents和.children**

tag的`.contents`属性可以将tag的子节点以列表的方式输出:

![soup-16.png](https://aaron-13.github.io/images/soup-16.png)

BeautifulSoup对象本身一定会包含子节点，也就是说<html>标签也是BeautifulSoup对象的子节点：

![soup-17.png](https://aaron-13.github.io/images/soup-17.png)

字符串没有`.contents`属性，因为字符串没有子节点：

通过tag的`.children`生成器，可以对tag的子节点进行循环：

```
for child in title_tag.children
	print(child)
# the dormouse's story

```

**.descendants**

`.contents`和`.children`属于仅包含tag的直接子节点，例如，<head>标签只有一个直接子节点<title>

```
head.contents
# <title>The Dormousee's story</title>
```

但是<title>标签也包含一个子节点：字符串"The Dormouse's story"，这种情况下字符串也属于<head>标签的子孙节点。`.descendants`属于可以对所有tag的子孙节点进行递归：

![soup-18.png](https://aaron-13.github.io/images/soup-18.png)


**.string**

如果tag只有一个`NavigableString`类型子节点，那么这个tag可以使用`.strirng`得到子节点

```
title.string
# The Dormouse's story
```

如果一个tag仅有一个子节点，那么这个tag也可以使用.string方法，输出结果与当前唯一子节点的.string结果相同

```
title.contents
# <title>The Dormouse's story</title>

title.string
# The Dormouse's story

```

如果tag包含了多个子节点，tag就无法确定.string方法应该调用哪个子节点的内容，输出结果为`None`


**.strings和stripped_strings**

如果tag中包含多个字符串，可以使用.strings来循环获取：

```python

for string in soup.strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u'\n\n'
    # u"The Dormouse's story"
    # u'\n\n'
    # u'Once upon a time there were three little sisters; and their names were\n'
    # u'Elsie'
    # u',\n'
    # u'Lacie'
    # u' and\n'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'\n\n'
    # u'...'
    # u'\n'
```

输出的字符串中可能包含很多空格或空行，使用`stripped_strings`可以去除多余空白内容

```
for string in soup.stripped_strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u"The Dormouse's story"
    # u'Once upon a time there were three little sisters; and their names were'
    # u'Elsie'
    # u','
    # u'Lacie'
    # u'and'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'...'
```
全部是空格的行会被忽略掉，段首和段末的空白会被删除


## 父节点

每个tag或字符串都有父节点：被包含在某个tag中


**.parent**
通过.parent属性来获取某个元素的父节点

```
title = soup.title
title
# <title>The Dormouse's story</title>
title.parent
# <head><title>The Dormouse's story</title></head>
```

文档title的字符串也有父节点：<title>标签

```
title.string.parent
# <title>The Dormouse's story</title>
```

文档的顶层节点比如<html>的父节点是BeautifulSoup对象：

```
html = soup.html
type(html)
# <class 'bs4.BeautifulSoup'>
```

BeautifulSoup对象的.parent是`None`


**.parents**

通过元素的.parents属性可以递归得到元素的所有父节点，使用.parents遍历<a>标签到根节点的所有节点

![soup-19.png](https://aaron-13.github.io/images/soup-19.png)


## 兄弟节点

```
sibling_soup = BeautifulSoup("<a><b>text1</b><c>text2</c></b></a>")
print(sibling_soup.prettify())
# <html>
#  <body>
#   <a>
#    <b>
#     text1
#    </b>
#    <c>
#     text2
#    </c>
#   </a>
#  </body>
# </html>
```

因为<b>和<c>标签是同一层，他们是同一个元素的子节点，，可以被称为兄弟节点。一段文档以标准格式输出时，兄弟节点有相同的缩进级别，在代码中也可以使用这种关系

**.netx_sibling和.previous_sibling**

在文档中，使用`.next_sibling`和`.previous_sibling`属性来查询兄弟节点

通过这两个属性可以对当前节点迭代输出：

```
for sibling in soup.a.next_siblings:
    print(repr(sibling))
    # u',\n'
    # <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
    # u' and\n'
    # <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
    # u'; and they lived at the bottom of a well.'
    # None

for sibling in soup.find(id="link3").previous_siblings:
    print(repr(sibling))
    # ' and\n'
    # <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
    # u',\n'
    # <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
    # u'Once upon a time there were three little sisters; and their names were\n'
    # None
```


## 回退和前进

```
<html><head><title>The Dormouse's story</title></head>
<p class="title"><b>The Dormouse's story</b></p>
```

**.netx_element和.previous_element**

`.netx_element`属性指向解析过程中下一个被解析的对象(字符串或tag),结果可能与`.next_element`相同，但通常不一样。

它的`.next_element`结果是一个字符串，因为当前的解析过程因为遇到<a>标签而中断了
```
last_a_tag = soup.find("a", id="link3")
last_a_tag
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

last_a_tag.next_sibling
# '; and they lived at the bottom of a well.'
```

但是这个<a>标签的`.next_element`属性结果是在<a>标签被解析之后的解析内容，不是<a>标签后的句子部分，应该是字符串"Tillie":

```
last_a_tag.next_element
# u'Tillie'
```

这是因为在原始文档中，字符串"Tillie"在分号前出现，解析器先进入<a>标签，然后是字符串"Tillie"，然后关闭</a>标签，然后是分号和剩余部分。分号与<a>标签在统一层级，但是字符串"Tillie"会被先解析



## 搜索文档树

BeautifulSoup定义了很多搜索方法，这里着重介绍2个：find()和find_all()


**过滤器**

**字符串**

最简单的过滤器是字符串，在搜索方法中传入一个字符串参数，BeautifulSoup会查找与字符串完整匹配的内容哦，下面的例子用于查找文档中所有的<b>标签：

```
soup.find_all('b')
# [<b>The Dormouse's story</b>]
```

如果传入字节码参数，BeautifulSoup会当作UTF-8编码，可以传入一段Unicode编码来避免BeautifulSoup解析编码出错


**正则表达式**

如果传入正则表达式作为参数，BeautifulSoup会通过正则表达式的match()来匹配内容：

![soup-20.png](https://aaron-13.github.io/images/soup-20.png)

![soup-21.png](https://aaron-13.github.io/images/soup-21.png)


## 列表

如果传入列表参数，BeautifulSoup会将与列表中任一元素匹配的内容返回

![soup-22.png](https://aaron-13.github.io/images/soup-22.png)


**True**

True可以配置任何值，查找所有的tag，但是不会返回字符串节点

![soup-23.png](https://aaron-13.github.io/images/soup-23.png)


## 方法

如果没有合适的过滤器，那么还可以定义一个方法，方法只接受一个元素参数，如果这个方法返回True，表示当前元素匹配并且被找到，如果不是则返回False

```
def has_class_but_no_id(tag):
	return tag.has_attr('class') and not tag.has_attr('id')
```

将这个方法作为参数传入find_all()方法，将得到所有<p>标签：
```
soup.find_all(has_class_but_no_id)
```

通过一个方法来过滤一类标签属性的时候，这个方法的参数是要被过滤的属性的值，而不是这个标签。

找出href属性不符合指定正则的a标签

![soup-24.png](https://aaron-13.github.io/images/soup-24.png)

标签过滤方法可以使用复杂方法，下面例子可以过滤出前后都有文字的标签：

![soup-25.png](https://aaron-13.github.io/images/soup-25.png)


## find_all()

**find_all(name,attrs,recursive,string,\*\*kwargs)**

find_all()方法搜索当前tag的所有tag子节点，并判断是否符合过滤器的条件

```
soup.find_all("title")
# [<title>The Dormouse's story</title>]

soup.find_all("p", "title")
# [<p class="title"><b>The Dormouse's story</b></p>]

soup.find_all("a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find_all(id="link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

import re
soup.find(string=re.compile("sisters"))
# u'Once upon a time there were three little sisters; and their names were\n'
```

**name参数**

name参数可以查找所有名字为name的tag，字符串对象会被自动忽略掉
简单用法：

```
soup.find_all("title")
# [<title>The Dormouse's story</title>]
```

重申：搜索name参数的值可以使任一类型的过滤器，字符串，正则表达式，列表，方法或是True


**keyword参数**

如果一个指定名字的参数不是搜索内置的参数名，搜索时会把该参数当做指定名字tag的属性来搜索，如果包含一个名字为id的参数，BeautifulSoup会搜索每个tag的"id"属性：

```
soup.find_all(id="link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

如果传入href参数，BeautifulSoup会搜索每个tag的"href"属性

```
soup.find_all(href=re.compile("elsie"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```

搜索指定名字的属性时可以使用的参数值包括: 字符串,正则表达式，列表，True

查找所有包含id属性的tag，无论id是什么值

```
soup.find_all(id=True)
```

使用多个指定名字的参数可以同时过滤tag的多个属性：

```
soup.find_all(href=re.compile("elsie"),id="link1")
```

有些tag属性在搜索不能使用，比如HTML5中的data-*属性：

![soup-26.png](https://aaron-13.github.io/images/soup-26.png)

但是可以通过find_all()方法的attrs参数定义一个字典参数来搜索包含特殊属性的tag：

```
data.find_all(attrs={"data-foo":"value"})
# [<div data-foo="value">foo!</div>]
```


## 按CSS搜索

按照CSS类名搜索tag的功能非常实用，但标识CSS类名的关键字class在Python中是保留字，实用class做参数会导致语法错误，从BeautifulSoup的4.1.1版本开始，可以通过class_参数搜索有指定CSS类名的tag:

![soup-27.png](https://aaron-13.github.io/images/soup-27.png)


class_参数同样接受不同类型的过滤器，字符串，正则表达式，方法或True：

```
soup.find_all(class_=re.compile('itl'))
# [<p class="title"><b>The Dormouse's story</b></p>]

def fun(css_class):
	return css_class is not None and len(css_class) == 6

soup.find_all(class_=fun)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

tag的class属性是多值属性，按照CSS类名搜索tag时，可以分别搜索tag中的每个CSS类名:

```
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.find_all('p',class_="strikeout")
# [<p class="body strikeout"></p>]

css_soup.find_all('p',class_='body')
# [<p class="body strikeout"></p>]

```

搜索class属性时也可以通过CSS值完全匹配：
```
css_soup.find_all('p',class_="body strikeout")
# [<p class="body strikeout"></p>]
```

完全匹配class的值时，如果CSS类名的顺序与实际不符，将搜索不到结果：
```
soup.find_all('p',attrs={"class": "body strikeout"})
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```


**string参数**

通过string参数可以搜索文档中的字符串内容，与name参数的可选值一样，string参数接受字符串，正则表达式，列表，True：

```
soup.find_all(string="Elsie")
# [u'Elsie']

soup.find_all(string=["Tillie", "Elsie", "Lacie"])
# [u'Elsie', u'Lacie', u'Tillie']

soup.find_all(string=re.compile("Dormouse"))
[u"The Dormouse's story", u"The Dormouse's story"]

def st(s):
	""Return True if this string is the only child of its parent tag""
	return(s == s.parent.string)

soup.find_all(string=st)
# [u"The Dormouse's story", u"The Dormouse's story", u'Elsie', u'Lacie', u'Tillie', u'...']

```

虽然string参数用于搜索字符串，还可以与其他混合使用来过滤tag，BeautifulSoup会找到.string方法与string参数值相符的tag

```
soup.find_all('a',string='Elsie')
# [<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>]

```


**limit参数**

find_all()方法返回全部的搜索结果，如果文档树很大那么搜索会很慢，如果不需要全部结果，可以使用limit参数限制返回结果的数量，效果和SQL中的limit关键字类似，当搜索到的结果数量达到limit的限制时，就停止搜索返回结果：

```
soup.find_all('a',limit=2)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```


**recursive参数**

调用tag的find_all()方法时，BeautifulSoup会检索当前tag的所有子孙节点，如果只想搜索tag的直接子节点，可以使用参数recursive=False

```
<html>
 <head>
  <title>
   The Dormouse's story
  </title>
 </head>
...
```

是否使用recursive参数的搜索结果

```
soup.html.find_all('title')
# [<title>The Dormouse's story</title>]

soup.html.find_all('title',recursive=False)
# []

```

<title>标签在<html>标签下，但并不是直接子节点，<head>标签才是直接子节点，在允许查询所有后代节点时BeautifulSoup能够查找到<title>，但是使用了recursive=False参数之后，只能查找直接子节点，这样就查不到<title>标签了

BeautifulSoup提供了多种DOM树搜索方法，这些方法都使用了类似的参数定义，比如这些方法：find_all():name,attrs,text,limit.但是只有find_all()和find()支持recursive参数u


**像调用find_all()一样调用tag**

BeautifulSoup对象和tag对象可以被当做一个方法来使用，这个方法的执行结果与调用这个对象的find_all()方法相同

```
soup.find_all('a')ZQ
soup("a")

soup.title.find_all(string=True)
soup.title(string=True)

```


**find()**

**find(name,attrs,recursive,string,\*\*kwargs)**

文档中只有一个的标签，使用find_all()方法来查找不合适，使用limit=1参数不如直接使用find()方法：

```
soup.find_all("title",limit=1)

soup.find("title")

# <title>The Dormouse's story</title>

```

唯一区别是find_all()方法返回的值包含一个元素的列表，而find()方法直接返回结果

find_all()方法没有找到目标是返回空列表，find()方法找不到目标，返回None

soup.head.title是tag的名字方法的简写，这个简写的原理就是多次调用当前tag的find()方法：

```
soup.head.title

soup.find("head").find("title")

# <title>The Dormouse's story</title>
```


**find_parent和find_parents()**

find_parent(name,attrs,recursive,string,\*\*kwargs)
find_parents(name,attrs,recursive,string,\*\*kwargs)

```
st = soup.find(string="Lacie")

st.find_parents("a")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

st.find_parent("p")
# <p class="story">Once upon a time there were three little sisters; and their names were
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
#  and they lived at the bottom of a well.</p>

```

**注：在较新版本中，find/find_all中的string参数被替换为了text。**


**find_next_siblings()和find_next_sibling()**

**find_previous_siblings()和find_previous_sibling()**

**find_all_next()和find_next()**

这两个方法通过.next_elements属性对当前tag之后的tag和字符串进行迭代

**find_all_previous()和find_previous()**


**CSS选择器**

BeautifulSoup支持大部分的CSS选择器，在Tag或Beautiful对象的.select()方法中传入字符串参数，即可使用CSS选择器的语法找到tag:

```
soup.select('title')
# [<title>The Dormouse's story</title>]

soup.select("p nth-of-type(3)")
#[<p class="story">...</p>]
```

通过tag标签逐层查找：

```
soup.select("body a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("html head title")
# [<title>The Dormouse's story</title>]
```

找到某个tag标签下的直接子标签：

```
soup.select("head > title")
# [<title>The Dormouse's story</title>]

soup.select("p > a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("p > a:nth-of-type(2)")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

soup.select("p > #link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("body > a")
# []

```

找到兄弟节点标签：

```
soup.select("#link1 ~ .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie"  id="link3">Tillie</a>]

soup.select('#link1 + .sister')
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

```


通过CSS的类名查找：

```
soup.select(".sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("[class~=sister]")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

通过是否存在某个属性来查找：

```
soup.select('a[href]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

通过属性的值来查找：

```
soup.select('a[href="http://example.com"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href$="titllie"]')
# [<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href*=".com/el"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```


通过语言设置来查找：

```
markup ="""
 <p lang="en">Hello</p>
 <p lang="en-us">Howdy, y'all</p>
 <p lang="en-gb">Pip-pip, old fruit</p>
 <p lang="fr">Bonjour mes amis</p>
"""
soup = BeautifulSoup(markup)
soup.select('p[lang|=en]')
# [<p lang="en">Hello</p>,
#  <p lang="en-us">Howdy, y'all</p>,
#  <p lang="en-gb">Pip-pip, old fruit</p>]

```

返回查找到的元素的第一个：

```
soup.select_one(".sister")
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```


## 修改文档树

```
soup = BeautifulSoup('<b class="boldest">Extremely blod</b>')
tag = soup.b

tag.name = "blockquote"
tag['class'] = 'verybold'
tag['id'] = 1
tag
# <blockquote class="verybold" id="1">Extremely bold</blockquote>

del tag['class']
del tag['id']
tag
# <blockquote>Extremely bold</blockquote>
```

**修改.string**

给tag的.string属性赋值，就相当于用当前的内容替代了原来的内容

```
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)

tag = soup.a
tag.string = "New link text."
tag
# <a href="http://example.com/">New link text.</a>
```

如果当前的tag包含了其他tag，那么给它的.string属性赋值会覆盖掉所有内容包含子tag


**append()**
Tag.append()方法想tag中添加内容，就好像Python的列表.append()方法：

```
soup = BeautifulSoup("<a>Foo</a>")
soup.a.append("Bar")

soup
# <html><head></head><body><a>FooBar</a></body></html>
soup.a.contents
# [u'Foo', u'Bar']
```


**NavigableString()和。new_tag()**

如果想添加一段文本内容到文档中也没问题，可以调用Python的append()方法或调用NavigableString的构造方法：

```
soup = BeautifulSoup('<b></b>')
tag = soup.b
tag.append("hello")
tag.contents
# [u'hello']
```

如果想要创建一段注释，或NavigableString的任何子类，只要调用NavigableString的构造方法：

```
from bs4 import Comment
new = soup.new_string("nice to see you",Comment)
soup.b.append(new)
soup.b
# <b>Hello there<!--Nice to see you.--></b>

soup.b.contents
# [u'Hello', u' there', u'Nice to see you.']
```

创建一个tag最好的方法是调用工厂方法BeautifulSoup.new_tag():

```
soup = BeautifulSoup("<b></b>")
tag = soup.b
new = soup.new_tag('a',href="http://www.example.com")
tag.append(new)
tag
# <b><a href="http://www.example.com"></a></b>

new.string = "hello"
# <b><a href="http://www.example.com">hello</a></b>

```

第一个参数作为tag的name是必填，其他参数可选


**insert**

Tag.insert()方法与Tag.append()方法类似，区别是不会把新元素添加到父节点.contents属性的最后，而是把元素插入到指定的位置：

```
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
tag = soup.a

tag.insert(1, "but did not endorse ")
tag
# <a href="http://example.com/">I linked to but did not endorse <i>example.com</i></a>
tag.contents
# [u'I linked to ', u'but did not endorse', <i>example.com</i>]
```


**insert_before()和insert_after()**

insert_before()方法在当前tag或文本节点前插入内容：


**clear()**

Tag.clear()方法移除当前tag的内容


**extract()**

PageElement.extract()方法将当前tag移除文档树，并作为方法结果返回：

```
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

i_tag = soup.i.extract()

a_tag
# <a href="http://example.com/">I linked to</a>

i_tag
# <i>example.com</i>

print(i_tag.parent)
None
```

这个方法实际产生了2个文档树：一个是用来解析原始文档的BeautifulSoup对象，另一个是被移除并且返回的tag。被移除并返回的tag可以继续调用extract方法：

```
my_string = i_tag.string.extract()
my_string
# u'example.com'

print(my_string.parent)
# None
i_tag
# <i></i>
```


**decompose()**

Tag.decompose()方法将当前节点移除文档树并完全销毁：

```
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

soup.i.decompose()

a_tag
# <a href="http://example.com/">I linked to</a>
```

**replace_with()**

PageElement.replace_with()方法移除文档树中的某段内容，并用新tag或文本节点替代它：

```
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

new_tag = soup.new_tag("b")
new_tag.string = "example.net"
a_tag.i.replace_with(new_tag)

a_tag
# <a href="http://example.com/">I linked to <b>example.net</b></a>
```

replace_with()方法返回替代的tag或文本几点，可以用来浏览或添加到文档树其他地方。


**wrap**

PageElement.wrap()方法可以对指定的tag元素进行包装，并返回包装后的结果

```
soup = BeautifulSoup("<p>I wish I was bold.</p>")
soup.p.string.wrap(soup.new_tag("b"))
# <b>I wish I was bold.</b>

soup.p.wrap(soup.new_tag("div"))
# <div><p><b>I wish I was bold.</b></p></div>
```


**unwrap()**

Tag.wrap()方法与wrap()方法相反，将移除tag内的所有tag标签，该方法常被用来进行标记的解包：

```
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

a_tag.i.unwrap()
a_tag
# <a href="http://example.com/">I linked to example.com</a>
```

与replace_with()方法相同，unwrap()方法返回将被移除的tag


## 输出

格式化输出
prettify()方法将BeautifulSoup的文档树格式化后以Unicode编码输出，每个XML/HTML标签独占一行

```
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
soup.prettify()
# '<html>\n <head>\n </head>\n <body>\n  <a href="http://example.com/">\n...'

print(soup.prettify())
# <html>
#  <head>
#  </head>
#  <body>
#   <a href="http://example.com/">
#    I linked to
#    <i>
#     example.com
#    </i>
#   </a>
#  </body>
# </html>
```

**压缩输出**

如果只想得到结果字符串，不重视格式，可以对BeautifulSoup对象或tag对象使用Python的unicode()或str()方法：

```
str(soup)
# '<html><head></head><body><a href="http://example.com/">I linked to <i>example.com</i></a></body></html>'

unicode(soup.a)
# u'<a href="http://example.com/">I linked to <i>example.com</i></a>'
```

str()方法返回UTF-8编码的字符串，可以指定编码的设置
还可以调用encode()方法获取字节码或调用decode()方法获得Unicode


**输出格式**

BeautifulSoup输出是会将HTML中的特殊字符转换成Unicode，比如"&lquot;"

```
soup = BeautifulSoup("&ldquo;Dammit!&rdquo; he said.")
unicode(soup)
# u'<html><head></head><body>\u201cDammit!\u201d he said.</body></html>'
```


**get_text()**

如果只想得到tag中包含的文本内容，那么可以调用get_text()方法，这个方法获取到tag中包含的所有文版内容包括子孙tag中的内容，并将结果作为Unicode字符串返回：

```
markup = '<a href="http://example.com/">\nI linked to <i>example.com</i>\n</a>'
soup = BeautifulSoup(markup)

soup.get_text()
u'\nI linked to example.com\n'
soup.i.get_text()
u'example.com'
```

还可以通过参数指定tag的文本内容的分隔符：

```
# soup.get_text("|")
u'\nI linked to |example.com|\n'
```

还可以去除获得文本内容的前后空白：

```
# soup.get_text("|", strip=True)
u'I linked to|example.com'
```

或者使用.stripped_strings生成器，获得文本列表后手动处理列表：

```
[text for text in soup.stripped_strings]
# [u'I linked to', u'example.com']
```


## 指定文档解析器

如果想要解析HTML文档，只要用文档创建BeautifulSoup对象就可以了，BeautifulSoup会自动选择一个解析器来解析，但还可以通过参数指定使用哪种解析器来解析当前文档。
BeautifulSoup第一个参数应该是要被解析的文档字符串或是文件句柄，第二个参数用来表示怎样解析文档，如果第二个参数为空，那么BeautifulSoup根据当前系统安装的库自动选择解析器，解析器的优先顺序：lxml,html5lib,python标准库。
下面两种条件下解析器优先顺序会变化：

+ 要解析的文档是什么类型：目前支持，"html","xml","html5"

+ 指定使用哪种解析器：目前支持，"lxml","html5lib","html.parser"


**解析器之间的区别**

BeautifulSoup为不同的解析器提供了相同的接口，但解析器本身有区别的，同一篇文档被不同的解析器解析后可能会生成不同结构的树型文档：

```
BeautifulSoup("<a></p>", "lxml")
# <html><body><a></a></body></html>

BeautifulSoup("<a></p>", "html5lib")
# <html><head></head><body><a><p></p></a></body></html>

BeautifulSoup("<a></p>", "html.parser")
# <a></a>
```


**编码**

任何HTML或XML文档都有自己的编码方式，比如ASCII或UTF-8，但是使用BeautifulSoup解析后，文档都被转换成了Unicode：

```
markup = "<h1>Sacr\xc3\xa9 bleu!</h1>"
soup = BeautifulSoup(markup)
soup.h1
# <h1>Sacré bleu!</h1>
soup.h1.string
# u'Sacr\xe9 bleu!'
```

BeautifulSoupp用了编码自动检测子库来识别当前文档编码并转换成Unicode编码，BeautifulSoup对象的.original_encodinig属性记录了自动识别编码的结果：

```
soup.original_encoding
'utf-8'
```

设置编码参数来减少自动检查编码出错的概率并且提高文档解析速度，在创建BeautifulSoup对象的时候设置from_encoding参数：

```
soup = BeautifulSoup(markup, from_encoding="iso-8859-8")
soup.h1
<h1>םולש</h1>
soup.original_encoding
'iso8859-8'
```

通过exclude_encoding参数，可以排除错误编码

```
soup = BeautifulSoup(markup, exclude_encodings=["ISO-8859-7"])
soup.h1
<h1>םולש</h1>
soup.original_encoding
'WINDOWS-1255'
```


**输出编码**

通过Beautifulsoup输出文档时，不管输入文档是什么编码，输出编码均为UTF-8编码，下面为输入文档时Latin-1编码：

```
markup = b'''
<html>
  <head>
    <meta content="text/html; charset=ISO-Latin-1" http-equiv="Content-type" />
  </head>
  <body>
    <p>Sacr\xe9 bleu!</p>
  </body>
</html>
'''

soup = BeautifulSoup(markup)
print(soup.prettify())
# <html>
#  <head>
#   <meta content="text/html; charset=utf-8" http-equiv="Content-type" />
#  </head>
#  <body>
#   <p>
#    Sacré bleu!
#   </p>
#  </body>
# </html>
```

注意，输出文档中的<meta>标签的编码设置已经修改成了与输出编码一致的UTF-8。如果不想用UTF-8编码输出，可以将编码方式传入prettify()方法：

```
print(soup.prettify("latin-1"))
# <html>
#  <head>
#   <meta content="text/html; charset=latin-1" http-equiv="Content-type" />
# ...
```

还可以调用BeautifulSoup对象或任意节点的encode()方法，就像Python的字符串调用encode()方法一样：

```
soup.p.encode("latin-1")
# '<p>Sacr\xe9 bleu!</p>'

soup.p.encode("utf-8")
# '<p>Sacr\xc3\xa9 bleu!</p>'
```


**UnicodeDammit
UnicodeDammit是BS内置库，主要用来猜测文档编码

```
from bs4 import UnicodeDammit
dammit = UnicodeDammit("Sacr\xc3\xa9 bleu!")
print(dammit.unicode_markup)
# Sacré bleu!
dammit.original_encoding
# 'utf-8'

```

如果Python中安装了charet或cchardet那么编码检测功能的准确率将大大提高，输入的字符越多检测结果越精确，将猜测的编码作为参数，这样将优先检测这些编码：

```
dammit = UnicodeDammit("Sacr\xe9 bleu!", ["latin-1", "iso-8859-1"])
print(dammit.unicode_markup)
# Sacré bleu!
dammit.original_encoding
# 'latin-1'
```


**只能引号**

使用Unicode时，BeautifulSOup还会只能的把引号转换成HTML或XML中的特殊字符：

```
markup = b"<p>I just \x93love\x94 Microsoft Word\x92s smart quotes</p>"

UnicodeDammit(markup, ["windows-1252"], smart_quotes_to="html").unicode_markup
# u'<p>I just &ldquo;love&rdquo; Microsoft Word&rsquo;s smart quotes</p>'

UnicodeDammit(markup, ["windows-1252"], smart_quotes_to="xml").unicode_markup
# u'<p>I just &#x201C;love&#x201D; Microsoft Word&#x2019;s smart quotes</p>'
```

也可以把引号转换为ASCII码：

```
UnicodeDammit(markup, ["windows-1252"], smart_quotes_to="ascii").unicode_markup
# u'<p>I just "love" Microsoft Word\'s smart quotes</p>'
```

默认情况下，BeautifulSoup把引号转换成Unicode


**比较对象是否相同**

两个NavigableString或Tag对象具有相同的HTML或XML结构时，BeautifulSoup就判断这两个对象相同

```
markup = "<p>I want <b>pizza</b> and more <b>pizza</b>!</p>"
soup = BeautifulSoup(markup, 'html.parser')
first_b, second_b = soup.find_all('b')
print first_b == second_b
# True

print first_b.previous_element == second_b.previous_element
# False
```

如果想判断两个对象是否严格的指向同一个对象可以通过is来判断

```
print first_b is second_b
# False
```


**复制BeautifulSoup对象**

copy.copy()方法可以复制任意tag或NavigableString对象

```
import copy
p = copy.copy(soup.p)
print p
# <p>I want <b>pizza</b> and more <b>pizza</b>!</p>
```

复制后的对象与对象是相等的，但指向不同的内存地址

```
print soup.p == p_copy
# True

print soup.p is p_copy
# False
```


## 解析部分文档

SoupStrainer类可以定义文档的某段内容，这样搜索文档时就不必先解析整篇文档，只会解析在SoupStrainer中定义过的文档。创建一个SoupStrainer对象并作为parse_only参数给BeautifulSoup的构造方法即可

**SoupStrainer**

SoupStrainer类接受与典型搜索方法相同的参数：name,attrs,recursive,string,**kwarg

```
from bs4 import SoupStrainer

only_a_tags = SoupStrainer("a")

only_tags_with_id_link2 = SoupStrainer(id="link2")

def is_short_string(string):
    return len(string) < 10

only_short_strings = SoupStrainer(string=is_short_string)

print(BeautifulSoup(html_doc, "html.parser", parse_only=only_a_tags).prettify())
# <a class="sister" href="http://example.com/elsie" id="link1">
#  Elsie
# </a>
# <a class="sister" href="http://example.com/lacie" id="link2">
#  Lacie
# </a>
# <a class="sister" href="http://example.com/tillie" id="link3">
#  Tillie
# </a>

print(BeautifulSoup(html_doc, "html.parser", parse_only=only_tags_with_id_link2).prettify())
# <a class="sister" href="http://example.com/lacie" id="link2">
#  Lacie
# </a>

print(BeautifulSoup(html_doc, "html.parser", parse_only=only_short_strings).prettify())
# Elsie
# ,
# Lacie
# and
# Tillie
# ...
#
```

还可以将SoupStrainer作为参数传入搜索文档树种提到的方法，这可能不是常用方法：

```
soup = BeautifulSoup(html_doc)
soup.find_all(only_short_strings)
# [u'\n\n', u'\n\n', u'Elsie', u',\n', u'Lacie', u' and\n', u'Tillie',
#  u'\n\n', u'...', u'\n']
```


## 常见问题

**代码诊断**

如果想知道BeautifulSoup到底怎样处理一份文档，可以将文档传入diagnose()方法

```
from bs.diagnose import diagnose
data = open('bad.html').read()
diagnose(data)

#  running on Beautiful Soup 4.2.0
# Python version 2.7.3 (default, Aug  1 2012, 05:16:07)
# I noticed that html5lib is not installed. Installing it may help.
# Found lxml version 2.3.2.0
#
# Trying to parse your data with html.parser
# Here's what html.parser did with the document:
# ...
```

**文档解析错误**

文档解析错误有两种，一种是崩溃，BeautifulSoup尝试解析一段文档结果抛出了异常，通常是HTMLParser.HTMLParseError，还有一种异常情况，是BeautifulSoup解析后的文档树看起来与原来的内容相差很多。

最常见的解析错误是HTMLParser.HTMLParseError: malformed start tag和HTMLParser.HTMLParseError:bad end tag，这都是由Python内置的解析器引起的，解决方案是安装lxml或html5lib


**版本错误**
SyntaxError: Invalid syntax (异常位置在代码行: ROOT_TAG_NAME = u'[document]' ),因为Python2版本的代码没有经过迁移就在Python3中窒息感
ImportError: No module named HTMLParser 因为在Python3中执行Python2版本的Beautiful Soup
ImportError: No module named html.parser 因为在Python2中执行Python3版本的Beautiful Soup
ImportError: No module named BeautifulSoup 因为在没有安装BeautifulSoup3库的Python环境下执行代码,或忘记了BeautifulSoup4的代码需要从 bs4 包中引入
ImportError: No module named bs4 因为当前Python环境下还没有安装BeautifulSoup4


**名称变化**

renderContents -> encode_contents
replaceWith -> replace_with
replaceWithChildren -> unwrap
findAll -> find_all
findAllNext -> find_all_next
findAllPrevious -> find_all_previous
findNext -> find_next
findNextSibling -> find_next_sibling
findNextSiblings -> find_next_siblings
findParent -> find_parent
findParents -> find_parents
findPrevious -> find_previous
findPreviousSibling -> find_previous_sibling
findPreviousSiblings -> find_previous_siblings
nextSibling -> next_sibling
previousSibling -> previous_sibling

构造方法的参数部分也有名字变化：

BeautifulSoup(parseOnlyThese=...) -> BeautifulSoup(parse_only=...)
BeautifulSoup(fromEncoding=...) -> BeautifulSoup(from_encoding=...)

为了适配Python3,修改了一个方法名:

Tag.has_key() -> Tag.has_attr()
修改了一个属性名,让它看起来更专业点:

Tag.isSelfClosing -> Tag.is_empty_element
修改了下面3个属性的名字,以免雨Python保留字冲突.这些变动不是向下兼容的,如果在BS3中使用了这些属性,那么在BS4中这些代码无法执行.

UnicodeDammit.Unicode -> UnicodeDammit.Unicode_markup``
Tag.next -> Tag.next_element
Tag.previous -> Tag.previous_element


**生成器**

childGenerator() -> children
nextGenerator() -> next_elements
nextSiblingGenerator() -> next_siblings
previousGenerator() -> previous_elements
previousSiblingGenerator() -> previous_siblings
recursiveChildGenerator() -> descendants
parentGenerator() -> parents

所以迁移到BS4版本时要替换这些代码:

for parent in tag.parentGenerator():
    ...
替换为:

for parent in tag.parents:
    ...


BS4中增加了2个新的生成器, .strings 和 stripped_strings . .strings 生成器返回NavigableString对象, .stripped_strings 方法返回去除前后空白的Python的string对象.




