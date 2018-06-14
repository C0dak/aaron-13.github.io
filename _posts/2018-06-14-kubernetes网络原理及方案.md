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

