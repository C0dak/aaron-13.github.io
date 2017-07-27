# HugePage

------

默认情况下，对于64bit的Linux kernel，内存页面是以4K为单位来管理的。这对于内存比较小的主机来说没什么问题，但对于较大的内存服务器，这么小的粒度明显会带来性能问题。

对于传统的Linux内存管理机制有几个问题往往会在大内存的情况下对系统的性能带来意想不到的反效果。

+ 上面提及的4K页面问题，如果OS的内存非常大，那么系统将会维护一个庞大的内存地址转换列表(PGD,PMD,PTE+offset),那每次访问需要花费更多时间

+ 再一个就是swap交换空间，如果一部分的内存空间被非预期的swap到了磁盘上，那每次对于地址空间的访问就会凭空多出了一个磁盘读写同步的过程，当然可以简单粗暴的关闭swap了事，但这样的结果往往会是以内存撑爆之后的宕机为代价

于是Linux有了hugepage的管理机制

+ Hugepage不会被交换，理论上开辟hugepage的应用都是大内存消耗型，而这类应用往往不希望被swap

+ 减少了页面表的大小，减轻了cpu的寻址压力


首先，设置系统为Hugepage保留的内存大小

```
sysctl -w vm.nr_hugepages = 16
```

查看hugepage的启用状况

```
grep Huge /proc/meminfo

```

挂载一个hugetlbfs
```
mount -t hugetlbfs none /mnt -o pagesize=2MB
```

| 概念 | 说明 |
| :--: | :--: |
| page table | page table是操作系统上的虚拟内存系统的数据结构模型，用于存储虚拟地址与物理地址的对应关系。当我们访问内存时，首先访问page table，然后Linux再通过page table的mapping来访问真是物理内存(ram + swap) |
| TLB | A Translation Lookaside Buffer(TLB),TLB是在CPU中分配的一个固定大小的buffer(or cache),用于保存page table的部分内容，使CPU更快的访问病进行地址转换 | 
| hugetlb | hugetlb是记录再TLB中的条目并指向Hugepages |
| hugetlbfs | 这是一个新的基于2.6kernel之上的内存文件系统，如同tmpfs。在TLB中通过hugetlb来指向hugepage。这些被分配的hugepage作为内存文件系统hugetlbfs提供给进程使用 |


**使用HugePages的意义**

HugePages是Linux内核的一个特性，使用hugepage可以用更大的内存页来取代传统的4K页面，使用hugepages主要带来以下好处：

+ HugePages会在系统启动时，直接分配并保留对应大小的内存区域

+ HugePages在开机之后，如果没有管理员介入，是不会释放和改变的

+ 没有swap

+ 大大提高了CPU cache中存放的page table所覆盖的内存大小，从而提高了TLB命中率

+ 减轻page table的负载

+ 提高内存的性能，降低CPU负载


**使用HugePages需要注意的地方**

+ HugePages是在分配后就会预留出来的，其大小一定要比服务器上所有实例的SGA总和要大，差一点也不行

+ 其他进程无法使用HugePages的内存，所以不要设置太大，稍微大于SGA，保证SGA可以使用到HugePages就好了

+ 在meminfo中和Hugepage相关的有四项
	HugePages_Total: 4611
	HugePages_Free: 421
	HugePages_Rsvd: 434
	Hugepagesize: 2048kB

HugePages_Total为所分配的页面数目，和Hugepagesize相乘后得到所分配的内存大小。4611*2/1024大约为9GB

HugePages_Free为从来没有被使用过的HugePages数目。及即使oracle sga已经分配这部分内存，但是如果没有实际写入，那么看到的还是Free的

HugePages_Rsvd为已经被分配预留但是还没有使用的page数目。在Oracle刚刚启动时，大部分内存应该都是Reserved并且free的，随着oracle SGA的使用received和free都会不断的降低

HugePages_Free-HugePages_Rsvd这部分是没有被使用到的内存，如果没有其他oracle instance，这部分内存也许永远都不会被使用到

+ HugePages和oracle AMM(自动内存管理)是互斥的，所以使用HugePages必须设置内存参数MEMORY_TARGET/MEMORY_MAX_TARGET为0



## 配置HugePages

**修改内核参数memlock**
修改内核参数memlock，单位是KB，如果内存是16G，memlock的大小要 稍微小于物理内存。计划lock 12GB的内存大小。参数设置为大于SGA是没有坏处的。

+ 修改limits.conf文件
增加如下内容:
	* soft memlock 12582912
	* hard memlock 12582912


+ 禁用AMM
如果使用11G及以后的版本，AMM已经默认开启，但是AMM与HugePages是不兼容的，必须先disable AMM，禁用AMM的步骤如下：

1.关闭数据库实例
	sqlplus / as sysdba
	shutdown immediate

2.创建pfile
	sqlplus / as sysdba
	create pfile='/home/oracle/pfile.ora' from spfile='+DG_ORA/orcl/spfileorcl.ora';

3.编辑pfile
	编辑pfile.ora，删除memory_max_target和memory_target参数：
	删除：
	orcl1.memory_max_target=11114905600
	orcl2.memory_max_target=11114905600
	*.memory_max_target=0
	orcl1.memory_target=11114905600
	orcl2.memory_target=11114905600
	*.memory_target=0

4.创建spfile
	sqlplus / as sysdba
	create spfile='+DG_ORA/orcl/spfileorcl.ora' from pfile='/home/oracle/pfile.ora';

5.修改系统参数kernel.shmall
	kernel.shmall是系统一次可以使用的最大共享内存大小。单位是page(4KB)。禁用AMM后，需要修改系统参数kernel.shmall，该参数设置过小的话，可能会导致数据库启动失败

	ORACLE建议将其设置为系统中所有数据库实例的SGA总和。例如SGA综合为9GB，则需要设置kernel.shmall=9*1024*1024/4=2359296

	编辑sysctl.conf文件
	kernel.shmall=2359296
	vm.nr_hugepages=4611
	执行sysctl -p


