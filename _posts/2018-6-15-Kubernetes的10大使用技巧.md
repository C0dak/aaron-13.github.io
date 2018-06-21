# Kubernetes使用技巧

------

## bash针对kubectl自动补全
```
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

## 给每个namespace添加默认的内存和CPU限额
如果把应用部署到没有限额设置的集群，那么该应用可能会crash掉一个节点
为了避免这种情况，Kubernetes允许为每个namespace设置默认的限额。
```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

## kubelet清理镜像
这是kubelet默认已实现的功能。如果kubelet启动时没有设置flag，当/var/lib/docker目录到达90%的容量时，它就会自动进行垃圾回收。但针对inode阈值没有默认设置(Kubernetes 1.7之前)

Kubelet版本在1.4到1.6之间，可以给kubelet添加以下flag来设置：
```
--eviction-hart=memory.available<100Mi,nodefs.available<10%,nodefs.inodesFree<5%
```

Kubelet 1.7+，它是默认配。1.6默认不会监控inode的使用率，所以要添加那个flag来解决问题

## minikube
minikube是本地启动Kubernetes集群最容易的方式，只需要遵循指南下载所有东西
安装完所有组件后，运行如下命令
```
minikube start
```

当想在本地构建一个应用并在本地运行时，有一个技巧。当在本地构建一个docker镜像时，如果不运行其他命令，镜像将被构建在本地计算机

为了使构建的Docker镜像能够直接push到本地kubernetes集群，需要使用如下命令告知Docker机器:
```
eval $(minikube docker-env)
```

## 不要将kubectl开放给所有人
不要开放一个通用的kubectl给每个人。建议是基于namespace来隔离团队，然后使用RBAC策略来限制能且仅能访问哪个namespace

## Pod中断预算(Pod Disruption Budgets)
在Kubernetes集群中如何确保应用零宕机
PodDisruptBudget
集群会更新。节点会打上drain标签且Pod会被移除，这没法避免。所以应用针对每个deployment都设置一个PDB，保证至少有一个实例。可以使用一个简单的yaml来创建一个PDB，应用到集群里，并使用标签选择器来确定这个PDB覆盖了哪些资源

```
apiVersion: policy/v1beta1
kind: PodDisruptBudget
metadata:
  name: app-a-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app-a
```

需要注意的字段是matchLabels和minAvailable
matchLabels字段用来确定是否一个deployment可以关联到这个PDB。
minAvailable字段是kubernetes在某些场景下，比如node被打上drain标签时，进行操作的依据

## APP的liveness和readliness
Kubernetes允许定义探针，供Kubelet确认Pod和APP是否健康
Kubernetes提供了两种探针，Readliness和Liveness探针
Readliness探针用来确认容器是否可接受流量
Liveness探针用来确认容器是否健康的，或者需要被重启

这些配置可以追加到deployment的yaml，并且可以自定义超时时间，重时次数，延时时间等。

## 给所有事物打上标签
标签是Kubernetes的其中一个基石。它使得对象和对象之间保持松耦合，且允许根据标签来查询对象。甚至可以使用go client根据标签来监控事件

## 主动清理
Kubelet必须进行任何被告知的校验，同时它也进行自己的校验。
Kubernetes有一个服务无法连接，系统是不会挂掉的，因为它支持扩缩容。

## 持续学习

