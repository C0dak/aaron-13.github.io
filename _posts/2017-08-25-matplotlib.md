# Matplotlib教程

------

[原文](http://www.loria.fr/~rougier/teaching/matplotlib/)
[译文](https://liam0205.me/2014/09/11/matplotlib-tutorial-zh-cn/)


IPython以及pylab模式
IPython是Python的一个增强版本，增强方面: 命名输入输出，使用系统命令(shell commands),排错(debug)能力。使用`ipython --pylab`可以进行交互式画图。

pylad
pylab是matplotlib面向对象绘图库的一个接口。它的语法和Matlab十分相似。


### 初级绘制

第一步: 取得正弦函数和余弦函数

```

from pylab import *

// numpy的linspace方法生成一个数组(start,stop,num=50,endpoint=True,retstep=False,dtype=None) endpoint指明是否排除stop端点,retstep显示步长值
X = np.linspace(-np.pi,np.pi,256,endpoint=True)

C,S = np.sin(X),np.cos(X)
```

X是一个numpy数组，包含了从-π到π的等间隔的256个值。C和S则分别是这256个值对应的余弦和正弦函数值组成的numpy数组


使用默认配置
Matplotlib的默认配置允许用户自定义，可以调整大多数的默认配置: 图片大小和分辨率(dpi),线宽，颜色，风格，坐标轴以及网格属性，文字与字体属性等。不过，默认配置在大多数情况下已经做得足够好。

```
from pylab import *

X = np.linspace(-np.pi,np.pi,256,endpoint=True)
S,C = np.sin(X),np.cos(X)

plot(X,C)
plot(X,S)

show()
```

![pylad-01.png](https://aaron-13.github.io/images/pylad-01.png)


默认配置的具体内容

```
# 导入matplotlib的所有内容，numpy可以用np这个名字来使用
from pylab import *

# 创建一个 8 * 6点(point)的图，并设置分辨率为80
figure(figsize=(8,6),dpi=80)

# 创建一个新的1 * 1的子图，接下来的图样绘制在其中的第一块(也是唯一一块)
subplot(1,1,1)

# 绘制余弦曲线，使用蓝色的，连续的，宽度是1(像素)的线条
plot(X,C,color="blue",linewidth=1.0,linestyle="-")

# 绘制正弦去吸纳，使用红色，连续的，宽度是1(像素)的线条
plot(X,S,color="red",linewidth=1.0,linestyle="-")

# 设置横轴上下限
xlim(-4.0,4.0)

# 设置横轴记号
xticks(np.linspace(-4,4,9,endpoint=True))

# 设置纵轴上下限
ylim(-1.0,1.0)

# 设置纵轴记号
yticks(np.linspace(-1.0,1.0,5,endpoint=True))

# 以分辨率72来保存图片
savefig("1.png",dpi=72)

# 在屏幕上显示
show()
```

![pylab-02.png](https://aaron-13.github.io/images/pylab-02.png)



**改变线条的颜色和粗细**

```
figure(figsize=(10,6),dpi=100)
plot(X,C,color="blue",linewidth=2.5,linestyle="-")
plot(X,S,color="red",linewidth=2.5,linestyle="-")
...
```


**设置图片边界**

```
xlim(X.min()*1.1,X.max()*1.1)
ylim(C.min()*1.1,C.max()*1.1)
```

更好地方式:

```
xmin,xmax = X.min(),X.max()
cmin,cmax = C.min(),C.max()

dx = (xmax - xmin) * 0.2
dc = (xmax - xmin) * 0.2

xlim(xmin - dx,xmax + dx)
ylim(cmin - dc,cmax + dc)
```


**设置记号**

在讨论正弦和余弦的时候，通常希望知道函数在±π和±π/2的值

```
xticks(-np.pi,-np.pi/2,0,np.pi/2,np.pi)
yticks(-1,0,+1)
```

![pylab-03.png](https://aaron-13.github.io/images/pylab-03.png)


**设置记号的标签**

当设置记号的时候，可以同时设置记号的标签，这里使用了LaTeX

```
xticks([-np.pi,-np.pi/2,0.np.pi/2,np.pi],[r'$-\pi$',r'$-\pi/2$',r'$0$',r'$+\pi/2$',r'$+\pi$'])
yticks([-1,0,+1],[r'$-1$',r'$0$',r'$+1$'])
```
![pylab-04.png](https://aaron-13.github.io/images/pylab-04.png)


**移动脊柱**

坐标轴线和上面的记号连在一起就形成了脊柱(Spine)，记录了数据区域的范围，它们可以放在任意位置。

实际上每幅图有四条脊柱(上下左右)，为了将脊柱放在图的中间，必须将其中的两条(上，右)设置为无色，然后调整剩下的两条到合适的位置--数据空间的0点。

```
...
ax = gca()
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('nonne')
ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data',0))
ax.yaxis.set_ticks_position('left')
ax.spines['left'].set_postion(('data',0))
...
```


**添加图例**

只需要在plot函数里以[键值]的形式增加一个参数
```
plot(X,C,color='blue',linewidth=2.0,linestyle="-",label="cosine")
plot(X,S,color="red",linewidth=2.0,linestyle="-",label="sine")

// postion: right,center left,upper right,lower right,best,center,lower left,center right,upper left,center,lower center
legend(loc="upper left")
```


**一些特殊点做注释**

在2π/3的位置给两条函数曲线加上一个注释，首先，在对应的函数图像位置上画一个点，然后，向横轴 引一条垂线，以虚线做标记，最后写上标签。

```
t = 2*np.pi/3
plot([t,t],[0,np.cos(t)],color="blue",linewidth=2.5,linestyle='--')
scatter([t,],[np.cos(t),],50,color='blue')

// xy指明注释信息指向的点的位置，其余都有默认值
annotate(r'$\sin(\frac{2\pi}{3})=\frac{\sqrt{3}}{2}$',
		xy=(t,np.sin(t)),xycords='data',xytext=(+10,+30),textcoords='offset points',fontsize=16,arrowprops=dict(arrowstyle='->',connectionstyle="arc3,rad=.2"))

plot([t,t],[0,np.sin(t)],color='red',linestyle='--',linewidth=2.5)
scatter([t,],[np.sin(t),],50,color='red')
annotate(r'$\cos(\frac{2\pi}{3})=-\frac{1}{2}$',xy=(t,np.cos(t)),xycoords='data',xytext=(-90,-50),textcoords='offset points',fontsize=16,arrowprops=dict(arrowstyle="->",connectionstyle="arc3,rad=.2"))
```

![pylab-05.png](https://aaron-13.github.io/images/pylab-05.png)


**精益求精**

坐标轴上的记号标签被曲线挡住了，可以进行放大，然后添加一个白色的半透明底色，这样可以保证标签和曲线同时可见。

```
for label in ax.get_xticklabels() + ax.get_yticklabels():
	label.set_fontsize(16)
	label.set_bbox(dict(facecolor="white",edgecolor="None",alpha=0.65))
```


**图像，子图，坐标轴和记号**

Matplotlib中的图像值得是用户界面看到的整个窗口内容，在图像里面有所谓子图，子图的文职是由坐标网格确定的，而坐标轴不受此限制，可以放在图像的任意位置。当调用plot函数的时候，matplotlib调用gca()函数以及gcf()函数来获取当前的坐标轴和图像，如果无法获取图像，则会调用figure()函数来创建一个--严格来说，是用subplot(1,1,1)创建一个只有一个子图的图像。


图像
所谓图像就是GUI里以[Figure #]为标题的那些窗口。图像编号从1开始，与MATLAB的风格一致，而于Python从0开始编号的风格不同

|参数|默认值|描述|
|:--:|:--:|:--:|
|num| 1 | 图像的数量|
|figsize| figure.figsize| 图像的长和宽(英寸)|
|dpi| figure.dpi| 分辨率(点/英寸)|
|facecolor| figure.facecolor| 绘图区域的背景颜色|
| edgecolor| figure.edgecolor|绘图区域边缘的颜色|
|frameon| True| 是否绘制图像边缘|

这些默认值都可以在源文件中指明，不过除了图像数量这个参数，其余的参数都很少修改。

Matplotlib也提供了名为close的函数来关闭这个窗口。close函数的具体行为取决于提供的参数:
1.不传递参数:关闭当前窗口
2.传递窗口编号或窗口实例(instance)作为参数: 关闭指定的窗口
3.all: 关闭所有窗口

和其他对象一样，可以使用setp或者set_something这样的方法来设置图像的属性。


子图

可以用子图来将图样(plot)放在均匀的坐标网格中。用subplot函数的时候，需要指明网格的行列数量，以及图样存放的网格区域。此外，gridspec的功能强大，可以选择它来实现这个功能。


```
#!/usr/bin/python
# -*- coding: utf-8 -*-
#
from pylab import *
subplot(2,1,1)
xticks([]),yticks([])
// text中的0.5,0.5表示文本内容显示的位置
text(0.5,0.5,'subplot(2,1,2)',ha='center',va='center',size=24,alpha=.5)

subplot(2,1,2)
xticks([]),yticks([])
text(0.5,0.5,'subplot(2,1,2)',ha='center',va='center',size=32,alpha=1)
show()
```

![pylab-06.png](https://aaron-13.github.io/images/pylab-06.png)


gridspec设置图像显示位置

```
import matplotlib.gridspec as gridspec

gs = gridspec.GridSpec(3,3)
ax1 = gs.subplot(gs[0,:])
ax2 = gs.subplot(gs[1,:-1])
ax3 = gs.subplot(gs[1:,-1])
ax4 = gs.subplot(gs[-1,0])
ax5 = gs.subplot(gs[-1,-2])
```

![pylab-08.png](https://aaron-13.github.io/images/pylab-08.png)


调整gridspec的布局

```
gs1 = gridspec.GridSpec(3,3)
gs1.update(left=0.05,right=0.48,wspace=0.05)
ax1 = subplot(gs1[:-1,:])
ax2 = subplot(gs1[-1,:-1])
ax3 = subplot(gs1[-1,-1])

gs2 = gridspec.GridSpec(3,3)
gs2.update(left=0.55,right=0.98,hspace=0.05)
ax4 = subplot(gs2[:,:-1])
ax5 = subplot(gs2[:-1,-1])
ax6 = subplot(gs2[-1,-1])
```

调整gridspec布局的比例

```
gs = gridspec.GridSpec(2,2,width_ratios=[1,2],height_ratios=[4,1])
ax1 = subplot(gs[0])
ax2 = subplot(gs[1])
ax3 = subplot(gs[2])
ax4 = subplot(gs[3])
```