# matplotlib中的一些方法

------

## Random Sampling(随机取样:numpy.random)

**Simple random data**

+ rand(d0,d1,...,dn): 随机值

+ randn(d0,d1,...,dn): 从标准正态分布中返回数据

+ randint(low[,high,size,dtype]): 随机整数，左开右闭区间

+ random_integers(low[,high,size]): 随机整数，闭区间

+ random_sample([size]): 0-1随机浮点数，左开右闭区间

+ ranf([size]): 0-1随机浮点数，左开右闭区间

+ sample([size]): 0-1随机浮点数，左开右闭区间

+ choice(a[,size,replace,p]): 从给定的1-D数组中选取随机数，p用来指定相应权重

+ bytes(length): 随机字节数


**Permutations(排序)**

+ shuffle(x): 通过移动其内容来修改队列

+ permutation(x): 返回一个随机队列


**Distributions(分布)**

+ beta(a,b[,size]): β分布

+ binomial(n,p[,size]): 二项式分布

+ chisquare(df[,size]): 卡方分布

+ dirichlet(alpha[,size]): 狄利克雷分布

+ exponential([scale,size]): 指数分布

+ f(dfnum,dfden[,size]): f分布

+ gamma(shape[,scale,size]): 伽马分布

+ geometric(p[,size]): 几何分布

+ gumbel([loc,scale,size]): gumbel分布

+ hypergeometric(ngood,nbad,nsample[,size]): 超几何分布

+ laplace([loc,scale,size]): 双指数分布

+ logistic([loc,scale,size]): logsitic分布

+ lognormal([mean,sigma,size]): 对数正态分布

+ logseries(p[,size]): 对数级数分布

+ nultinomial(n,pvals[,size]): 多项分布

+ mulitvariate_normal(mean,cov[,size]): 多元正态分布

+ negative_binomial(n,p[,size]): 负二项分布

+ noncentral_chisquare(df,nonc[,size]): 非中心卡方分布

+ noncentral_f(dfnum,dfden,nonc[,size]): 非中心F分布

+ normal([loc,scale,size]): 正态(高斯)分布，loc表示均值，scale表示方差

+ pareto(a[,size]): 帕累托(lomax)分布

+ poisson([lam,size]): 泊松分布

+ power(a[,size]): power分布，正指数a到1

+ rayleigh([scale,size]): rayleigh分布

+ standard_cauchy([size]): 标准柯西分布

+ standard_exponential([size]): 标准的指数分布

+ standard_gamma(shape[,size]): 标准伽马分布

+ standard_normal([size]): 标准正态分布(mean=0,stdev=1)

+ standard_t(df[,size]): 从标准的学生t分布中取样，返回自由度为df的样本

+ triangular(left,mode,right[,size]): 三角分布

+ uniform([low,high,size): 均匀分布

+ vonmises(mu,kappa[,size]): von mises分布

+ wald(mean,scale[,size]): 瓦尔德(逆高斯)分布

+ weibull(a[,size]): weibull分布

+ zipf(a[,size]): 齐普夫分布


**Random generator**

+ Randomstate 梅森伪随机数生成器的容器

+ seed([seed]): 种子生成器，如果设置了seed，每次生成的随意数相同

+ get_state(): 返回一个表示生成器内容状态的元组

+ set_state(): 从一个元组中设置生成器内部的状态