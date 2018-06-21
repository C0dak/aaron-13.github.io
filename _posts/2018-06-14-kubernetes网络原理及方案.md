# Kubernetes网络原理及方案

------

## kubernetes网络模型
在Kubernetes网络中存在两种IP(Pod IP和Service Cluster IP)，Pod IP地址是实际存在于某个网卡(可以是虚拟设备)上的，Service Cluster IP它是一个虚拟IP，是由kube-proxy使用Iptables规则重新定向到其他本地端口，再均衡到后端Pod的。

1.基本原则
每个Pod都拥有一个独立的IP地址(IPper Pod),而且假定所有的Pod都在一个可以直接连通的，扁平的网络空间中

2.设计原因
用户不需要额外考虑如何建立Pod之间的连接，也不需要考虑将容器端口映射到主机端口等问题

3.网络要求
所有的容器都可以在不用NAT的方式下同别的容器通讯；所有节点都可在不用NAT的方式下同所有容器通讯；容器的地址和别人看到的地址是同一个地址

## Docker网络基础
+ 网络名词解释
    - 网络命名空间：Linux在网络栈中引入网络命名空间，将独立的网络协议栈隔离到不同的命名空间中，彼此间无法通行；docker利用这一特性，实现不同容器间的网络隔离
    - Veth设备对： Veth设备对的引入是为了实现在不同网络命名空间的通信
    - Iptables/Netfilter: Netfilter负责在内核中执行各种挂接的规则(过滤，修改，丢弃等)，运行在内核模式中；Iptables模式是在用户模式下运行的进程，负责协助内核中Netfilter的各种规则表；通过二者的配合来实现整个Linux网络协议栈中灵活的数据包处理机制
    - 网桥： 网桥是一个二层网络设备，通过网桥可以将Linux支持的不同的端口连接起来，并实现类似交换机那样的多对多的通信
    - 路由： Linux系统包含一个完整的路由功能，当IP层在处理数据发送或转发的时候，会使用路由表来决定发往哪里

+ Docker生态技术栈
    - 服务应用层
    - 服务发现层/容器编排层
    - Docker网络层：这一层封装了底层的网络层，提供各种抽象模型，例如单主机网络模式，多主机的每个容器一个IP地址的解决方案
    - 底层的网络层：这一层包括一系列网络工具，iptables，路由，ipvlan和Linux命名空间

+ Docker网络实现
    - 单机网络模式： Bridge，Host，Container，None
    - 多机网络模式：一类是Docker在1.9版本主公引入Libnetwork项目，对跨节点网络的原生支持；一类是通过插件方式引入的第三方实现方式，如Flannel，Calico等


## Kubernetes网络基础
+ 容器间通信
同一个Pod的容器共享同一个网络命名空间，它们之间的访问可以用localhost地址+容器端口就可以访问。

+ 同一Node中的Pod间
同一Node中的Pod默认路由都是docker0的地址，由于它们关联在同一个docker0网桥上，地址网段相同，所有它们之间应当是直接通信的

+ 不同Node中Pod间通信
不同Node中Pod间通信需要满足两个条件： Pod的IP不能冲突；将Pod的IP和是所在的Node的IP关联起来，通过这个关联让Pod可以相互访问

+ Service介绍
Service是一组Pod的服务抽象，相当于一组Pod的LB，负责将请求分发给对应的Pod；Service会为这个LB提供一个IP，一般称为ClusterIP

```
{
    "kind": "Services",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            ""app": "MyApp"
        },
        "ports": [
            {
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9375
            }
        ]
    }
}
```

**ClusterIP**
- Service的ClusterIP地址是Kubernetes系统中虚拟的IP地址，由系统动态分配
- Kubernetes集群中的每个节点都运行kube-proxy；kube-proxy负责为ExternalName意外的服务实现一种虚拟IP形式；从Kbuernetes v1.2，iptables代理是默认
- 从Kubernetes v1.0，服务是一个"第三层"(TCP/UDP over IP)构造。在Kubernetes v1.1中，添加Ingress API以表示"第七层"HTTP服务

------

```
{
    "kind": "Services",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            ""app": "MyApp"
        },
        "ports": [
            {
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9375
            }
        ]
    }
}
```

**NodePort**
- 在具有集群内部IP之上，在集群的每个节点(每个节点相同端口)上的端口上公开服务。可以在任何<NodeIP>:NodePort地址上访问服务
- 将类型字段设置为NodePort，Kubernetes主机将从标志配置的范围(默认30000-32767)分配一个端口，每个节点将代理该端口到服务

------
```
{
    "kind": "Services",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            ""app": "MyApp"
        },
        "ports": [
            {
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9375
            }
        ]，
        "clusterip": "10.0.1.2",
        "loadBalancerIP": "78.12.21.24",
        "type": "LoadBalancer"
    },
    "status": {
        "loadBalancer": {
            "ingress": [
                {
                    "ip": "112.134.234.32"
                }
            ]
        }
    }
}
```

**LoadBalancer**
- 除了具有集群内部IP和在NodePort上过公开服务之外，还要求云提供商负载均衡转发到为每个节点显示为<NodeIP>:NodePort的服务
- 来自外部负载均很启的流量将被定向到后端Pods，尽管其工作原理取决于云提供商。一些云提供商允许指定loadBalancerIP。在这些情况下，将使用用户指定的loadBalancer创建负载均衡器。如果未指定loadBalancerIP字段，则会将一个临时IP分配给loadBalancer，如果指定了loadBalancerIP，但是云提供程序不支持该功能，则忽略该字段
-
------

```
{
    "kind": "Services",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            ""app": "MyApp"
        },
        "ports": [
            {
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9375
            }
        ]，
        "externalIPs": [
            "80.11.12.34"
        ]
    }
}
```

**ExternalIPs**
- 如果有外部IP路由到一个或多个集群节点，Kubernetes服务可以暴露在这些外部IP
- 以服务端口上的外部IP(作为目标IP)进入集群的流量将路由到其中一个服务端点；外部IP不由Kubernetes管理，是集群管理员的责任
- 在ServiceSpec中，可以与任何ServiceTypes一起指定外部IP


+ Kube-proxy介绍
Kube-proxy是一个简单的网络代理和负载均衡器，它的作用主要是负责Service的实现，具体来说，就是实现了内部从Pod到Service和外部的从Node向Service的访问

**实现方式**
- userspace是在用户空间，通过kube-proxy实现LB的代理服务，这是kube-proxy最初的版本，效率不高
- iptables是纯采用iptables来实现LB，是目前kube-proxy默认的方式


## Kubernetes网络开源组件
1.技术术语
+ IPAM：IP地址管理；这个IP地址管理并不是容器特有的，传统的网络比如说DHCP其实也是一种IPAM，主流的两种方法：基于CIDR的IP地址段分配或者精确为每个容器分配IP。但总之一旦形成一个容器主机集群之后，上面的容器都要给它分配一个全局唯一的IP地址
+ Overlay：在现有二层或三层网络上再构建一个独立的网络，这个网络通常会有自己独立的IP地址空间，交换或者路由的实现
+ IPSesc: 一个点对点的一个加密通信协议，一般会用到Overlay网络的数据通道里
+ vxLAN： 有VMware，Cisco，RedHat等联合提出的这么一个解决方案，这个解决方案最主要是解决VLAN支持虚拟网络数量4096过少的问题，因为在公有云上每个租户都有不同的VPC，4096明显不够用。就有了vxLAN，可以支持1600万个虚拟网络，基本上公有云是够用的
+ 网桥Bridge： 连接两个对等网络之间的网络设备，这里指的是Docker0
+ BGP: 主干网自治路由协议(边界网关协议)，互联网由很多小的自治网络构成，自治网络之间的三层路由是由BGP实现的
+ SDN，Openflow：软件定义网络里面的一个术语，如流表，控制平面或者转发平面都是Openflow里的术语

2.容器网络方案(Overlay Network)
隧道方案：通过隧道，或者说Overlay networking的方式
    - Weave，UDP广播，本机建立新的BR，通过PCAP互通
    - OpenvSwitch(OVS),基于VxLAN和GRE协议，但是性能方面损失比较严重
    - Flannel，UDP广播，VxLan
    - Racher: IPsec

路由方案
    - Calico，基于BGP协议的路由方案，支持很细致的ACL控制，对混合云亲和度较高
    - Macvlan，从逻辑和Kernel层来看隔离性和性能最优的方案，基于二层隔离，所以需要二层路由器支持，大多数云服务商不支持，所以混合云上比较难实现
    - 性能好，没有NAT，效率表较高，但是受限于路由表，另外每个容易都有一个IP，那么业务IP可能会被用光

3. CNM & CNI
+ Docker Libnetwork Container Network Model(CNM)阵营(Docker Libnetwork的优势就是原生，而且和Docker容器声明周期结合紧密)
    - Docker Swarm overlay
    - Macvlan & IP nettwork drivers
    - Calico
    - Contiv(from Cisco)

+ Container Network Interface(CNI)阵营(CNI的优势是兼容其他容器技术如rkt，及上层编排系统Kubernetes & Mesos)
    - Kubernetes
    - Weave
    - Macvlan
    - Flannel
    - Calico
    - Contiv
    - Mesos CNI

4. Flannel容器网络
Flannel之所以可以搭建Kubernetes依赖的底层网络，是因为可以实现以下两点：
- 它给每个node上的docker容器分配相互不冲突的IP地址
- 它能给这些IP之间建立一个覆盖网络，通过覆盖网络，将数据包原封不动的传递到目标容器内

**Flannel介绍**
- Flannel是CoreOS团队针对Kbuernetes设计的一个网络规划服务，简单来说，它的功能是让集群中不同节点主机创建的Docker容器具有去集群唯一的虚拟IP地址
- 在默认的docker配置中，每个节点上的Docker服务会分别夫妇则所在节点容器的IP分配。这样会导致一个问题是，不同节点上容器可能会获得相同内环IP地址。并使这些容器能够通过IP地址相互找到
- FLannel的设计目的是为集群中的所有节点重新规划IP地址的使用改规则，从而使得不同节点上的容器能够获得"同属一个内网"且"不重复"IP地址，并让属于不同节点上的容器能够直接通过内网IP通信
- Flannel实际上是一种"覆盖网络(Overlay Network)",也就是将TCP数据包装在另一种网络包里进行路由转发和通信，目前已经支持udp，vxlan，host-gw，aws-vpc，gce和alloc路由等数据转发方式，默认的节点间数据通信方式是UDP转发

**Flannel原理**
- 数据从源容器中发出后，经由所在主机的docker0虚拟网卡转发到flnnel0虚拟网卡，这个是P2P的虚拟网卡，flanneld服务坚挺在网卡的另一端，flannel通过Etcd服务维护一张节点间的路由表
- 源主机的flanneld服务将原本的数据内容UDP封装后根据自己的路由表投递给目的节点的flanneld服务，数据到达以后被解包，然后直接进入目的的节点flannel0虚拟网卡，然后被转发到目的主机的docker0虚拟网卡
- 最后就像本机容器通信，由docker0路由到达目标容器，这样整个数据包的传递就完成了

5. Calico容器网络
**Calico介绍**
- Calico是一个纯3层的数据中心网络方案，而且无缝集成像OpenStack这种Iaas云架构，能够提供可控的VM，容器，裸机之间的IP通信。Calico不使用重叠网络比如flannel和libnetwork重叠网络驱动，它是一个纯三层的方法，使用虚拟路由代替虚拟交换，每台虚拟机由通过BGP协议传播科大信息(路由)到剩余数据中心
- Calico节点组网可以直接利用数据中心的网络结构(无论L2还是L3)，不需要额外的NAT，隧道或者Overlay Network
- Calico基于iptables还提供了丰富灵活的网络Policy，保证通过各个节点上的ACLs来提供Workload的多租户隔离，安全组和其他可达性限制等功能

Felix，Calico Agent：跑在每台需要运行Wrokload的节点上，主要负责配置路由及ACLs等信息来确保Endpoint的连通状态
etcd：分布式键值存储，主要负责网络元数据一致性，确保Calico网络状态的准确性
BGP Client(BIRD): 主要负责把Felix写入Kernel的路由信息分发到Calico网络，确保Workload建的通信的有效性
BGP Route Reflector(BIRD): 大规模部署时使用，摒弃所有节点互联的mesh模式，通过一个或者多个BGP Route Reflector来完成集中式的路由分发

纯粹的三层实现
通过BGP路由
扩展性好，没有隧道技术，性能好
可用于容器，虚拟机，裸机
外界可以直接通过路由访问IP，也可以主机上做端口映射
需要数据中心内路由器做好配置适应

## 性能对比
Calico BGP方案最好，不能用BGP可以考虑Calico IPIP tunnel方案；如果是Coreos系又能开udp offload，flannel是不错的选择；Docker原生Overlay还有很多需要改进的地方

|方案|结论|优势|劣势|
|:-:|:-:|:-:|:-:|
|Calico|calico的2个方案都不错，其中ipip的方案在bigmsg size上表现更好，bgp方案比较稳定，CPU消耗没有ipip大|性能好，可控性搞，隔离性好|操作比较复杂，如对iptables有依赖|
|flannel|flannel的2个方案还行，vxlan方案在启用udp offload时，性能基本可达到无损，cpu占用率好，udp方案受限于userspace的解包，仅比weave要好一点|部署简单，性能还行|没法实现固定IP的容器漂移，没法多子网隔离，对上层设计有依赖，没有IPAM，IP地址浪费，duidocker启动方式有绑定|
|docker原生overlay|其实也是基于vxlan实现，受限于cloud上不一定会开的网卡udp offload，vxlan方案性能上限是裸机的55%左右，大体表现和flannel vxlan一致|docker原生，性能凑合|内核要求>3.16，对docker daemon有依赖要求(consul/etcd),本身驱动实现略差，对CPU利用率和带宽比同样比flannel vxlan的差，虽有api，但对network以及子网络隔离局部交叉这种需求比较麻烦|
