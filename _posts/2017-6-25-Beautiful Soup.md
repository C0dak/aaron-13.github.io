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
html_doc = """
<html><head><title>The Dormouse's story</title><li>loop</li>></head>
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

![soup-15.png](https://aaron-15.github.io/images/soup-15.png)

![soup-16.png](https://aaron-16.github.io/images/soup-16.png)

