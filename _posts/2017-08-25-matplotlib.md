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
ylim(cmin - dx,cmax + dc)
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
