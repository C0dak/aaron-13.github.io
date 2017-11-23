# 静态方法和类方法

------

![python-static.png](https://aaron-13.github.io/images/python-static.png)


+ 如果要用实例来调用方法，那么在定义方法的时候，要把第一个参数设置为self; 实例方法需要将类实例化后调用，如果使用类直接调用实例方法，需要显示地将实例作为参数传入。

+ 如果要使用静态方法，那么要在方法前面加上@staticmethod修饰符; 静态方法指类中无序实例参与即可调用的方法，在调用过程中，无序将类实例化，直接在类之后使用.号运算符调用方法

+ 如果要使用类方法，那么要在方法前面加上@classmethod修饰符，并且在方法中至少使用一个参数，第一个参数在方法中的作用就是代表该类本身; 类方法可以通过类直接调用，但无论哪种调用方式，最左侧传入的参数一定是类本身


```
class A():
	# 实例方法
	def foo(self,x):
		print("excuting foo %s %s" %(self,x))

	# 静态方法
	@staticmethod
	def static_foo(x):
		print("excuting static_foo %s" %x)

	# 类方法
	@classmethod
	def class_foo(cls,x):
		print("excuting class_foo %s %s" %(cls,x))
```

|\|实例方法|类方法|静态方法|
|:---:|:---:|:---:|:---:|
|a=A()|a.foo(x)|a.class_fox(x)|a.static_foo(x)|
|A()|  |A.class_foo()|A.static_foo()|



