# Python的MRO算法

------

**MRO(Method Resolution Order): 方法解析顺序**
Python中的多重继承，会引发很多问题，如二义性，Python中一切皆引用，这使得不会像C++一样使用虚基类处理基类对象重复的问题，Python中处理这种问题的方式就是MRO

Python2.2以前的版本: 经典类(classic class)时代
经典类是一种没有继承的类，实例类型都是type类型，如果经典类被作为父类，子类调用父类的构造函数时会出错。
这时MRO的方法是DFS(深度优先搜索(子节点顺序: 从左到右))

![mro_01.png](https://aaron-13.github.io/images/mro_01.png)

MRO的DFS顺序如下:

![mro_02.png](https://aaron-13.github.io/images/mro_02.png)

**两种继承模式在DFS的优缺点:
第一种：两个互不相关的类的多继承，这种情况下DFS顺序正常，不会引起问题

第二种: 菱形继承模式，存在公共父类(D)的多继承，这种情况下DFS必定经过公共父类(D),如果这个公共父类有一些初始化属性或者方法，但是子类(C)又重写了这些属性或者方法，那么按照DFS顺序必定是会先找到D的属性或方法，那么C的属性或方法将不会被访问到，导致C只能继承无法重写(override)。这也就是新式类不使用DFS的原因，因为它们都由一个公共的祖先object


Python2.2版本: 新式类(new-style class)诞生
为了使类和内置类型更加统一，引入了新式类。新式类的每个类都继承于一个基类，可以是自定义或者其他类，默认承于object。子类可以调用父类的构造函数。

这时MRO有两种方法:
1. 如果是经典类MRO为DFS(深度优先搜索(子节点顺序: 从左到右))
2. 如果是新式类MRO为BFS(广度优先搜索(子节点顺序: 从左到右))

MRO的BFS顺序:
![mro_03.png](https://aaron-13.github.io/images/mro_03.png)

从B和B的父类开始找的顺序，称之为单调性

第一种:	正常继承模式，实现的是优先广度优先，先通过B找到C,然后再去找D。

第二种:	菱形继承模式，这种模式下面，BFS的查找顺序虽然解了DFS顺序下面的菱形问题，但是也违背了查找的单调性。

因为违背了单调性，所以BFS方法只在Python2.2出现了，后面的版本中用C3算法取代了BFS


Python2.3到Python2.7: 经典类，新式类和平发展

MRO的C3算法:
![mro_04.png](https://aaron-13.github.io/images/mro_04.png)


Python3+: 新式类
Python3开始只存在新式类，采用的MRO算法依旧是C3算法

[C3]
C3算法解决了单调性问题和只能继承无法重写的问题

L[C(B)] = C + merge(L[B],B) = C + L[B]


