k8s经典   没总结到里面 

[📎云原生训练营 2022 年最新 Kubernetes 常见面试题汇总-V2(1).pdf](https://www.yuque.com/attachments/yuque/0/2024/pdf/35538885/1713683086861-7b4140da-d082-4567-b615-26410816d64e.pdf)

# ETCD

## 简述 ETCD 及其特点？ 

etcd 是 CoreOS 团队发起的开源项目，是一个管理配置信息和服务发现（service discovery）的项目， 

它的目标是构建一个高可用的分布式键值（key-value）数据库，基于 Go 语言实现。 

特点： 

简单：支持 REST 风格的 HTTP+JSON API 

安全：支持 HTTPS 方式的访问 

快速：支持并发 1k/s 的写操作 

可靠：支持分布式结构，基于 Raft 的一致性算法，Raft 是一套通过选举主节点来实现分布式系统 

一致性的算法。 

## etcd集群节点可以设置为偶数个吗，为什么要设置为奇数个呢？

不能，也不建议这么设置。

etcd采用了Raft一致性算法来确保数据的一致性和高可用性。根据Raft算法的要求，为了确保算法的正确性和容错性，集群一般包含2n+1个节点，所以进行Leader选举和数据复制时，节点数必须是奇数个。

奇数个节点与配对的偶数个节点（如3个节点和4个节点）相比，容错能力相同，但可以少一个节点；其次，偶数个节点的集群在选举过程中由于等额选票的存在，有较大概率触发下一轮选举，从而增加了不可用的风险。因此，综合考虑性能和容错能力，etcd官方文档推荐的etcd集群大小是3, 5, 7。同时需要注意的是，虽然增加节点可以提高读的吞吐和提高集群的可用性，但节点数越多可能会导致写操作的吞吐降低。

etcd官方推荐3、5、7个节点，虽然raft算法也是半数以上投票才能有 leader，但奇数只是推荐，其实偶数也是可以的。如 2、4、8个节点。下面分情况说明：

```
1 个节点：就是单实例，没有集群概念，不做讨论
2 个节点：是集群，但没人会这么配，这里说点废话：双节点的etcd能启动，启动时也能有主，可以正常提供服务，但是一台挂掉之后，就选不出主了，因为他只能拿到1票，剩下的那台也无法提供服务，也就是双节点无容错能力，不要使用。
3 节点：标准的3 节点etcd 集群只能容忍1台机器宕机，挂掉 1 台此时等于2个节点的情况，如果再挂 1 台，就和 2节点的情形一致了，一直选，一直增加任期，但就是选不出来，服务也就不可用了
4 节点：最大容忍1台服务器宕机
5 节点：最大容忍2台服务器宕机
6 节点：最大容忍2台服务器宕机
7和8个节点，最大容忍3台服务器宕机
以此类推，9和10个节点，最大容忍4台服务器宕机

```

总结以上可以得出结论：偶数节点虽然多了一台机器，但是容错能力是一样的，也就是说，虽然可以设置偶数节点，但没增加什么容错能力，还浪费了一台机器。同时etcd 是通过复制数据给所有节点来达到一致性，因此偶数集群多出一台机器既增加不了性能，反而还会拉低写入速度。

## etcd节点是越多越好吗？

不是，etcd 集群是一个 Raft Group，没有 shared。所以它的极限有两部分，一是单机的容量限制，内存和磁盘；二是网络开销，每次 Raft 操作需要所有节点参与，每一次写操作需要集群中大多数节点将日志落盘成功后，Leader 节点才能修改内部状态机，并将结果返回给客户端。因此节点越多性能越低，并且出错的概率会直线上升，并且是呈现线性的性能下降，所以扩展很多 etcd 节点是没有意义的，其次，如果etcd集群超过7个达到十几个几十个，那么，对运维来说也是一个不小的压力了，并且集群的配置什么的也会更加的复杂，而不是简单易用了。因此，etcd集群的数量一般是 3、5、7， 3 个是最低标准，7个已经是最高了.

## 简述 ETCD 适应的场景？ 

etcd 基于其优秀的特点，可广泛的应用于以下场景： 

服务发现 (Service Discovery) ：服务发现主要解决在同一个分布式集群中的进程或服务，要如何才 

能找到对方并建立连接。本质上来说，服务发现就是想要了解集群中是否有进程在监听 udp 或 tcp 端 

口，并且通过名字就可以查找和连接。

 消息发布与订阅 ：在分布式系统中，最适用的一种组件间通信方 

式就是消息发布与订阅。即构建一个配置共享中心，数据提供者在这个配置中心发布消息，而消息使用 

者则订阅他们关心的主题，一旦主题有消息发布，就会实时通知订阅者。通过这种方式可以做到分布式 

系统配置的集中式管理与动态更新。应用中用到的一些配置信息放到 etcd 上进行集中管理。 

负载均衡 ： 在分布式系统中，为了保证服务的高可用以及数据的一致性，通常都会把数据和服务部署多份，以此达 

到对等服务，即使其中的某一个服务失效了，也不影响使用。etcd 本身分布式架构存储的信息访问支持 

负载均衡。etcd 集群化以后，每个 etcd 的核心节点都可以处理用户的请求。所以，把数据量小但是访 

问频繁的消息数据直接存储到 etcd 中也可以实现负载均衡的效果。

 分布式通知与协调 ：与消息发布和订 阅类似，都用到了 etcd 中的 Watcher 机制，通过注册与异步通知机制，实现分布式环境下不同系统之 间的通知与协调，从而对数据变更做到实时处理。 

分布式锁 ：因为 etcd 使用 Raft 算法保持了数据的强 

一致性，某次操作存储到集群中的值必然是全局一致的，所以很容易实现分布式锁。锁服务有两种使用 

方式，一是保持独占，二是控制时序。集群监控与 Leader 竞选：通过 etcd 来进行监控实现起来非常简 

单并且实时性强。

# Kubernetes 相关概念？ 

master ：k8s 集群的管理节点，负责管理集群，提供集群的资源数据访问入口。拥有 Etcd 存储服 

务（可选），运行 Api Server 进程，Controller Manager 服务进程及 Scheduler 服务进程。 

node （worker）：Node（worker）是 Kubernetes 集群架构中运行 Pod 的服务节点，是 

Kubernetes 集群操作的单元，用来承载被分配 Pod 的运行，是 Pod 运行的宿主机。运行 docker 

eninge 服务，守护进程 kunelet 及负载均衡器 kube-proxy。

pod ：运行于 Node 节点上，若干相关容器的组合。Pod 内包含的容器运行在同一宿主机上，使用 

相同的网络命名空间、IP 地址和端口，能够通过 localhost 进行通信。Pod 是 Kurbernetes 进行创 

建、调度和管理的最小单位，它提供了比容器更高层次的抽象，使得部署和管理更加灵活。一个 

Pod 可以包含一个容器或者多个相关容器。 

label ：Kubernetes 中的 Label 实质是一系列的 Key/Value 键值对，其中 key 与 value 可自定 

义。Label 可以附加到各种资源对象上，如 Node、Pod、Service、RC 等。一个资源对象可以定义 

任意数量的 Label，同一个 -Label 也可以被添加到任意数量的资源对象上去。Kubernetes 通过 

Label Selector（标签选择器）查询和筛选资源对象。 

Replication Controller ：Replication Controller 用来管理 Pod 的副本，保证集群中存在指定 

数量的 Pod 副本。集群中副本的数量大于指定数量，则会停止指定数量之外的多余容器数量。反 

之，则会启动少于指定数量个数的容器，保证数量不变。Replication Controller 是实现弹性伸 

缩、动态扩容和滚动升级的核心。 

Deployment ：Deployment 在内部使用了 RS 来实现目的，Deployment 相当于 RC 的一次升级， 

其最大的特色为可以随时获知当前 Pod 的部署进度。 

HPA （Horizontal Pod Autoscaler）：Pod 的横向自动扩容，也是 Kubernetes 的一种资源，通过 

追踪分析 RC 控制的所有 Pod 目标的负载变化情况，来确定是否需要针对性的调整 Pod 副本数 

量。 

Service ：Service 定义了 Pod 的逻辑集合和访问该集合的策略，是真实服务的抽象。Service 提 

供了一个统一的服务访问入口以及服务代理和发现机制，关联多个相同 Label 的 Pod，用户不需要 

了解后台 Pod 是如何运行。 

Volume ：Volume 是 Pod 中能够被多个容器访问的共享目录，Kubernetes 中的 Volume 是定义 

在 Pod 上，可以被一个或多个 Pod 中的容器挂载到某个目录下。 

Namespace ：Namespace 用于实现多租户的资源隔离，可将集群内部的资源对象分配到不同的 

Namespace 中，形成逻辑上的不同项目、小组或用户组，便于不同的 Namespace 在共享使用整 

个集群的资源的同时还能被分别管理。

# 简述 Kubernetes 集群相关组件？ 

Kubernetes Master 控制组件，调度管理整个系统（集群），包含如下组件: 

Kubernetes API Server ：作为 Kubernetes 系统的入口，其封装了核心对象的增删改查操作， 

以 RESTful API 接口方式提供给外部客户和内部组件调用，集群内各个功能模块之间数据交互和通 

信的中心枢纽。 

Kubernetes Scheduler ：为新建立的 Pod 进行节点(node)选择(即分配机器)，负责集群的资源调 

度。 

Kubernetes Controller ：负责执行各种控制器，目前已经提供了很多控制器来保证 

Kubernetes 的正常运行。 

Replication Controller ：管理维护 Replication Controller，关联 Replication Controller 和 - 

Pod，保证 Replication Controller 定义的副本数量与实际运行 Pod 数量一致。 

Node Controller ：管理维护 Node，定期检查 Node 的健康状态，标识出(失效|未失效)的 

Node 节点。 

Namespace Controller ：管理维护 Namespace，定期清理无效的 Namespace，包括 

Namesapce 下的 API 对象，比如 Pod、Service 等。 

Service Controller ：管理维护 Service，提供负载以及服务代理。 

EndPoints Controller ：管理维护 Endpoints，关联 Service 和 Pod，创建 Endpoints 为 

Service 的后端，当 Pod 发生变化时，实时更新 Endpoints。 

Service Account Controller ：管理维护 Service Account，为每个 Namespace 创建默认的 

Service Account，同时为 Service Account 创建 Service Account Secret。 

Persistent Volume Controller ：管理维护 Persistent Volume 和 Persistent Volume Claim， 

为新的 Persistent Volume Claim 分配 Persistent Volume 进行绑定，为释放的 Persistent 

Volume 执行清理回收。Daemon Set Controller ：管理维护 Daemon Set，负责创建 Daemon Pod，保证指定的 Node 

上正常的运行 Daemon Pod。 

Deployment Controller ：管理维护 Deployment，关联 Deployment 和 Replication 

Controller，保证运行指定数量的 Pod。当 Deployment 更新时，控制实现 Replication 

Controller 和 Pod 的更新。 

Job Controller ：管理维护 Job，为 Jod 创建一次性任务 Pod，保证完成 Job 指定完成的任务数 

目 

Pod Autoscaler Controller ：实现 Pod 的自动伸缩，定时获取监控数据，进行策略匹配，当 

满足条件时执行 Pod 的伸缩动作。

# kubelet的功能、作用是什么？（重点，经常会问）

答：kubelet部署在每个node节点上的，它主要有4个功能：

1、节点管理。kubelet启动时会向api-server进行注册，然后会定时的向api-server汇报本节点信息状态，资源使用状态等，这样master就能够知道node节点的资源剩余，节点是否失联等等相关的信息了。master知道了整个集群所有节点的资源情况，这对于 pod 的调度和正常运行至关重要。

2、pod管理。kubelet负责维护node节点上pod的生命周期，当kubelet监听到master的下发到自己节点的任务时，比如要创建、更新、删除一个pod，kubelet 就会通过CRI（容器运行时接口）插件来调用不同的容器运行时来创建、更新、删除容器；常见的容器运行时有docker、containerd、rkt等等这些容器运行时，我们最熟悉的就是docker了，但在新版本的k8s已经弃用docker了，k8s1.24版本中已经使用containerd作为容器运行时了。

3、容器健康检查。pod中可以定义启动探针、存活探针、就绪探针等3种，我们最常用的就是存活探针、就绪探针，kubelet 会定期调用容器中的探针来检测容器是否存活，是否就绪，如果是存活探针，则会根据探测结果对检查失败的容器进行相应的重启策略；

4、Metrics Server资源监控。在node节点上部署Metrics Server用于监控node节点、pod的CPU、内存、文件系统、网络使用等资源使用情况，而kubelet则通过Metrics Server获取所在节点及容器的上的数据。

# kube-api-server的端口是多少？各个pod是如何访问kube-api-server的？

kube-api-server的端口是8080和6443，前者是http的端口，后者是https的端口，（注意：有些8080是k8s低版本的才有的端口，高版本中不开放此端口了）以我本机使用kubeadm安装的k8s为例：

在命名空间的kube-system命名空间里，有一个名称为kube-api-master的pod，这个pod就是运行着kube-api-server进程，它绑定了master主机的ip地址和6443端口，但是在default命名空间下，存在一个叫kubernetes的服务，该服务对外暴露端口为443，目标端口6443，这个服务的ip地址是clusterip地址池里面的第一个地址，同时这个服务的yaml定义里面并没有指定标签选择器，也就是说这个kubernetes服务所对应的endpoint是手动创建的，该endpoint也是名称叫做kubernetes，该endpoint的yaml定义里面代理到master节点的6443端口，也就是kube-api-server的IP和端口。这样一来，其他pod访问kube-api-server的整个流程就是：pod创建后嵌入了环境变量，pod获取到了kubernetes这个服务的ip和443端口，请求到kubernetes这个服务其实就是转发到了master节点上的6443端口的kube-api-server这个pod里面。

# 

# k8s中命名空间的作用是什么？

namespace是kubernetes系统中的一种非常重要的资源，namespace的主要作用是用来实现多套环境的资源隔离，或者说是多租户的资源隔离。

k8s通过将集群内部的资源分配到不同的namespace中，可以形成逻辑上的隔离，以方便不同的资源进行隔离使用和管理。不同的命名空间可以存在同名的资源，命名空间为资源提供了一个作用域。

可以通过k8s的授权机制，将不同的namespace交给不同的租户进行管理，这样就实现了多租户的资源隔离，还可以结合k8s的资源配额机制，限定不同的租户能占用的资源，例如CPU使用量、内存使用量等等来实现租户可用资源的管理。

# k8s提供了大量的REST接口，其中有一个是Kubernetes Proxy API接口，简述一下这个Proxy接口的作用，已经怎么使用。

好的。kubernetes proxy api接口，从名称中可以得知，proxy是代理的意思，其作用就是代理rest请求；Kubernets API server 将接收到的rest请求转发到某个node上的kubelet守护进程的rest接口，由该kubelet进程负责响应。我们可以使用这种Proxy接口来直接访问某个pod，这对于逐一排查pod异常问题很有帮助。

下面是一些简单的例子：

```
http://<kube-api-server>:<api-sever-port>/api/v1/nodes/node名称/proxy/pods  	#查看指定node的所有pod信息
http://<kube-api-server>:<api-sever-port>/api/v1/nodes/node名称/proxy/stats  	#查看指定node的物理资源统计信息
http://<kube-api-server>:<api-sever-port>/api/v1/nodes/node名称/proxy/spec  	#查看指定node的概要信息

http://<kube-api-server>:<api-sever-port>/api/v1/namespace/命名名称/pods/pod名称/pod服务的url/  	#访问指定pod的程序页面
http://<kube-api-server>:<api-sever-port>/api/v1/namespace/命名名称/servers/svc名称/url/  	#访问指定server的url程序页面

```

#    pod大合集

## pod的原理是什么？

在微服务的概念里，一般的，一个容器会被设计为运行一个进程，除非进程本身产生子进程，这样，由于不能将多个进程聚集在同一个单独的容器中，所以需要一种更高级的结构将容器绑定在一起，并将它们作为一个单元进行管理，这就是k8s中pod的背后原理。                    

## pod有什么特点？

```
1、每个pod就像一个独立的逻辑机器，k8s会为每个pod分配一个集群内部唯一的IP地址，所以每个pod都拥有自己的IP地址、主机名、进程等；
2、一个pod可以包含1个或多个容器，1个容器一般被设计成只运行1个进程，1个pod只可能运行在单个节点上，即不可能1个pod跨节点运行，pod的生命
周期是短暂，也就是说pod可能随时被消亡（如节点异常，pod异常等情况）；
2、每一个pod都有一个特殊的被称为"根容器"的pause容器，也称info容器，pause容器对应的镜像属于k8s平台的一部分，除了pause容器，每个pod还
包含一个或多个跑业务相关组件的应用容器；
3、一个pod中的容器共享network命名空间；
4、一个pod里的多个容器共享pod IP，这就意味着1个pod里面的多个容器的进程所占用的端口不能相同，否则在这个pod里面就会产生端口冲突；既然每
个pod都有自己的IP和端口空间，那么对不同的两个pod来说就不可能存在端口冲突；
5、应该将应用程序组织到多个pod中，而每个pod只包含紧密相关的组件或进程；
6、pod是k8s中扩容、缩容的基本单位，也就是说k8s中扩容缩容是针对pod而言而非容器。

```

## pause容器作用是什么？

每个pod里运行着一个特殊的被称之为pause的容器，也称根容器，而其他容器则称为业务容器；创建pause容器主要是为了为业务容器提供 Linux命名空间，共享基础：包括 pid、icp、net 等，以及启动 init 进程，并收割僵尸进程；这些业务容器共享pause容器的网络命名空间和volume挂载卷，当pod被创建时，pod首先会创建pause容器，从而把其他业务容器加入pause容器，从而让所有业务容器都在同一个命名空间中，这样可以就可以实现网络共享。pod还可以共享存储，在pod级别引入数据卷volume，业务容器都可以挂载这个数据卷从而实现持久化存储。

## pod的重启策略有哪些？（经常问）

  pod的重启策略（RestartPolicy）决定了当容器异常退出或健康检查失败时，kubelet将如何响应。（注意是kubelet重启容器，因为是kubelet负责容器的健康检测）

需要注意的是，虽然名为pod的重启策略（更规范的说法应该是pod中容器重启策略），但实际上是作用于pod内的所有容器。所有容器都将遵守这个策略，而不是单独的某个容器。

可以通过pod.spec.restartPolicy字段配置重启容器的策略，重启策略如下3种配置：

```
Always: 当容器终止退出后，总是重启容器，默认策略就是Always。
OnFailure: 当容器异常退出，退出状态码非0时，才重启容器。
Never: 当容器终止退出，不管退出状态码是什么，从不重启容器。

```

## pod的镜像拉取策略有哪几种？（经常问）

pod镜像拉取策略可以通过imagePullPolicy字段配置镜像拉取策略，主要有3种镜像拉取策略，如下：

```
IfNotPresent: 默认值，镜像在node节点宿主机上不存在时才拉取。
Always: 总是重新拉取，即每次创建pod都会重新从镜像仓库拉取一次镜像。
Never: 永远不会主动拉取镜像，仅使用本地镜像，需要你手动拉取镜像到node节点，如果本地节点不存在镜像则pod启动失败。
```

## pod的存活探针有哪几种？（必须记住3种探测方式，重点，经常问）

kubernetes可以通过存活探针检查容器是否还在运行，可以为pod中的每个容器单独定义存活探针，kubelet将定期执行探针，如果探测失败，将杀死容器，并根据restartPolicy策略来决定是否重启容器，kubernetes提供了3种探测容器的存活探针，如下：

```
httpGet：通过容器的IP、端口、路径发送http 请求，返回200-400范围内的状态码表示成功。
exec：在容器内执行shell命令，根据命令退出状态码是否为0进行判断，0表示健康，非0表示不健康。
TCPSocket：与容器的IP、端口建立TCP Socket链接，能建立则说明探测成功，不能建立则说明探测失败。
```

# 

## 简单讲一下 pod创建过程（经常问，必须牢记）

```
情况一、如果面试官问的是使用kubectl run命令创建的pod，可以这样说：
#注意：kubectl run 在旧版本中创建的是deployment，但在新的版本中创建的是pod则其创建过程不涉及deployment
如果是单独的创建一个pod，则其创建过程是这样的：
1、首先，用户通过kubectl或其他api客户端工具提交需要创建的pod信息给api-server；
2、api-server验证客户端的用户权限信息，验证通过开始处理创建请求生成pod对象信息，并将信息存入etcd，然后返回确认信息给客户端；
3、api-server开始反馈etcd中pod对象的变化，其他组件使用watch机制跟踪api-server上的变动；
4、scheduler发现有新的pod对象要创建，开始调用内部算法机制为pod分配最佳的主机，并将结果信息更新至api-server；
5、node节点上的kubelet通过watch机制跟踪api-server发现有pod调度到本节点，尝试调用docker启动容器，并将结果反馈api-server；
6、api-server将收到的pod状态信息存入etcd中。
至此，整个pod创建完毕。

情况二、如果面试官说的是使用deployment来创建pod，则可以这样回答：
1、首先，用户使用kubectl create命令或者kubectl apply命令提交了要创建一个deployment资源请求；
2、api-server收到创建资源的请求后，会对客户端操作进行身份认证，在客户端的~/.kube文件夹下，已经设置好了相关的用户认证信息，这样api-server会知道是哪个用户请求，并对此用户进行鉴权，当api-server确定客户端的请求合法后，就会接受本次操作，并把相关的信息保存到etcd中，然后返回确认信息给客户端。（仅返回创建的信息并不是返回是否成功创建的结果）
3、api-server开始反馈etcd中过程创建的对象的变化，其他组件使用watch机制跟踪api-server上的变动。
4、controller-manager组件会监听api-server的信息，controller-manager是有多个类型的，比如Deployment Controller, 它的作用就是负责监听Deployment，此时Deployment Controller发现有新的deployment要创建，那么它就会去创建一个ReplicaSet，一个ReplicaSet的产生，又被另一个叫做ReplicaSet Controller监听到了，紧接着它就会去分析ReplicaSet的语义，它了解到是要依照ReplicaSet的template去创建Pod, 它一看这个Pod并不存在，那么就新建此Pod，当Pod刚被创建时，它的nodeName属性值为空，代表着此Pod未被调度。
5、接着调度器Scheduler组件开始介入工作，Scheduler也是通过watch机制跟踪api-server上的变动，发现有未调度的Pod，则根据内部算法、节点资源情况，pod定义的亲和性反亲和性等等，调度器会综合的选出一批候选节点，在候选节点中选择一个最优的节点，然后将pod绑定到该节点，将信息反馈给api-server。
6、kubelet组件布署于Node之上，它也是通过watch机制跟踪api-server上的变动，监听到有一个Pod应该要被调度到自身所在Node上来，kubelet首先判断本地是否在此Pod，如果不存在，则会进入创建Pod流程，创建Pod有分为几种情况，第一种是容器不需要挂载外部存储，则相当于直接docker run把容器启动，但不会直接挂载docker网络，而是通过CNI调用网络插件配置容器网络，如果需要挂载外部存储，则还要调用CSI来挂载存储。kubelet创建完pod，将信息反馈给api-server，api-servier将pod信息写入etcd。
7、Pod建立成功后，ReplicaSet Controller会对其持续进行关注，如果Pod因意外或被我们手动退出，ReplicaSet Controller会知道，并创建新的Pod，以保持replicas数量期望值。

以上即是pod的调度过程。

```

  

## 简单描述一下pod的终止过程（记住，经常问）

```
1、用户向api-server发送删除pod对象的命令；
2、api-server中的pod对象信息会随着时间的推移而更新，在宽限期内（默认30s），pod被视为dead；
3、将pod标记为terminating状态；
4、kubectl通过watch机制监听api-server，监控到pod对象为terminating状态了就会启动pod关闭过程；
5、endpoint控制器监控到pod对象的关闭行为时将其从所有匹配到此endpoint的server资源endpoint列表中删除；
6、如果当前pod对象定义了preStop钩子处理器，则在其被标记为terminating后会以同步的方式启动执行；
7、pod对象中的容器进程收到停止信息；
8、宽限期结束后，若pod中还存在运行的进程，那么pod对象会收到立即终止的信息；
9、kubelet请求api-server将此pod资源的宽限期设置为0从而完成删除操作，此时pod对用户已不可见。

```

  

## pod的生命周期有哪几种？（记住，经常问）

```
Pending（挂起）：API server已经创建pod，但是该pod还有一个或多个容器的镜像没有创建，包括正在下载镜像的过程；
Running（运行中）：Pod内所有的容器已经创建，且至少有一个容器处于运行状态、正在启动括正在重启状态；
Succeed（成功）：Pod内所有容器均已退出，且不会再重启；
Failed（失败）：Pod内所有容器均已退出，且至少有一个容器为退出失败状态
Unknown（未知）：某于某种原因api-server无法获取该pod的状态，可能由于网络通行问题导致；

```

## pod状态一般有哪些？

```
ContainerCreating（容器正在创建）：容器正在创建中
Pending（挂起）：API server已经创建pod，但是该pod还有一个或多个容器的镜像没有创建，包括正在下载镜像的过程；
Running（运行中）：Pod内所有的容器已经创建，且至少有一个容器处于运行状态、正在启动括正在重启状态；
MatchNodeSelector （匹配节点选择器）：Pod正在等待被调度到匹配其nodeSelector的节点上，当一个Pod定义有节点选择器但没有任何节点存在指定的标签时，Pod将处于“MatchNodeSelector”状态。
ErrImagePull（镜像拉取异常）: 这个错误表示Kubernetes无法从指定的镜像仓库拉取镜像。可能的原因有很多，比如网络问题、镜像名称或标签错误、或者没有权限访问这个镜像仓库等。
ImagePullBackOff（镜像拉取异常）: 这个错误表示Kubernetes尝试拉取镜像，但是失败了，然后它回滚了之前的操作。这通常是因为镜像仓库的问题，比如网络问题、镜像不存在、或者没有权限访问这个镜像仓库等。
Error（pod异常）：可能是容器运行时异常
CrashLoopBackOff（崩溃重启） ：Pod正在经历一个无限循环的崩溃和重启过程。
Succeed（成功）：Pod内所有容器均已退出，且不会再重启；
Failed（失败）：Pod内所有容器均已退出，且至少有一个容器为退出失败状态
Unknown（未知）：某于某种原因api-server无法获取该pod的状态，可能由于网络通行问题导致；

```

## pod一直处于pending状态一般有哪些情况，怎么排查？（重点，持续更新）

（这个问题被问到的概率非常大）

答：一个pod一开始创建的时候，它本身就是会处于pending状态，这时可能是正在拉取镜像，正在创建容器的过程。

如果等了一会发现pod还一直处于pending状态，那么我们可以使用kubectl describe命令查看一下pod的Events详细信息。一般可能会有这么几种情况导致pod一直处于pending状态：

```
1、调度器调度失败。Scheduer调度器无法为pod分配一个合适的node节点。而这又会有很多种情况，比如，node节点处在cpu、内存压力，导致无节点可
调度；pod定义了资源请求，没有node节点满足资源请求；node节点上有污点而pod没有定义容忍；pod中定义了亲和性或反亲和性而没有节点满足这些亲
和性或反亲和性；以上是调度器调度失败的几种情况。
2、pvc、pv无法动态创建。如果因为pvc或pv无法动态创建，那么pod也会一直处于pending状态，比如要使用StatefulSet创建redis集群，因为粗心大
意，定义的storageClassName名称写错了，那么会造成无法创建pvc，这种情况pod也会一直处于pending状态，或者，即
使pvc是正常创建了，但是由于某些异常原因导致动态供应存储无法正常创建pv，那么这种情况pod也会一直处于pending状态。

```

### 

## pod的初始化容器是干什么的？

init container，初始化容器用于在启动应用容器之前完成应用容器所需要的前置条件，初始化容器本质上和应用容器是一样的，但是初始化容器是仅运行一次就结束的任务，初始化容器具有两大特征：

```
1、初始化容器必须运行完成直至成功结束，若某初始化容器运行失败，那么kubernetes需要重启它直到成功完成；
2、初始化容器必须按照定义的顺序执行，当且仅当前一个初始化容器成功之后，后面的一个初始化容器才能运行；

举个例子，我们最常见的es容器里面就有一个初始化容器，这个初始化容器的执行命令就是配置内核参数，因为es对某些内核参数要求设置比较大，所以
直接通过初始化容器修改了内核参数。（容器与宿主机共享内核，所以修改的就是宿主内核）

```

## pod的资源请求、限制如何定义？

pod的资源请求、资源限制可以直接在pod中定义，主要包括两块内容，limits，限制pod中容器能使用的最大cpu和内存，requests，pod中容器启动时申请的cpu和内存。

```
 resources:						#资源配额
      limits:					#限制最大资源，上限
        cpu: 2					#CPU限制，单位是code数
        memory: 2G				#内存最大限制
      requests:					#请求资源（最小，下限）
        cpu: 1					#CPU请求，单位是code数
        memory: 500G			#内存最小请求

```

## pod的定义中有个command和args参数，这两个参数不会和docker镜像的entrypointc冲突吗？（1*********）

在pod中定义的command参数用于指定容器的启动命令列表，如果不指定，则默认使用Dockerfile打包时的启动命令，args参数用于容器的启动命令需要的参数列表；

特别说明：

kubernetes中的command、args其实是实现覆盖dockerfile中的ENTRYPOINT的功能的。当：

```
1、如果command和args均没有写，那么使用Dockerfile的配置；
2、如果command写了但args没写，那么Dockerfile默认的配置会被忽略，执行pod容器指定的command；
3、如果command没写但args写了，那么Dockerfile中的ENTRYPOINT的会被执行，使用当前args的参数；
4、如果command和args都写了，那么Dockerfile会被忽略，执行pod当前定义的command和args。

```

## 

# 简述 Kubernetes 中 Pod 可能位于的状态？ 

Pending ：API Server 已经创建该 Pod，且 Pod 内还有一个或多个容器的镜像没有创建，包括正 

在下载镜像的过程。 

Running ：Pod 内所有容器均已创建，且至少有一个容器处于运行状态、正在启动状态或正在重 

启状态。 

Succeeded ：Pod 内所有容器均成功执行退出，且不会重启。 

Failed ：Pod 内所有容器均已退出，但至少有一个容器退出为失败状态。 

Unknown ：由于某种原因无法获取该 Pod 状态，可能由于网络通信不畅导致。

# 简述 Kubernetes 创建一个 Pod 的主要流程？ 

Kubernetes 中创建一个 Pod 涉及多个组件之间联动，主要流程如下： 

\1. 客户端提交 Pod 的配置信息（可以是 yaml 文件定义的信息）到 kube-apiserver。 

\2. Apiserver 收到指令后，通知给 controller-manager 创建一个资源对象。 

\3. Controller-manager 通过 api-server 将 pod 的配置信息存储到 ETCD 数据中心中。 

\4. Kube-scheduler 检测到 pod 信息会开始调度预选，会先过滤掉不符合 Pod 资源配置要求的节点， 

然后开始调度调优，主要是挑选出更适合运行 pod 的节点，然后将 pod 的资源配置单发送到 

node 节点上的 kubelet 组件上。 

\5. Kubelet 根据 scheduler 发来的资源配置单运行 pod，运行成功后，将 pod 的运行信息返回给 

scheduler，scheduler 将返回的 pod 运行状况的信息存储到 etcd 数据中心。

# 简述 Kubernetes 中 Pod 的重启策略？ 

Pod 重启策略（RestartPolicy）应用于 Pod 内的所有容器，并且仅在 Pod 所处的 Node 上由 kubelet 

进行判断和重启操作。当某个容器异常退出或者健康检查失败时，kubelet 将根据 RestartPolicy 的设置 

来进行相应操作。 

Pod 的重启策略包括 Always、OnFailure 和 Never，默认值为 Always。 

Always ：当容器失效时，由 kubelet 自动重启该容器； 

OnFailure ：当容器终止运行且退出码不为 0 时，由 kubelet 自动重启该容器； 

Never ：不论容器运行状态如何，kubelet 都不会重启该容器。 

同时 Pod 的重启策略与控制方式关联，当前可用于管理 Pod 的控制器包括 ReplicationController、 

Job、DaemonSet 及直接管理 kubelet 管理（静态 Pod）

不同控制器的重启策略限制如下： 

RC 和 DaemonSet：必须设置为 Always，需要保证该容器持续运行； 

Job：OnFailure 或 Never，确保容器执行完成后不再重启； 

kubelet：在 Pod 失效时重启，不论将 RestartPolicy 设置为何值，也不会对 Pod 进行健康检查。

# .简述 Kubernetes 中 Pod 的健康检查方式？ 

对 Pod 的健康检查可以通过两类探针来检查：LivenessProbe 和 ReadinessProbe。 

LivenessProbe 探针 ：用于判断容器是否存活（running 状态），如果 LivenessProbe 探针探测 

到容器不健康，则 kubelet 将杀掉该容器，并根据容器的重启策略做相应处理。若一个容器不包含 

LivenessProbe 探针，kubelet 认为该容器的 LivenessProbe 探针返回值用于是“Success”。 

ReadineeProbe 探针 ：用于判断容器是否启动完成（ready 状态）。如果 ReadinessProbe 探针 

探测到失败，则 Pod 的状态将被修改。Endpoint Controller 将从 Service 的 Endpoint 中删除包 

含该容器所在 Pod 的 Eenpoint。 

startupProbe 探针 ：启动检查机制，应用一些启动缓慢的业务，避免业务长时间启动而被上面 

两类探针 kill 掉。

# 简述 Kubernetes Pod 的 LivenessProbe 探针的常见方式？ 

kubelet 定期执行 LivenessProbe 探针来诊断容器的健康状态，通常有以下三种方式： 

ExecAction ：在容器内执行一个命令，若返回码为 0，则表明容器健康。 

TCPSocketAction ：通过容器的 IP 地址和端口号执行 TCP 检查，若能建立 TCP 连接，则表明容 

器健康。 

HTTPGetAction ：通过容器的 IP 地址、端口号及路径调用 HTTP Get 方法，若响应的状态码大于 

等于 200 且小于 400，则表明容器健康。

# 简述 Kubernetes Pod 的常见调度方式？ 

Kubernetes 中，Pod 通常是容器的载体，主要有如下常见调度方式： 

Deployment 或 RC：该调度策略主要功能就是自动部署一个容器应用的多份副本，以及持续监控 

副本的数量，在集群内始终维持用户指定的副本数量。 

NodeSelector：定向调度，当需要手动指定将 Pod 调度到特定 Node 上，可以通过 Node 的标签 

（Label）和 Pod 的 nodeSelector 属性相匹配。 

NodeAffinity 亲和性调度：亲和性调度机制极大的扩展了 Pod 的调度能力，目前有两种节点亲和 

力表达： 

requiredDuringSchedulingIgnoredDuringExecution：硬规则，必须满足指定的规则，调度器才 

可以调度 Pod 至 Node 上（类似 nodeSelector，语法不同）。 

preferredDuringSchedulingIgnoredDuringExecution：软规则，优先调度至满足的 Node 的节 

点，但不强求，多个优先级规则还可以设置权重值。 

Taints 和 Tolerations（污点和容忍）： 

Taint：使 Node 拒绝特定 Pod 运行； 

Toleration：为 Pod 的属性，表示 Pod 能容忍（运行）标注了 Taint 的 Node。

# .简述 Kubernetes 初始化容器（init container）？ 

init container 的运行方式与应用容器不同，它们必须先于应用容器执行完成，当设置了多个 init 

container 时，将按顺序逐个运行，并且只有前一个 init container 运行成功后才能运行后一个 init 

container。当所有 init container 都成功运行后，Kubernetes 才会初始化 Pod 的各种信息，并开始创 

建和运行应用容器

# 就绪探针与存活探针区别是什么？

两者作用不一样。

存活探针，是检测容器是否存活，如果检测失败，kubelet将调用容器运行时（如docker）将检查失败的容器杀死，创建新的启动容器来保持pod正常工作；

就绪探针，是检测容器是否可以正常接收流量，当就绪探针检查失败，并不重启容器，而是将pod移出endpoint列表，就绪探针确保了service中的pod都是可用的，确保客户端只与正常的pod交互并且客户端永远不会知道系统存在问题

 

# service是如何与pod关联的？

答案：通过标签选择器。每一个由deployment创建的pod都带有标签，这样，service就可以定义标签选择器来关联哪些pod是作为其后端了，就是这样，service就与pod关联在一起了。

### service的类型有哪几种

service的类型一般有4种，分别是：

```
ClusterIP：表示service仅供集群内部使用,默认值就是ClusterIP类型
NodePort：表示service可以对外访问应用,会在每个节点上暴露一个端口,这样外部浏览器访问地址为：任意节点的IP：NodePort就能连上service了
LoadBalancer：表示service对外访问应用,这种类型的service是公有云环境下的service,此模式需要外部云厂商的支持,需要有一个公网IP地址
ExternalName：这种类型的service会把集群外部的服务引入集群内部,这样集群内直接访问service就可以间接的使用集群外部服务了

一般情况下，service都是ClusterIP类型的，通过ingress接入的外部流量。

```

# 一个应用pod是如何发现service的，或者说，pod里面的容器用于是如何连接service的？

有两种方式，一种是通过环境变量，另一种是通过service的dns域名方式。

1、环境变量：当pod被创建之后，k8s系统会自动为容器注入集群内有效的service名称和端口号等信息为环境变量的形式，这样容器应用直接通过取环境变量值就能访问service了，如，每个pod都会自动注入了api-server的svc：curl http://${KUBERNETES_SERVICE_HOST}:{KUBERNETES_SERVICE_PORT}

2、DNS方式：使用dns域名解析的前提是k8s集群内有DNS域名解析服务器，默认k8s中会有一个CoreDNS作为k8s集群的默认DNS服务器提供域名解析服务器；service的DNS域名表示格式为<servicename>.<namespace>.svc.<clusterdomain>，servicename是service的名称，namespace是service所处的命名空间，clusterdomain是k8s集群设置的域名后缀，一般默认为 cluster.local ，这样容器应用直接通过service域名就能访问service了，如wget <http://nginx-svc.default.svc.cluster.local:80>，另外，service的port端口如果定义了名称，那么port也可以通过DNS进行解析，格式为：_<portname>._<protocol>.<servicename>.<namespace>.svc.<clusterdomain>

# 如何创建一个service代理外部的服务，或者换句话来说，在k8s集群内的应用如何访问外部的服务，如数据库服务，缓存服务等?

答：可以通过创建一个没有标签选择器的service来代理集群外部的服务。

1、创建service时不指定selector标签选择器，但需要指定service的port端口、端口的name、端口协议等，这样创建出来的service因为没有指定标签选择器就不会自动创建endpoint；

2、手动创建一个与service同名的endpoint，endpoint中定义外部服务的IP和端口，endpoint的名称一定要与service的名称一样，端口协议也要一样，端口的name也要与service的端口的name一样，不然endpoint不能与service进行关联。

完成以上两步，k8s会自动将service和同名的endpoint进行关联，这样，k8s集群内的应用服务直接访问这个service就可以相当于访问外部的服务了。

# service、endpoint、kube-proxy三种的关系是什么？

```
service：在kubernetes中，service是一种为一组功能相同的pod提供单一不变的接入点的资源。当service被建立时，service的IP和端口不会改变，这样外部的客户端（也可以是集群内部的客户端）通过service的IP和端口来建立链接，这些链接会被路由到提供该服务的任意一个pod上。通过这样的方式，客户端不需要知道每个单独提供服务的pod地址，这样pod就可以在集群中随时被创建或销毁。
endpoint：service维护一个叫endpoint的资源列表，endpoint资源对象保存着service关联的pod的ip和端口。从表面上看，当pod消失，service会在endpoint列表中剔除pod，当有新的pod加入，service就会将pod ip加入endpoint列表；但是正在底层的逻辑是，endpoint的这种自动剔除、添加、更新pod的地址其实底层是由endpoint controller控制的，endpoint controller负责监听service和对应的pod副本的变化，如果监听到service被删除，则删除和该service同名的endpoint对象，如果监听到新的service被创建或者修改，则根据该service信息获取得相关pod列表，然后创建或更新service对应的endpoint对象，如果监听到pod事件，则更新它所对应的service的endpoint对象。
kube-proxy：kube-proxy运行在node节点上，在Node节点上实现Pod网络代理，维护网络规则和四层负载均衡工作，kube-proxy会监听api-server中从而获取service和endpoint的变化情况，创建并维护路由规则以提供服务IP和负载均衡功能。简单理解此进程是Service的透明代理兼负载均衡器，其核心功能是将到某个Service的访问请求转发到后端的多个Pod实例上。
————————————————


```

# 无头service和普通的service有什么区别，无头service使用场景是什么？

答：无头service没有cluster ip，在定义service时将 service.spec.clusterIP：None，就表示创建的是无头service。

普通的service是用于为一组后端pod提供请求连接的负载均衡，让客户端能通过固定的service ip地址来访问pod，这类的pod是没有状态的，同时service还具有负载均衡和服务发现的功能。普通service跟我们平时使用的nginx反向代理很相识。

但是，试想这样一种情况，有6个redis pod ,它们相互之间要通信并要组成一个redis集群，不在需要所谓的service负载均衡，这时无头service就是派上用场了，无头service由于没有cluster ip，kube-proxy就不会处理它也就不会对它生成规则负载均衡，无头service直接绑定的是pod 的ip。无头service仍会有标签选择器，有标签选择器就会有endpoint资源。

使用场景：无头service一般用于有状态的应用场景，如Kaka集群、Redis集群等，这类pod之间需要相互通信相互组成集群，不在需要所谓的service负载均衡。

# deployment怎么扩容或缩容？

直接修改pod副本数即可，可以通过下面的方式来修改pod副本数：

1、直接修改yaml文件的replicas字段数值，然后kubectl apply -f xxx.yaml来实现更新；

2、使用kubectl edit deployment xxx 修改replicas来实现在线更新；

3、使用kubectl scale --replicas=5 deployment/deployment-nginx命令来扩容缩容。

# deployment的更新升级策略有哪些？

答：deployment的升级策略主要有两种。

1、Recreate 重建更新：这种更新策略会杀掉所有正在运行的pod，然后再重新创建的pod；

2、rollingUpdate 滚动更新：这种更新策略，deployment会以滚动更新的方式来逐个更新pod，同时通过设置滚动更新的两个参数maxUnavailable、maxSurge来控制更新的过程。

# deployment的滚动更新策略有两个特别主要的参数，解释一下它们是什么意思？

deployment的滚动更新策略，rollingUpdate 策略，主要有两个参数，maxUnavailable、maxSurge。

```
maxUnavailable：最大不可用数，maxUnavailable用于指定deployment在更新的过程中不可用状态的pod的最大数量，maxUnavailable的值可以是
一个整数值，也可以是pod期望副本的百分比，如25%，计算时向下取整。
maxSurge：最大激增数，maxSurge指定deployment在更新的过程中pod的总数量最大能超过pod副本数多少个，maxUnavailable的值可以是一个整数
值，也可以是pod期望副本的百分比，如25%，计算时向上取整。

```

# deployment更新的命令有哪些？

答：可以通过三种方式来实现更新deployment。

1、直接修改yaml文件的镜像版本，然后kubectl apply -f xxx.yaml来实现更新；

2、使用kubectl edit deployment xxx 实现在线更新；

3、使用kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1 命令来更新。

# 简述一下deployment的更新过程 （经常问）

deployment是通过控制replicaset来实现，由replicaset真正创建pod副本，每更新一次deployment，都会创建新的replicaset，下面来举例deployment的更新过程：

假设要升级一个nginx-deployment的版本镜像为nginx:1.9，deployment的定义滚动更新参数如下：

```
replicas: 3
deployment.spec.strategy.type: RollingUpdate
maxUnavailable：25%
maxSurge：25%

通过计算我们得出，3*25%=0.75，maxUnavailable是向下取整，则maxUnavailable=0，maxSurge是向上取整，则maxSurge=1，所以我们得出在整个deployment升级镜像过程中，不管旧的pod和新的pod是如何创建消亡的，pod总数最大不能超过3+maxSurge=4个，最大pod不可用数3-maxUnavailable=3个。

现在具体讲一下deployment的更新升级过程：
使用`kubectl set image deployment/nginx nginx=nginx:1.9 --record` 命令来更新；
1、deployment创建一个新的replaceset，先新增1个新版本pod，此时pod总数为4个，不能再新增了，再新增就超过pod总数4个了；旧=3，新=1，总=4；
2、减少一个旧版本的pod，此时pod总数为3个，这时不能再减少了，再减少就不满足最大pod不可用数3个了；旧=2，新=1，总=3；
3、再新增一个新版本的pod，此时pod总数为4个，不能再新增了；旧=2，新=2，总=4；
4、减少一个旧版本的pod，此时pod总数为3个，这时不能再减少了；旧=1，新=2，总=3；
5、再新增一个新版本的pod，此时pod总数为4个，不能再新增了；旧=1，新=3，总=4；
6、减少一个旧版本的pod，此时pod总数为3个，更新完成，pod都是新版本了；旧=0，新=3，总=3；

```

# 讲一下都有哪些存储卷，作用分别是什么?

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1713680293873-64f5be89-8d49-4945-9f6d-c2719e973f8f.png)

# pv的访问模式有哪几种

pv的访问模式有3种，如下：

ReadWriteOnce，简写：RWO	表示，只仅允许单个节点以读写方式挂载；

ReadOnlyMany，简写：ROX	表示，可以被许多节点以只读方式挂载；

ReadWriteMany，简写：RWX	表示，可以被多个节点以读写方式挂载；

# 

# pv的回收策略有哪几种

主要有2中回收策略：retain 保留、delete 删除。

Retain：保留，该策略允许手动回收资源，当删除PVC时，PV仍然存在，PV被视为已释放，管理员可以手动回收卷。

Delete：删除，如果Volume插件支持，删除PVC时会同时删除PV，动态卷默认为Delete，目前支持Delete的存储后端包括AWS EBS，GCE PD，Azure Disk，OpenStack Cinder等。

Recycle：回收，如果Volume插件支持，Recycle策略会对卷执行rm -rf清理该PV，并使其可用于下一个新的PVC，但是本策略将来会被弃用，目前只有NFS和HostPath支持该策略。（这种策略已经被废弃，不用记）

 

# 在pv的生命周期中，一般有几种状态

pv一共有4中状态，分别是：
创建pv后，pv的的状态有以下4种：Available（可用）、Bound（已绑定）、Released（已释放）、Failed（失败） 

```
Available，表示pv已经创建正常，处于可用状态；
Bound，表示pv已经被某个pvc绑定，注意，一个pv一旦被某个pvc绑定，那么该pvc就独占该pv，其他pvc不能再与该pv绑定；
Released，表示pvc被删除了，pv状态就会变成已释放；
Failed，表示pv的自动回收失败；

```

  

# 存储类的资源回收策略:

主要有2中回收策略，delete 删除，默认就是delete策略、retain 保留。

Retain：保留，该策略允许手动回收资源，当删除PVC时，PV仍然存在，PV被视为已释放，管理员可以手动回收卷。

Delete：删除，如果Volume插件支持，删除PVC时会同时删除PV，动态卷默认为Delete，目前支持Delete的存储后端包括AWS EBS，GCE PD，Azure Disk，OpenStack Cinder等。

注意：使用存储类动态创建的pv默认继承存储类的回收策略，当然当pv创建后你也可以手动修改pv的回收策略。

# pv存储空间不足怎么扩容?

一般的，我们会使用动态分配存储资源，在创建storageclass时指定参数 allowVolumeExpansion：true，表示允许用户通过修改pvc申请的存储空间自动完成pv的扩容，当增大pvc的存储空间时，不会重新创建一个pv，而是扩容其绑定的后端pv。这样就能完成扩容了。但是allowVolumeExpansion这个特性只支持扩容空间不支持减少空间。

# 简述 Kubernetes RC 的机制

Replication Controller 用来管理 Pod 的副本，保证集群中存在指定数量的 Pod 副本。当定义了 RC 并 

提交至 Kubernetes 集群中之后，Master 节点上的 Controller Manager 组件获悉，并同时巡检系统中 

当前存活的目标 Pod，并确保目标 Pod 实例的数量刚好等于此 RC 的期望值，若存在过多的 Pod 副本在 

运行，系统会停止一些 Pod，反之则自动创建一些 Pod。

# .简述 Kubernetes Replica Set 和 Replication Controller 之间 有什么区别？ 

Replica Set 和 Replication Controller 类似，都是确保在任何给定时间运行指定数量的 Pod 副本。不同 

之处在于 RS 使用基于集合的选择器，而 Replication Controller 使用基于权限的选择器。

# 简述 kube-proxy 作用？ 

kube-proxy 运行在所有节点上，它监听 apiserver 中 service 和 endpoint 的变化情况，创建路由规则 

以提供服务 IP 和负载均衡功能。简单理解此进程是 Service 的透明代理兼负载均衡器，其核心功能是将 

到某个 Service 的访问请求转发到后端的多个 Pod 实例上。

# 简述 Kubernetes 中什么是静态 Pod？ 

静态 pod 是由 kubelet 进行管理的仅存在于特定 Node 的 Pod 上，他们不能通过 API Server 进行管 

理，无法与 ReplicationController、Deployment 或者 DaemonSet 进行关联，并且 kubelet 无法对他 

们进行健康检查。静态 Pod 总是由 kubelet 进行创建，并且总是在 kubelet 所在的 Node 上运行。

# 

# .简述 Kubernetes deployment 升级过程？ 

初始创建 Deployment 时，系统创建了一个 ReplicaSet，并按用户的需求创建了对应数量的 Pod 

副本。 

当更新 Deployment 时，系统创建了一个新的 ReplicaSet，并将其副本数量扩展到 1，然后将旧 

ReplicaSet 缩减为 2。 

之后，系统继续按照相同的更新策略对新旧两个 ReplicaSet 进行逐个调整。 

最后，新的 ReplicaSet 运行了对应个新版本 Pod 副本，旧的 ReplicaSet 副本数量则缩减为 0。

# .简述 Kubernetes deployment 升级策略？ 

在 Deployment 的定义中，可以通过 spec.strategy 指定 Pod 更新的策略，目前支持两种策略： 

Recreate（重建）和 RollingUpdate（滚动更新），默认值为 RollingUpdate。 

Recreate ：设置 spec.strategy.type=Recreate，表示 Deployment 在更新 Pod 时，会先杀掉所 

有正在运行的 Pod，然后创建新的 Pod。 

RollingUpdate ：设置 spec.strategy.type=RollingUpdate，表示 Deployment 会以滚动更新的 

方式来逐个更新 Pod。同时，可以通过设置 spec.strategy.rollingUpdate 下的两个参数 

（maxUnavailable 和 maxSurge）来控制滚动更新的过程。

# 简述 Kubernetes DaemonSet 类型的资源特性？ 

DaemonSet 资源对象会在每个 Kubernetes 集群中的节点上运行，并且每个节点只能运行一个 pod， 

这是它和 deployment 资源对象的最大也是唯一的区别。 

因此，在定义 yaml 文件中，不支持定义 replicas。 

它的一般使用场景如下： 

在去做每个节点的日志收集工作。 

监控每个节点的的运行状态

# 简述 Kubernetes 自动扩容机制？ 

Kubernetes 使用 Horizontal Pod Autoscaler（HPA）的控制器实现基于 CPU 使用率进行自动 Pod 扩 

缩容的功能。 

HPA 控制器周期性地监测目标 Pod 的资源性能指标，并与 HPA 资源对象中的扩缩容条件进行对比，在 

满足条件时对 Pod 副本数量进行调整。

**HPA ****原理 **

ubernetes 中的某个 Metrics Server（Heapster 或自定义 Metrics Server）持续采集所有 Pod 副本的 

指标数据。 

HPA 控制器通过 Metrics Server 的 API（Heapster 的 API 或聚合 API）获取这些数据，基于用户定义的 

扩缩容规则进行计算，得到目标 Pod 副本数量。 

当目标 Pod 副本数量与当前副本数量不同时，HPA 控制器就向 Pod 的副本控制器（Deployment、RC 

或 ReplicaSet）发起 scale 操作，调整 Pod 的副本数量，完成扩缩容操作。
Horizontal Pod Autoscaler（HPA）和 Vertical Pod Autoscaler（VPA）都是 Kubernetes 中用于自动调整应用资源的工具，但它们的作用对象和调整策略略有不同。

1. Horizontal Pod Autoscaler（HPA）：

- - HPA 用于自动调整部署中 Pod 的数量，以应对应用负载的变化。
  - 当应用的 CPU 使用率或内存使用率超过或低于预设的阈值时，HPA 可以自动增加或减少 Pod 的数量，以确保应用能够处理更多的流量或节省资源。
  - HPA 通过增加或减少 Pod 的数量来实现水平扩展或收缩，以满足应用的性能需求。

1. Vertical Pod Autoscaler（VPA）：

- - VPA 用于自动调整单个 Pod 中的资源限制和请求，以优化 Pod 的性能和资源利用率。
  - VPA 根据 Pod 的实际资源需求和使用情况，动态调整 Pod 中的 CPU 和内存的请求和限制，以避免资源浪费和提高应用性能。
  - VPA 通过调整单个 Pod 中的资源请求和限制来优化资源利用率和性能，而不是通过增加或减少 Pod 的数量来实现扩展或收缩。

总的来说，HPA 主要用于水平伸缩应用程序，而 VPA 则主要用于优化单个 Pod 的资源利用率和性能。在实际应用中，可以根据需求组合使用这两种 Autoscaler，以获得更好的应用性能和资源利用率。

# .简述 Kubernetes Service 类型？ 

通过创建 Service，可以为一组具有相同功能的容器应用提供一个统一的入口地址，并且将请求负载分发 

到后端的各个容器应用上。其主要类型有： 

ClusterIP ：虚拟的服务 IP 地址，该地址用于 Kubernetes 集群内部的 Pod 访问，在 Node 上 

kube-proxy 通过设置的 iptables 规则进行转发；NodePort ：使用宿主机的端口，使能够访问各 Node 的外部客户端通过 Node 的 IP 地址和端口号 

就能访问服务； 

LoadBalancer ：使用外接负载均衡器完成到服务的负载分发，需要在 spec.status.loadBalancer 

字段指定外部负载均衡器的 IP 地址，通常用于公有云

# 简述 Kubernetes Service 分发后端的策略？ 

Service 负载分发的策略有：RoundRobin 和 SessionAffinity 

RoundRobin：默认为轮询模式，即轮询将请求转发到后端的各个 Pod 上。 

SessionAffinity：基于客户端 IP 地址进行会话保持的模式，即第 1 次将某个客户端发起的请求转 

发到后端的某个 Pod 上，之后从相同的客户端发起的请求都将被转发到后端相同的 Pod 上。

# .简述 Kubernetes Headless Service？ 

在某些应用场景中，若需要人为指定负载均衡器，不使用 Service 提供的默认负载均衡的功能，或者应 

用程序希望知道属于同组服务的其他实例。 

Kubernetes 提供了 Headless Service 来实现这种功能，即不为 Service 设置 ClusterIP（入口 IP 地 

址），仅通过 Label Selector 将后端的 Pod 列表返回给调用的客户端。

# 简述 Kubernetes 外部如何访问集群内的服务？

对于 Kubernetes，集群外的客户端默认情况，无法通过 Pod 的 IP 地址或者 Service 的虚拟 IP 地址:虚 

拟端口号进行访问。 

通常可以通过以下方式进行访问 Kubernetes 集群内的服务： 

映射 Pod 到物理机：将 Pod 端口号映射到宿主机，即在 Pod 中采用 hostPort 方式，以使客户端 

应用能够通过物理机访问容器应用。 

映射 Service 到物理机：将 Service 端口号映射到宿主机，即在 Service 中采用 nodePort 方式， 

以使客户端应用能够通过物理机访问容器应用。 

映射 Sercie 到 LoadBalancer：通过设置 LoadBalancer 映射到云服务商提供的 LoadBalancer 地 

址。这种用法仅用于在公有云服务提供商的云平台上设置 Service 的场景。

# 简述 Kubernetes ingress？ 

Kubernetes 的 Ingress 资源对象，用于将不同 URL 的访问请求转发到后端不同的 Service，以实现 

HTTP 层的业务路由机制。 

Kubernetes 使用了 Ingress 策略和 Ingress Controller，两者结合并实现了一个完整的 Ingress 负载均 

衡器。 

使用 Ingress 进行负载分发时，Ingress Controller 基于 Ingress 规则将客户端请求直接转发到 Service 

对应的后端 Endpoint（Pod）上，从而跳过 kube-proxy 的转发功能，kube-proxy 不再起作用，全过 

程为：ingress controller + ingress 规则 —-> services。 

同时当 Ingress Controller 提供的是对外服务，则实际上实现的是边缘路由器的功能。

# 简述 Kubernetes 镜像的下载策略？ 

K8s 的镜像下载策略有三种：Always、Never、IFNotPresent。 

Always ：镜像标签为 latest 时，总是从指定的仓库中获取镜像。 

Never ：禁止从仓库中下载镜像，也就是说只能使用本地镜像。 

IfNotPresent ：仅当本地没有对应镜像时，才从目标仓库中下载。默认的镜像下载策略是：当镜 

像标签是 latest 时，默认策略是 Always；当镜像标签是自定义时（也就是标签不是 latest），那么默认策略是 IfNotPresent。

# .简述 Kubernetes 的负载均衡器？ 

负载均衡器是暴露服务的最常见和标准方式之一。 

根据工作环境使用两种类型的负载均衡器，即内部负载均衡器或外部负载均衡器。 

内部负载均衡器自动平衡负载并使用所需配置分配容器，而外部负载均衡器将流量从外部负载引导至后 

端容器

# 简述 Kubernetes 各模块如何与 API Server 通信？ 

Kubernetes API Server 作为集群的核心，负责集群各功能模块之间的通信。 

集群内的各个功能模块通过 API Server 将信息存入 etcd，当需要获取和操作这些数据时，则通过 API 

Server 提供的 REST 接口（用 GET、LIST 或 WATCH 方法）来实现，从而实现各模块之间的信息交互。 

如 kubelet 进程与 API Server 的交互：每个 Node 上的 kubelet 每隔一个时间周期，就会调用一次 API 

Server 的 REST 接口报告自身状态，API Server 在接收到这些信息后，会将节点状态信息更新到 etcd 

中。 

如 kube-controller-manager 进程与 API Server 的交互：kube-controller-manager 中的 Node 

Controller 模块通过 API Server 提供的 Watch 接口实时监控 Node 的信息，并做相应处理。 

如 kube-scheduler 进程与 API Server 的交互：Scheduler 通过 API Server 的 Watch 接口监听到新建 

Pod 副本的信息后，会检索所有符合该 Pod 要求的 Node 列表，开始执行 Pod 调度逻辑，在调度成功 

后将 Pod 绑定到目标节点上

# 简述 Kubernetes Scheduler 作用及实现原理？

Kubernetes Scheduler 是负责 Pod 调度的重要功能模块，Kubernetes Scheduler 在整个系统中承担了 

“承上启下”的重要功能，“承上”是指它负责接收 Controller Manager 创建的新 Pod，为其调度至目标 

Node；“启下”是指调度完成后，目标 Node 上的 kubelet 服务进程接管后继工作，负责 Pod 接下来生 

命周期。 

Kubernetes Scheduler 的作用是将待调度的 Pod（API 新创建的 Pod、Controller Manager 为补足副 

本而创建的 Pod 等）按照特定的调度算法和调度策略绑定（Binding）到集群中某个合适的 Node 上， 

并将绑定信息写入 etcd 中。 

在整个调度过程中涉及**三个对象**： 

分别是待调度 Pod 列表、可用 Node 列表，以及调度算法和策略。 

Kubernetes Scheduler 通过调度算法调度为待调度 Pod 列表中的每个 Pod 从 Node 列表中选择一个最 

适合的 Node 来实现 Pod 的调度。 

随后，目标节点上的 kubelet 通过 API Server 监听到 Kubernetes Scheduler 产生的 Pod 绑定事件，然 

后获取对应的 Pod 清单，下载 Image 镜像并启动容器。

# 简述 Kubernetes Scheduler 使用哪两种算法将 Pod 绑定到 worker 节点？

Kubernetes Scheduler 根据如下两种调度算法将 Pod 绑定到最合适的工作节点： 

预选（Predicates） ：输入是所有节点，输出是满足预选条件的节点。kube-scheduler 根据预选 

策略过滤掉不满足策略的 Nodes。如果某节点的资源不足或者不满足预选策略的条件则无法通过预 

选。如“Node 的 label 必须与 Pod 的 Selector 一致”。

优选（Priorities） ：输入是预选阶段筛选出的节点，优选会根据优先策略为通过预选的 Nodes 

进行打分排名，选择得分最高的 Node。例如，资源越富裕、负载越小的 Node 可能具有越高的排 

名。

# 简述 Kubernetes kubelet 的作用？ 

在 Kubernetes 集群中，在每个 Node（又称 Worker）上都会启动一个 kubelet 服务进程。 

该进程用于处理 Master 下发到本节点的任务，管理 Pod 及 Pod 中的容器。 

每个 kubelet 进程都会在 API Server 上注册节点自身的信息，定期向 Master 汇报节点资源的使用情 

况，并通过 cAdvisor 监控容器和节点资源

# 简述 Kubernetes kubelet 监控 Worker 节点资源是使用什么  

组件来实现的？ 

kubelet 使用 cAdvisor 对 worker 节点资源进行监控。 

在 Kubernetes 系统中，cAdvisor 已被默认集成到 kubelet 组件内，当 kubelet 服务启动时，它会自动 

启动 cAdvisor 服务，然后 cAdvisor 会实时采集所在节点的性能指标及在节点上运行的容器的性能指 

标。

# 简述 Kubernetes 如何保证集群的安全性？ 

Kubernetes 通过一系列机制来实现集群的安全控制， 

主要有如下不同的维度： 

基础设施方面：保证容器与其所在宿主机的隔离； 

权限方面： 

最小权限原则：合理限制所有组件的权限，确保组件只执行它被授权的行为，通过限制单个组件的 

能力来限制它的权限范围。 

用户权限：划分普通用户和管理员的角色。 

集群方面： 

API Server 的认证授权：Kubernetes 集群中所有资源的访问和变更都是通过 Kubernetes API 

Server 来实现的，因此需要建议采用更安全的 HTTPS 或 Token 来识别和认证客户端身份 

（Authentication），以及随后访问权限的授权（Authorization）环节。 

API Server 的授权管理：通过授权策略来决定一个 API 调用是否合法。对合法用户进行授权并且随 

后在用户访问时进行鉴权，建议采用更安全的 RBAC 方式来提升集群安全授权。 

敏感数据引入 Secret 机制：对于集群敏感数据建议使用 Secret 方式进行保护。 

AdmissionControl（准入机制）：对 kubernetes api 的请求过程中，顺序为：先经过认证 & 授 

权，然后执行准入操作，最后对目标对象进行操作

# Kubernetes 准入机制？

在对集群进行请求时，每个准入控制代码都按照一定顺序执行。 

如果有一个准入控制拒绝了此次请求，那么整个请求的结果将会立即返回，并提示用户相应的 error 信 

息。 

准入控制 （AdmissionControl）准入控制本质上为一段准入代码，在对 kubernetes api 的请求过程 

中，顺序为：先经过认证 & 授权，然后执行准入操作，最后对目标对象进行操作。常用组件（控制代 

码）如下： 

AlwaysAdmit ：允许所有请求 

AlwaysDeny ：禁止所有请求，多用于测试环境。ServiceAccount ：它将 serviceAccounts 实现了自动化，它会辅助 serviceAccount 做一些事 

情，比如如果 pod 没有 serviceAccount 属性，它会自动添加一个 default，并确保 pod 的 

serviceAccount 始终存在。 

LimitRanger ：观察所有的请求，确保没有违反已经定义好的约束条件，这些条件定义在 

namespace 中 LimitRange 对象中。NamespaceExists：观察所有的请求，如果请求尝试创建一 

个不存在的 namespace，则这个请求被拒绝。

# Kubernetes RBAC 及其特点（优势）？

RBAC 是基于角色的访问控制，是一种基于个人用户的角色来管理对计算机或网络资源的访问的方法。 

相对于其他授权模式，RBAC 具有如下优势： 

对集群中的资源和非资源权限均有完整的覆盖。 

整个 RBAC 完全由几个 API 对象完成， 同其他 API 对象一样， 可以用 kubectl 或 API 进行操作。 

可以在运行时进行调整，无须重新启动 API Server。

# Kubernetes Secret 作用？

Secret 对象，主要作用是保管私密数据，比如密码、OAuth Tokens、SSH Keys 等信息。 

将这些私密信息放在 Secret 对象中比直接放在 Pod 或 Docker Image 中更安全，也更便于使用和分 

发

# Kubernetes Secret 有哪些使用方式？ 

创建完 secret 之后，可通过如下三种方式使用： 

在创建 Pod 时，通过为 Pod 指定 Service Account 来自动使用该 Secret。 

通过挂载该 Secret 到 Pod 来使用它。 

在 Docker 镜像下载时使用，通过指定 Pod 的 spc.ImagePullSecrets 来引用它。

# Kubernetes PodSecurityPolicy 机制？ 

Kubernetes PodSecurityPolicy 是为了更精细地控制 Pod 对资源的使用方式以及提升安全策略。 

在开启 PodSecurityPolicy 准入控制器后，Kubernetes 默认不允许创建任何 Pod，需要创建 

PodSecurityPolicy 策略和相应的 RBAC 授权策略（Authorizing Policies），Pod 才能创建成功。

# Kubernetes PodSecurityPolicy 机制能实现哪些安全策 略？ 

在 PodSecurityPolicy 对象中可以设置不同字段来控制 Pod 运行时的各种安全策略，常见的有： 

特权模式：privileged 是否允许 Pod 以特权模式运行。 

宿主机资源：控制 Pod 对宿主机资源的控制，如 hostPID：是否允许 Pod 共享宿主机的进程空 

间。 

用户和组：设置运行容器的用户 ID（范围）或组（范围）。 

提升权限：AllowPrivilegeEscalation：设置容器内的子进程是否可以提升权限，通常在设置非 

root 用户（MustRunAsNonRoot）时进行设置。 

SELinux：进行 SELinux 的相关配置。

# 简述 Kubernetes 网络模型？ 

Kubernetes 网络模型中每个 Pod 都拥有一个独立的 IP 地址，并假定所有 Pod 都在一个可以直接连通 

的、扁平的网络空间中。 

所以不管它们是否运行在同一个 Node（宿主机）中，都要求它们可以直接通过对方的 IP 进行访问。 

设计这个原则的原因是，用户不需要额外考虑如何建立 Pod 之间的连接，也不需要考虑如何将容器端口 

映射到主机端口等问题。 

同时为每个 Pod 都设置一个 IP 地址的模型使得同一个 Pod 内的不同容器会共享同一个网络命名空间， 

也就是同一个 Linux 网络协议栈。这就意味着同一个 Pod 内的容器可以通过 localhost 来连接对方的端 

口。 

在 Kubernetes 的集群里，IP 是以 Pod 为单位进行分配的。 

一个 Pod 内部的所有容器共享一个网络堆栈（相当于一个网络命名空间，它们的 IP 地址、网络设备、 

配置等都是共享的）。

# .简述 Kubernetes CNI 模型？ 

CNI 提供了一种应用容器的插件化网络解决方案，定义对容器网络进行操作和配置的规范，通过插件的 

形式对 CNI 接口进行实现。 

CNI 仅关注在创建容器时分配网络资源，和在销毁容器时删除网络资源。在 CNI 模型中只涉及两个概 

念：容器和网络。 

容器 （Container）：是拥有独立 Linux 网络命名空间的环境，例如使用 Docker 或 rkt 创建的容 

器。容器需要拥有自己的 Linux 网络命名空间，这是加入网络的必要条件。 

网络 （Network）：表示可以互连的一组实体，这些实体拥有各自独立、唯一的 IP 地址，可以是 

容器、物理机或者其他网络设备（比如路由器）等。 

对容器网络的设置和操作都通过插件（Plugin）进行具体实现，CNI 插件包括两种类型： 

**CNI Plugin ****和**** IPAM****（****IP Address Management****）****Plugin****。 **

CNI Plugin 负责为容器配置网络资源，IPAM Plugin 负责对容器的 IP 地址进行分配和管理。 

IPAM Plugin 作为 CNI Plugin 的一部分，与 CNI Plugin 协同工作。

# 简述 Kubernetes 网络策略？ 

为实现细粒度的容器间网络访问隔离策略，Kubernetes 引入 Network Policy。 

Network Policy 的主要功能是对 Pod 间的网络通信进行限制和准入控制，设置允许访问或禁止访问的客 

户端 Pod 列表。 

Network Policy 定义网络策略，配合策略控制器（Policy Controller）进行策略的实现。

# .简述 Kubernetes 网络策略原理？ 

Network Policy 的工作原理主要为：policy controller 需要实现一个 API Listener，监听用户设置的 

Network Policy 定义，并将网络访问规则通过各 Node 的 Agent 进行实际设置（Agent 则需要通过 CNI 

网络插件实现）

# 简述 Kubernetes 中 flannel 的作用？ 

Flannel 可以用于 Kubernetes 底层网络的实现，主要作用有： 

它能协助 Kubernetes，给每一个 Node 上的 Docker 容器都分配互相不冲突的 IP 地址。 

它能在这些 IP 地址之间建立一个覆盖网络（Overlay Network），通过这个覆盖网络，将数据包原 

封不动地传递到目标容器内。

# Kubernetes Calico 网络组件实现原理？ 

Calico 是一个基于 BGP 的纯三层的网络方案，与 OpenStack、Kubernetes、AWS、GCE 等云平台都能 

够良好地集成。 

Calico 在每个计算节点都利用 Linux Kernel 实现了一个高效的 vRouter 来负责数据转发。每个 vRouter 

都通过 BGP 协议把在本节点上运行的容器的路由信息向整个 Calico 网络广播，并自动设置到达其他节 

点的路由转发规则。 

Calico 保证所有容器之间的数据流量都是通过 **IP ****路由的方式完成互联互通的。 **

Calico 节点组网时可以直接利用数据中心的网络结构（L2 或者 L3），不需要额外的 NAT、隧道或者 

Overlay Network，没有额外的封包解包，能够节约 CPU 运算，提高网络效率。 

# Kubernetes 共享存储的作用？ 

Kubernetes 对于有状态的容器应用或者对数据需要持久化的应用，因此需要更加可靠的存储来保存应 

用产生的重要数据，以便容器应用在重建之后仍然可以使用之前的数据。因此需要使用共享存储。

# Kubernetes 数据持久化的方式有哪些？ 

Kubernetes 通过数据持久化来持久化保存重要数据，常见的方式有： 

EmptyDir（空目录）：没有指定要挂载宿主机上的某个目录，直接由 Pod 内保部映射到宿主机 

上。类似于 docker 中的 manager volume。 

场景： 

只需要临时将数据保存在磁盘上，比如在合并/排序算法中； 

作为两个容器的共享存储。 

特性： 

同个 pod 里面的不同容器，共享同一个持久化目录，当 pod 节点删除时，volume 的数据也会被 

删除。 

emptyDir 的数据持久化的生命周期和使用的 pod 一致，一般是作为临时存储使用。 

Hostpath：将宿主机上已存在的目录或文件挂载到容器内部。类似于 docker 中的 bind mount 挂 

载方式。 

特性：增加了 pod 与节点之间的耦合。 

PersistentVolume（简称 PV）：如基于 NFS 服务的 PV，也可以基于 GFS 的 PV。它的作用是统一 

数据持久化目录，方便管理。

本质上，K8s volume是一个目录，这点和Docker volume差不多，当Volume被mount到Pod上，这个Pod中的所有容器都可以访问这个volume，在生产场景中，我们常用的类型有这几种：              

#  Kubernetes PV 和 PVC？ 

PV 是对底层网络共享存储的抽象，将共享存储定义为一种“资源”。 

PVC 则是用户对存储资源的一个“申请”。

#  Kubernetes PV 生命周期内的阶段？ 

某个 PV 在生命周期中可能处于以下 4 个阶段（Phaes）之一。 

Available：可用状态，还未与某个 PVC 绑定。 

Bound：已与某个 PVC 绑定。 

Released：绑定的 PVC 已经删除，资源已释放，但没有被集群回收。 

Failed：自动资源回收失败

# Kubernetes 所支持的存储供应模式？ 

Kubernetes 支持两种资源的存储供应模式：静态模式（Static）和动态模式（Dynamic）。 

静态模式 ***\***：**集群管理员手工创建许多 PV，在定义 PV 时需要将后端存储的特性进行设置。 

动态模式 ***\***：**集群管理员无须手工创建 PV，而是通过 StorageClass 的设置对后端存储进行描 

述，标记为某种类型。此时要求 PVC 对存储的类型进行声明，系统将自动完成 PV 的创建及与 PVC 

的绑定。

# Kubernetes CSI 模型？ 

Kubernetes CSI 是 Kubernetes 推出与容器对接的存储接口标准，存储提供方只需要基于标准接口进行 

存储插件的实现，就能使用 Kubernetes 的原生存储机制为容器提供存储服务。 

CSI 使得存储提供方的代码能和 Kubernetes 代码彻底解耦，部署也与 Kubernetes 核心组件分离，显 

然，存储插件的开发由提供方自行维护，就能为 Kubernetes 用户提供更多的存储功能，也更加安全可 

靠。 

CSI 包括 CSI Controller 和 CSI Node： 

CSI Controller 的主要功能是提供存储服务视角对存储资源和存储卷进行管理和操作。 

CSI Node 的主要功能是对主机（Node）上的 Volume 进行管理和操作。

# Kubernetes Worker 节点加入集群的过程？ 

通常需要对 Worker 节点进行扩容，从而将应用系统进行水平扩展。 

主要过程如下： 

在该 Node 上安装 Docker、kubelet 和 kube-proxy 服务； 

然后配置 kubelet 和 kubeproxy 的启动参数，将 Master URL 指定为当前 Kubernetes 集群 

Master 的地址，最后启动这些服务； 

通过 kubelet 默认的自动注册机制，新的 Worker 将会自动加入现有的 Kubernetes 集群中； 

Kubernetes Master 在接受了新 Worker 的注册之后，会自动将其纳入当前集群的调度范围。

# .简述 Kubernetes Pod 如何实现对节点的资源控制？ 

Kubernetes 集群里的节点提供的资源主要是计算资源，计算资源是可计量的能被申请、分配和使用的 

基础资源。 

当前 Kubernetes 集群中的计算资源主要包括 CPU、GPU 及 Memory。CPU 与 Memory 是被 Pod 使用 

的，因此在配置 Pod 时可以通过参数 CPU Request 及 Memory Request 为其中的每个容器指定所需使 

用的 CPU 与 Memory 量，Kubernetes 会根据 Request 的值去查找有足够资源的 Node 来调度此 

Pod。 

通常，一个程序所使用的 CPU 与 Memory 是一个动态的量，确切地说，是一个范围，跟它的负载密切 

相关：负载增加时，CPU 和 Memory 的使用量也会增加。

# .简述 Kubernetes Requests 和 Limits 如何影响 Pod 的调度？ 

当一个 Pod 创建成功时，Kubernetes 调度器（Scheduler）会为该 Pod 选择一个节点来执行。 

对于每种计算资源（CPU 和 Memory）而言，每个节点都有一个能用于运行 Pod 的最大容量值。调度 

器在调度时，首先要确保调度后该节点上所有 Pod 的 CPU 和内存的 Requests 总和，不超过该节点能提 

供给 Pod 使用的 CPU 和 Memory 的最大容量值。

# 简述 Kubernetes Metric Service？ 

在 Kubernetes 从 1.10 版本后采用 Metrics Server 作为默认的性能数据采集和监控，主要用于提供核心 

指标（Core Metrics），包括 Node、Pod 的 CPU 和内存使用指标。 

对其他自定义指标（Custom Metrics）的监控则由 Prometheus 等组件来完成。

# 简述 Kubernetes 中，如何使用 EFK 实现日志的统一管理？

在 Kubernetes 集群环境中，通常一个完整的应用或服务涉及组件过多，建议对日志系统进行集中化管 

理，通常采用 EFK 实现。 

EFK 是 Elasticsearch、Fluentd 和 Kibana 的组合，其各组件功能如下： 

Elasticsearch：是一个搜索引擎，负责存储日志并提供查询接口； 

Fluentd：负责从 Kubernetes 搜集日志，每个 node 节点上面的 fluentd 监控并收集该节点上面的 

系统日志，并将处理过后的日志信息发送给 Elasticsearch； 

Kibana：提供了一个 Web GUI，用户可以浏览和搜索存储在 Elasticsearch 中的日志。 

通过在每台 node 上部署一个以 DaemonSet 方式运行的 fluentd 来收集每台 node 上的日志。 

Fluentd 将 docker 日志目录/var/lib/docker/containers 和/var/log 目录挂载到 Pod 中，然后 Pod 会在 

node 节点的/var/log/pods 目录中创建新的目录，可以区别不同的容器日志输出，该目录下有一个日志 

文件链接到/var/lib/docker/contianers 目录下的容器日志输出。

# 简述 Kubernetes 如何进行优雅的节点关机维护？

由于 Kubernetes 节点运行大量 Pod，因此在进行关机维护之前，建议先使用 kubectl drain 将该节点的 

Pod 进行驱逐，然后进行关机维护。

# 简述 Kubernetes 集群联邦？

Kubernetes 集群联邦可以将多个 Kubernetes 集群作为一个集群进行管理。 

因此，可以在一个数据中心/云中创建多个 Kubernetes 集群，并使用集群联邦在一个地方控制/管理所 

有集群

# k8s中service和ingress的区别 

serivce是如何被设计的： 

在pod中运行的容器在动态，弹性的变化(比如容器的重启IP地址会变化），为了给pod提供一个固定 

的，统一访问的接口，以及负载均衡的能力，并借助DNS系统实现服务发现功能，解决客户端发现容器 

难的问题，于是变设计了service 

service 和pod对象的IP地址，在集群内部可达，但集群外部用户无法接入服务，解决的思路有： 

\1. node pod端口上做端口暴露 

\2. 在工作节点上用公用网络名称空间（hostname） 

\3. 使用service的nodeport或者loadbalancer 

\4. ingress七层负载和反向代理资源。 

service 提供pod的负载均衡的能力，但只在4层有负载，而没有功能，只能到IP层面。 

service的几种类型： 

clusetr IP： 默认类型，自动分配一个仅可以在内部访问的虚拟IP，仅供内部访问 

nodeport： 在clusterip的基础上，为集群内的每台物理机绑定一个端口，外网通过任意节点的物 

理机IP来访问服务，应用方式： 外服访问服务 

loadbalance: 在nodeport的基础上，提供外部负载均衡与外网统一IP，此IP可以将讲求转发到对 

应的服务上。 应用方式： 外服访问服务 

externalname : 引用集群外的服务，可以在集群内部通过别名的方式访问。 

**ingress: **

service 只能提供四层的负载，虽然可以通过nodeport的方式来服务，但随着服务的增多，会在物理机 

上开辟太多的端口，不便于管理，所以ingress换了思路来暴露服务 

创建一个具有n个副本的nginx服务，在nginx服务内配置各个服务的域名与集群内部的服务的ip，这些 

nginx服务在通过nodeport的方式来暴露，外部服务可以通过域名：nginx nodeport端口来访问 

nginx，nignx在通过域名反向代理到真实的服务器。 

ingress分为ingress controller和ingress 配置

## 标签及标签选择器是什么，如何使用？

标签是键值对类型，标签可以附加到任何资源对象上，主要用于管理对象，查询和筛选。标签常被用于标签选择器的匹配度检查，从而完成资源筛选；一个资源可以定义一个或多个标签在其上面。

标签选择器，标签要与标签选择器结合在一起，标签选择器允许我们选择标记有特定标签的资源对象子集，如pod，并对这些特定标签的pod进行查询，删除等操作。

标签和标签选择器最重要的使用之一在于，在deployment中，在pod模板中定义pod的标签，然后在deployment定义标签选择器，这样就通过标签选择器来选择哪些pod是受其控制的，service也是通过标签选择器来关联哪些pod最后其服务后端pod。

# 存活探针的属性参数有哪几个？

```
initialDelaySeconds：表示在容器启动后延时多久秒才开始探测；
periodSeconds：表示执行探测的频率，即间隔多少秒探测一次，默认间隔周期是10秒，最小1秒；
timeoutSeconds：表示探测超时时间，默认1秒，最小1秒，表示容器必须在超时时间范围内做出响应，否则视为本次探测失败；
successThreshold：表示最少连续探测成功多少次才被认定为成功，默认是1，对于liveness必须是1，最小值是1；
failureThreshold：表示连续探测失败多少次才被认定为失败，默认是3，连续3次失败，k8s 将根据pod重启策略对容器做出决定；

注意：定义存活探针时，一定要设置initialDelaySeconds属性，该属性为初始延时，如果不设置，默认容器启动时探针就开始探测了，这样可能会存在
应用程序还未启动就绪，就会导致探针检测失败，k8s就会根据pod重启策略杀掉容器然后再重新创建容器的莫名其妙的问题。
在生产环境中，一定要定义一个存活探针。

```

  

#  k8s是怎么做日志监控的 

一般有三种方式： 

\1. 将fluentd项目安装在宿主机上，然后把日志转发到远端的elsticSearch里保存起来以备检索。 

这样做的优点是： 在一个节点上只需要部署一个agent，并且不会对应用和pod有任何入侵性，这 

种用的比较多一些。 

缺点： 它要求应用输出日志，都必须直接输出到容器的stdout和stderr里 

\2. 第二种方案：当容器日志只能输出某些文件的时候，我们可以通过一个sidecar容器把这些日志文 

件，重新输出到sidecar的stdout和stderr上，然后在使用第一种方案。其实第二种方案就是对第一种方案缺点的补充 

\3. 第三种方案，通过一个sidecar的容器，直接把应用的日志发送到远程存储里面。 

这种其实也是第一种的延伸，就是把fluentd部署到pod中，后端存储还是可以用elasticsearch，知识 

fluentd输入源变成了应用文件日志。 

但是这种方法sidecar容器会消耗过多资源。 

日过日志量特比大，我们可以增加配额： 给容器上挂存储，讲日志输出到存储上。 

你在回答这个问题的时候，可以说第一种方式，只要你们的日志量不大即可，如果大的话，需要加存 

储。

# K8S监控指标 

一般在公司里我们都是使用prometheus进行监控，先说一下prometheus的工作核心： 

prometheus是使用 Pull （抓取）的方式去搜集被监控对象的 Metrics 数据（监控指标数据），然后， 

再把这些数据保存在一个 TSDB （时间序列数据库，比如 OpenTSDB、InfluxDB 等）当中，以便后续可 

以按照时间进行检索。 

有了这套核心监控机制， Prometheus 剩下的组件就是用来配合这套机制的运行。比如 

Pushgateway，可以允许被监控对象以 Push 的方式向 Prometheus 推送 Metrics 数据。 

而 Alertmanager，则可以根据 Metrics 信息灵活地设置报警。当然， Prometheus 最受用户欢迎的功 

能，还是通过 Grafana 对外暴露出的、可以灵活配置的监控数据可视化界面。 

kubernetes的监控体系： 

宿主机的监控数据： 比如节点的负载，CPU，内存，磁盘，网络这些常规的信息，当然你也可以查看htt 

ps://github.com/prometheus/node_exporter#enabled-by-default 

来看看这些指标，实在是太多了。 

对apiserver，kubelet等组件的监控，比如工作队列的长度，请求的QPS和数据延迟等，主要是检查k8s 

本身的工作情况 

k8s相关的监控数据，比如对pod，node，容器，service等主要k8s概念进行监控。 

在监控指标的规划上需要遵从USE原则和RED原则 

USE: 

利用率 

饱和度 

错误率 

RED原则： 

\4. 每秒请求数 

\5. 每秒错误数 

\6. 服务响应时间 

这里需要注意： promotheus采用的是pull的模式。

# HPA跟VPA

1. **Horizontal Pod Autoscaler (HPA)**:

- - HPA 用于根据应用程序的 CPU 使用率或自定义指标（如内存使用率）来自动扩展 Pod 的数量。当应用程序的负载增加时，HPA 可以根据定义的规则自动增加 Pod 的副本数量，以应对负载增加的情况。
  - HPA 主要关注的是水平扩展，即增加 Pod 的副本数量来应对负载增加的情况，从而实现应用程序的横向扩展。

1. **Vertical Pod Autoscaler (VPA)**:

- - VPA 则是用于根据应用程序的资源需求（如 CPU 和内存）来自动调整 Pod 的资源请求和限制。VPA 可以根据应用程序的实际资源需求来动态调整 Pod 的资源配置，以确保应用程序能够获得足够的资源。
  - VPA 主要关注的是垂直扩展，即根据应用程序的资源需求来调整 Pod 的资源配置，以确保应用程序能够获得足够的资源，从而实现应用程序的纵向扩展。

# 亲和性、反亲和性、污点、容忍和节点选择器

在 Kubernetes 中部署应用程序时，可以使用亲和性（Affinity）、反亲和性（Anti-Affinity）、污点（Taint）、容忍（Toleration）和节点选择器（Node Selector）等功能来控制 Pod 如何在集群中调度和运行。下面我会简要解释它们的含义和用途：

1. **亲和性（Affinity）**：

- - 亲和性用于指定 Pod 与其他 Pod 或节点之间的关系，以影响 Pod 的调度行为。通过亲和性，您可以要求 Kubernetes 将某个 Pod 调度到与其具有特定标签的节点上，或者将多个相关的 Pod 调度到同一个节点上。

1. **反亲和性（Anti-Affinity）**：

- - 反亲和性与亲和性相反，用于指定 Pod 与其他 Pod 或节点之间的关系，以避免将某个 Pod 调度到与其具有特定标签的节点上，或者将多个相关的 Pod 避免调度到同一个节点上。

1. **污点（Taint）**：

- - 污点是一种节点级别的属性，用于标记节点上的特殊条件或限制，如节点上已经运行了某些特定的服务或应用程序。通过给节点添加污点，您可以限制哪些 Pod 可以被调度到该节点上。

1. **容忍（Toleration）**：

- - 容忍是 Pod 级别的属性，用于指定 Pod 是否能够容忍节点上的污点。通过给 Pod 添加容忍，您可以允许 Pod 被调度到具有特定污点的节点上。

1. **节点选择器（Node Selector）**：

- - 节点选择器是一种简单的节点调度策略，通过为 Pod 指定标签选择器，您可以要求 Kubernetes 将 Pod 调度到具有匹配标签的节点上。

通过结合使用亲和性、反亲和性、污点、容忍和节点选择器等功能，您可以更精细地控制 Pod 的调度行为，实现更灵活和高效的资源管理和调度策略。在设计和部署应用程序时，可以根据实际需求和场景来合理配置这些功能，以满足应用程序的运行要求。

# 什么是kubernetes operator

Kubernetes Operator 是一种用于扩展 Kubernetes 功能的软件。它的主要功能包括:

自动化操作 - Operator 可以自动执行诸如部署、扩缩容、升级、备份等常见操作。用户不需要手动一个个 Pod 去执行这些任务。

无状态应用管理 - Operator 很适合管理那些无状态的应用,比如数据库、缓存等。它通过控制器模式让应用始终处于预期的状态。

减少重复工作 - Operator 让用户从重复的日常工作中解脱出来,这些工作可以交给 Operator 来自动完成。

基于 CRD 扩展 - Operator 通过自定义资源定义(CRD)扩展 Kubernetes 的 API,并基于这些 CRD 来实现自定义控制循环。

结合服务目录 - Operator 可以与服务目录(如 Service Catalog)集成,来启用基于权限的资源控制和分发。

总的来说,Kubernetes Operator 的核心思想是通过程序化和自动化的方式来管理和扩展 Kubernetes 集群。它极大地简化了在 Kubernetes 上安装和运行复杂应用的过程。

Operator 的工作原理基于 Kubernetes 的控制器模式。它会不断地监测 Kubernetes 集群的状态,一旦发现自定义资源(CR)的实际状态与预期状态不符,Operator 就会执行相应的操作以使其达到预期状态。这种模式使得 Operator 可以实现自我修复和自动恢复的功能。

常见的 Kubernetes Operator 包括 Rook 提供存储解决方案的 Operator,Prometheus Operator 用于监控集群的 Operator,Istio Operator 用于服务网格的 Operator 等。这些 Operator 为 Kubernetes 生态带来了很大的便利。

————————————————

# 下面以sidecar 模式和log-pilot这两种方式的日志收集形式做个详细对比说明：

第一种模式是 sidecar 模式，这种需要我们在每个 Pod 中都附带一个 logging 容器来进行本 Pod 内部容器的日志采集，一般采用共享卷的方式，但是对于这一种模式来说，很明显的一个问题就是占用的资源比较多，尤其是在集群规模比较大的情况下，或者说单个节点上容器特别多的情况下，它会占用过多的系统资源，同时也对日志存储后端占用过多的连接数。当我们的集群规模越大，这种部署模式引发的潜在问题就越大。

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1713594796692-bb1b0e44-f38c-49aa-9c51-12fb820b6999.png)

另一种模式是 Node 模式，这种模式是我们在每个 Node 节点上仅需布署一个 logging 容器来进行本 Node 所有容器的日志采集。这样跟前面的模式相比最明显的优势就是占用资源比较少，同样在集群规模比较大的情况下表现出的优势越明显，同时这也是社区推荐的一种模式。

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1713594802822-e374f600-e952-478c-b839-a6fd58ad616f.png)

经过多方面测试，log-pilot对现有业务pod侵入性很小，只需要在原有pod的内传入几行env环境变量，即可对此pod相关的日志进行收集，

底层利用的fieabet 

————————————————

# 监控报警

# 部门用普罗米修斯

# 还有阿里云开源的 kube——event 事件告警 

# 安全监控faclo

# Kubernetes（K8s）要求每个节点的 Pod 数量不超过110，节点数不超过5,000，Pod 总数不超过150,000，容器总数不超过300,000，主要是为了保证集群的稳定性、可靠性和性能。这些限制是为了避免集群过载，确保集群能够有效地管理和调度大规模的容器化应用。

1. **资源限制和调度**：Kubernetes需要根据每个 Pod 的资源需求和节点的资源容量来进行调度和分配。如果每个节点上的 Pod 数量过多，可能会导致资源竞争和调度延迟，影响应用程序的性能和稳定性。通过限制每个节点的 Pod 数量和节点总数，可以更好地管理资源分配和调度。
2. **网络通信和负载均衡**：在一个节点上运行过多的 Pod 可能会导致网络通信瓶颈和负载不均衡。限制每个节点的 Pod 数量可以有效地管理网络流量和负载均衡，确保应用程序之间的通信和数据传输效率。
3. **故障隔离和容错性**：当一个节点上的 Pod 数量过多时，节点发生故障或者需要维护时可能会影响到大量的应用程序。通过限制每个节点的 Pod 数量，可以实现更好的故障隔离和容错性，减少单点故障对整个集群的影响。
4. **管理和监控**：管理大规模的容器化应用需要消耗大量的计算资源和存储资源。限制 Pod 总数和容器总数可以减轻集群管理和监控的负担，确保集群管理工具的高效运行和性能稳定。

综上所述，Kubernetes对每个节点的 Pod 数量、节点数、Pod 总数和容器总数进行限制是为了确保集群的稳定性、可靠性和性能。通过合理的资源管理和调度策略，Kubernetes可以更好地支持大规模容器化应用的部署和运行，提高整个集群的效率和可管理性。

# cicid搭建

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1713602050054-c34ce94d-e897-4f73-9fa4-e99d038f4c9b.png)

gitlab  然后gitlab-tls  gitlab-runner

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1713619051574-d989caf0-7434-4208-afa5-247499a76ca2.png)

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1713619058148-1c215db2-0c52-46b4-ae0f-cc55faf4b0d6.png)

确定runner url

# 自建

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1713602200354-c98d52be-b05f-4ee0-ae18-f97a09df96c3.png)

# 用户访问我们的k8s集群里面的应用网站，出现500报错，你是如何排查这种问题的？

按F12直接查看是哪个节点异常，然后看服务器pod状态。（待补充）

### k8s后端存储使用的是什么？

后端存储方面，我们不同的项目使用的方案不一样，这可能是早期系统架构决定的，现在也是一直沿用，主要就是包含3种存储方案：

```
1、第一种存储方案是，hostPath和local-path-provisioner，先说hostPath：pod中直接使用hostPath卷来挂载数据，然后pod中要节点选择器固定调度到指定的节点，我们一般采用这种方式来实现pod直接挂载服务器数据或者pod日志落盘到服务器磁盘；但是hostPath卷属于静态卷，所以我们还使用了local-path-provisioner来动态供给localPath，local-path-provisioner能够让Kubernetes的本地存储支持动态pv，当使用local-path-provisioner的pod被调度时，scheduler调度器和pv控制器会同时进行控制，然后在pod所在节点上创建对应的本地存储目录，当pod被重新调度后，因为pod所对应的pv存在节点选择器，所以pod仍然能够调度到之前的节点上，从而继续使用或读取之前的数据。
2、第二中存储方案是glusterfs：我们使用glusterfs作为后端存储，先在服务器上大磁盘单独划分数据分区，创建目录，然后创建一个glusterfs文件系统（glusterfs没有选举一说，可以任意数量扩展），然后在glusterfs文件系统上创建卷，然后在k8s创建存储类，这样就实现了动态供给pv，有时候我们也会把glusterfs中的卷直接挂载到服务器上使用。
3、最后一种存储方案是cephfs：ceph官网上推荐在k8s集群中使用rook-ceph，我们使用的就是rook-ceph，rook-ceph我们使用的helm安装的，创建完成ceph集群之后再创建cephfs,然后创建存储类进行动态pv供给。

```

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1708948147255-0ea62d45-ef09-4a0b-8dff-015557bf0de5.png)

部

# 托管集群扩容怎么选择

托管k8s集群，节点池自动扩容 autoscaler

<https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler>

特色: 弹性容器服务

Aliyun ACK使用ECI : <https://help.aliyun.com/document_detail/119207.html>

## 

### 1.3 Pod生命周期（课程Pod章节）

![img](https://cdn.nlark.com/yuque/0/2023/jpeg/35538885/1703042450636-b9be62a4-071c-40d8-a2dd-8cc530ed6507.jpeg)

```
答：
   Pod创建：
      1. API Server 在接收到创建pod的请求之后，会根据用户提交的参数值来创建一个运行时的pod对象。
      2. 根据 API Server 请求的上下文的元数据来验证两者的 namespace 是否匹配，如果不匹配则创建失败。
      3. Namespace 匹配成功之后，会向 pod 对象注入一些系统数据，如果 pod 未提供 pod 的名字，则 API Server 会将 pod 的 uid 作为 pod 的名字。
      4. API Server 接下来会检查 pod 对象的必需字段是否为空，如果为空，创建失败。
      5. 上述准备工作完成之后会将在 etcd 中持久化这个对象，将异步调用返回结果封装成 restful.response，完成结果反馈。
      6. API Server 创建过程完成，剩下的由 scheduler 和 kubelet 来完成，此时 pod 处于 pending 状态。
      7. Scheduler选择出最优节点。
      8. Kubelet启动该Pod。
   Pod删除：
      1. 用户发出删除 pod 命令
      2. 将 pod 标记为“Terminating”状态
         监控到 pod 对象为“Terminating”状态的同时启动 pod 关闭过程
         endpoints 控制器监控到 pod 对象关闭，将pod与service匹配的 endpoints 列表中删除
         Pod执行PreStop定义的内容
      3. 宽限期（默认30秒）结束之后，若存在任何一个运行的进程，pod 会收到 SIGKILL 信号
      4. Kubelet 请求 API Server 将此 Pod 资源宽限期设置为0从而完成删除操作
```

### 

### 1.8 公司的架构是什么样的？

答：我们的架构是这样的，三台master，三台etcd，etcd和master没有放在一起。然后在指定的节点上部署了ingress nginx，然后外部有个网关（可以选择性说网关是硬件设备F5或者DMZ的nginx，或者公有云的LB）连接到了k8s ingress节点的80和433，然后有个通配符域名指向了ingress，在ingress上面又做的分发。

### 3.3 生产环境的pv回收策略如何选择？

答：目前pv的回收策略分为recycle、delete、retain，

，可以着重回答delete和retain。其中Delete回收策略一般用于动态存储，比如ceph、GFS这类的，也就是通过StorageClass进行管理创建的pv，Delete的策略也是StorageClass的默认策略，因为当一个项目用到存储时，会通过pvc或者volumeTemplateClaim申请存储，然后后端存储会自动创建pv，所以当你删除pvc或者pv时，就认为你已经不需要这个存储了，就会触发自动删除pv，防止造成存储池存储过多无人使用的垃圾pv。

而静态文件建议使用Retain，比如NFS、NAS这类的，因为这些文件一般都是手动管理的，所以最好是尽量保持这些文件的可用性，就算不用了，也是可以根据目录名称进行手动删除。所以retain和delete是用的比较多的。

### 容器的OOM的问题吗？怎么处理的？

答：遇到OOM有两种情况，第一种情况是这个程序确实需要4Gi（假设）内存，但是你的limit配置只给了3Gi，这样就会有OOM。另外一种情况是程序本身是有内存溢出的，可能没有做好垃圾回收，导致内存一直往上涨，这样的可能需要开发人员加上相应的垃圾回收，还有一种程序内存溢出是因为limit设置的太低导致不能正常的垃圾回收，比如一个程序正常运行需要3Gi，但是垃圾回收可能也需要占用内存，所以此时给3Gi肯定是不行的，一般需要超过3Gi，也就是limit配置要超过程序需求的800M-1Gi。

### 4.4 有状态应用如何上云？

答：有状态应用其实也分为需要存储数据的和不需要存储数据的。如果是有需要存储数据的部署在K8s上，最好有后端可靠的存储支持，比如分布式的ceph或者公有云的存储，最极端的情况是没有后端存储支持，可以采用hostPath挂载，采用固定节点的形式，可以参考csi hostpath，或者storageClass hostPath。而有的有状态应用并不需要存储数据，只是想要有规定的标识符。

## Kubernetes 中的 Pod 是什么？它与容器的关系是什么？

Pod是Kubernetes的最小部署单元，可以包含一个或多个容器。通常情况下，一个Pod包含一个主容器和若干辅助容器，它们共享相同的网络命名空间和存储卷。Pod内的容器共享资源，并可以通过本地的localhost进行通信。

## 请解释一下 Kubernetes 中的 Deployment 和 StatefulSet 的区别，并提供使用场景。

- **Deployment**用于部署无状态的应用，通常是Web服务、API服务等。Deployment管理的Pod可以根据需要进行水平扩展，具有自我修复能力，支持滚动更新。
- **StatefulSet**用于部署有状态的应用，如数据库、缓存等。StatefulSet中的Pod具有唯一的标识和稳定的网络标识符，能够保证Pod的顺序部署和有序扩展，支持
- 存储。

## Kubernetes 中的 Service 是什么？它的作用是什么？与 Pod 的关系是怎样的？

Service是Kubernetes中用于暴露应用程序的网络服务的一种方式。它定义了一组Pod的访问方式，并提供了负载均衡和服务发现功能。Service与Pod的关系是通过标签选择器来建立的，它可以将请求动态地分发到与标签匹配的Pod上。

## 

# Kubernetes 中的存储卷（Volume）是什么？提供一些常见的存储卷类型和它们的用途。

1. **EmptyDir**：

- - **用途**：临时性存储，生命周期与Pod相同。
  - **描述**：在Pod创建时生成，随着Pod的删除而被清空。适合于需要短期存储数据的场景。

1. **HostPath**：

- - **用途**：将主机文件系统挂载到Pod中，可以读写主机上的文件。
  - **描述**：允许Pod直接访问主机上的文件系统，但可能会导致跨主机不可移植性，适用于对主机文件系统有特殊需求的情况。

1. **PersistentVolumeClaim（PVC）**：

- - **用途**：声明式请求持久化存储资源，可以独立于Pod创建和删除。
  - **描述**：将存储卷的生命周期与Pod解耦，可以动态地请求和释放持久化存储资源，适用于需要持久化数据的应用场景。

1. **CSI Volume**（Container Storage Interface）：

- - **用途**：支持不同存储系统的插件化接口，允许使用各种外部存储系统。
  - **描述**：通过CSI插件，可以将各种存储系统（如AWS EBS、Azure Disk、GlusterFS等）挂载到Pod中，提供灵活的存储解决方案。

1. **ConfigMap 和 Secret**：

- - **用途**：用于存储配置信息和敏感数据，可以以文件或环境变量的形式挂载到Pod中。
  - **描述**：适用于需要将配置信息或敏感数据传递给应用程序的场景，例如数据库连接字符串、API密钥等。

1. **NFS Volume**（Network File System）：

- - **用途**：挂载网络文件系统到Pod中，实现多个Pod之间共享数据。
  - **描述**：适用于需要多个Pod共享相同数据的场景，如共享配置文件、共享日志等。

这些存储卷类型提供了丰富的选项，可以根据应用程序的需求和特性选择合适的存储卷类型来实现数据的持久化存储和管理。

### 

# 说下你在k8s中遇到的问题

在使用1.25版本之后 跟1.25版本之前  k8s更换的cri  新版本的是最新的contied  在connid中  使用kubctl run 指定镜像的时候  要指定全路径 指定容器名字 会导致镜像拉不下来   pending

service  创建yaml服务  创建一个mysql服务  指定为clustip 默认分配的ip    然后去创建连接数据库的服务  为指定这个默认分配的ip  进行操作

后面update的时候这个服务没有任何变化  还必须delete掉  然后在重启   可是重启的之后默认的ip变化了。随机分配的。连接数据库的服务超时  spec.clusterIP 字段指定要分配的 Cluster IP 地址，而不是让 Kubernetes 自动分配。在 Service 的 YAML 配置文件中指定该字段即可。  当时排查半天  

当时排查一会觉得是自己连接数据库服务的没有做好  在看文档  发现不对 后面回去看默认ip更变化 后面给强一之协调了。

我用edit方法 去写这个本地的文件的话  yaml运行文件与本地文件不之智。也容易产生错误 

用nodeport模式  开启端口  任意访问 都做端口映射  但是端口限制30000-320000  端口映射麻烦  然后管理端口麻烦 

后面loadbalancer  阿里云创建slb  然后连接   暴漏你的pod端口  

fs挂载的 时候 只考虑的宿主机  容器ip没有挂虑  

cpu1m是CPU资源的最小单位，表示1毫核（即1/1000的CPU核）。例如，0.5个CPU可以表示为500m（500毫核）。CPU资源的分配和限制可以使用cpu1m单位来设置。这样做是为了更好地管理和调度容器的CPU资源。

k8s申请存储的时候坑  团队没有统一  存储  到底1g  还是1gi  这就是1000 和1024的区别  

storageclass 自动申请这

配置告警面板  与集群时间不符合   导致排错异常  

# 当 Kubernetes Pod 的状态显示为 Running 状态，但前面的状态为 0/1 时，

通常表示 Pod 处于就绪状态（Ready）之前出现了问题。这可能是由于容器启动时发生了错误或应用程序出现了问题导致的。以下是一些排查此类问题的常见方法：

\1. **查看 Pod 日志**：使用 `kubectl logs <pod_name>` 命令查看 Pod 的日志，可以查看容器启动过程中的错误消息或异常情况。

\2. **查看容器状态**：使用 `kubectl describe pod <pod_name>` 命令查看 Pod 的详细信息，包括容器的状态。您可以查看容器是否正常启动以及是否有任何错误。

\3. **进入容器调试**：使用 `kubectl exec -it <pod_name> -- /bin/bash` 进入 Pod 中的容器，手动查看容器内部的情况，例如确认应用程序是否正常运行、网络是否通畅等。

\4. **查看事件**：使用 `kubectl get events --field-selector involvedObject.name=<pod_name>` 查看与 Pod 相关的事件，可能会有一些有用的提示和警告。

\5. **查看状态检查器和存活性探针配置**：检查 Pod 的配置文件，确保状态检查器和存活性探针（liveness probe）配置正确。这些探针可以帮助 Kubernetes 确定容器是否处于运行状态，并且能够重新启动失败的容器。

通过以上方法，您可以更好地了解 Pod 运行过程中发生的问题，并采取相应的措施来排查和解决 Pod 就绪状态为 0/1 的问题。  

# 在 Kubernetes 中实现有状态 Pod 的在线迁移（热迁移）通常涉及以下几种方法：

\1. **StatefulSet 的滚动更新**：

   \- 使用 StatefulSet 来管理有状态应用，通过逐个更新 Pod 的方式来实现在线迁移。Kubernetes 会逐个终止旧 Pod 并启动新 Pod，确保在迁移过程中数据持久性和服务可用性。

   

\2. **持久卷**：

   \- 在 StatefulSet 中使用持久卷（Persistent Volume）来存储数据，确保数据在不同节点之间的持久性。这样在迁移时只需要将卷挂载到新的 Pod 上，而无需担心数据丢失。

\3. **组合使用 StatefulSet 和 Service**：

   \- 使用 StatefulSet 来管理 Pod，通过 Service 来暴露服务。在迁移过程中，可以先创建新的 Pod，并通过 Service 进行流量控制，逐渐将流量从旧 Pod 转移到新 Pod，最后再移除旧 Pod。

\4. **Operator 模式**：

   \- 编写自定义 Operator，通过监控应用状态和节点状态，实现自动化的在线迁移过程。Operator 可以根据事先定义的策略和规则来管理有状态应用的迁移过程。

\5. **服务代理**：

   \- 在迁移过程中引入服务代理，例如使用 Sidecar 容器作为代理，来自动地将流量从旧 Pod 路由到新 Pod。这样可以实现无缝的迁移过程，服务不中断。

需要注意的是，在进行有状态应用的在线迁移时，需要特别关注数据同步和服务不中断的问题。同时，针对具体的场景和需求，选择合适的迁移方案并进行充分的设计和测试是非常重要的。有状态应用的在线迁移是一个复杂的问题，需要综合考虑各种因素来选择最合适的方法。

# Sidecar 

在 Kubernetes 中，Sidecar 是一种设计模式，它指的是将附加的辅助容器（Sidecar 容器）与主应用容器共同部署在同一个 Pod 中，以提供额外的功能、辅助任务或增强主应用的能力。

通常情况下，Sidecar 容器与主应用容器共享同一网络命名空间和存储卷，它们之间可以通过 localhost 或共享的卷进行通信。Sidecar 容器可以用于日志收集、监控、安全代理、数据同步等方面。

举例来说，一个常见的用例是将日志收集器作为 Sidecar 容器部署在主应用容器旁边，用于收集和发送主应用的日志到中心化的日志存储系统。这样的设计能够将日志收集逻辑与主应用逻辑分离开来，提高了系统的可维护性和灵活性。

# rbac

Role：基于名称空间的角色。可以操作名称空间下的资源

RoleRinding: 来把一个Role。绑定给一个用户。

ClusterRole：基于集群的角色。可以操作集群资源

ClusterRoleBing: 来把一个ClusterRole,绑定给一个用户。

放弃了  记录一些笔记看看把  

## istio原理介绍

Istio是一个开源的服务网格平台，用于连接、管理和保护微服务。它通过注入代理来实现服务间的通信，提供了流量管理、安全性、可观测性和策略实施等功能。Istio的核心组件包括Envoy代理、Pilot、Citadel和Mixer。Envoy负责实际的流量代理和负载均衡，Pilot负责服务发现和路由管理，Citadel提供服务间的安全性，Mixer负责策略和遥测数据的收集。Istio通过这些组件实现了对微服务架构的管理和控制。

6-istio流量管理和命名空间

Istio提供了强大的流量管理功能，以及对命名空间的支持。

1. **流量管理：** Istio允许您通过Traffic Management模块来控制微服务之间的流量。您可以使用Istio的流量路由规则定义不同版本之间的流量分配，实现灰度发布、蓝绿部署等策略。此外，Istio还支持流量限制、故障注入和重试等功能，帮助您更好地管理和控制流量。
2. **命名空间：** 在Kubernetes中，命名空间是一种用于将集群内的资源划分为不同逻辑组的机制。Istio通过与Kubernetes命名空间的集成，使得您可以在不同的命名空间中部署和管理Istio资源。这意味着您可以在不同的命名空间中应用流量管理策略，例如，将不同命名空间中的微服务隔离开来，或者在特定命名空间中应用不同的安全策略。

总之，Istio的流量管理功能可以帮助您灵活地控制微服务之间的通信和流量，而命名空间的支持则使得这些管理功能更具灵活性和可扩展性。

7-istio虚拟服务和目标规则

Istio中的虚拟服务和目标规则是用于定义流量路由和负载均衡策略的重要组件。

1. **虚拟服务（VirtualService）：** 虚拟服务允许您定义对微服务的流量路由规则。您可以指定流量应该如何分发到不同的目标服务版本或实例，也可以定义路由规则以支持灰度发布、蓝绿部署等策略。通过虚拟服务，您可以控制流量的路由、重试策略和超时设置等。
2. **目标规则（DestinationRule）：** 目标规则用于定义微服务的负载均衡和连接池的配置。您可以在目标规则中指定负载均衡策略、连接池大小以及其他与服务端通信相关的设置。目标规则与虚拟服务结合使用，可以更精细地控制流量的路由和负载均衡行为。

这些规则的灵活性和组合可以帮助您实现对微服务间流量的精细化控制和管理，从而提高系统的可靠性和性能。

8-istio超时，重试，以及灰度发布

Istio提供了超时、重试和灰度发布等功能，帮助您更好地管理和控制微服务之间的通信和流量。

1. **超时（Timeout）：** 可以通过Istio设置超时来控制服务之间的通信时间。当一个服务调用另一个服务时，如果未能在规定时间内收到响应，则会触发超时。通过设置适当的超时策略，可以避免长时间等待响应而导致的系统资源浪费和性能问题。
2. **重试（Retry）：** Istio允许您配置重试策略，以应对由于网络不稳定或服务暂时不可用等原因而导致的请求失败。通过在服务之间进行自动重试，可以提高系统的健壮性和可靠性，确保请求能够成功完成。
3. **灰度发布（Canary Deployment）：** 灰度发布是一种逐步将新版本服务引入生产环境的策略。Istio支持通过流量路由规则来实现灰度发布，您可以逐步将流量从旧版本服务引导到新版本服务，以降低风险并及时发现潜在的问题。这种方式可以帮助您在保证系统稳定性的前提下，快速推出新功能或更新。

通过这些功能，Istio使得对微服务架构的管理和控制更加灵活和可靠，帮助您构建健壮的分布式系统。

# 9-istioessgateway

Istio的istio-ingressgateway是一个用于管理入站流量的组件，通常用于将外部流量引入到服务网格中。它基于Envoy代理构建，可以与Kubernetes中的Service对象结合使用，将外部请求路由到内部微服务。istio-ingressgateway可以配置路由规则、TLS终止、负载均衡等功能，提供了强大的入口流

# 10-istio熔断和故障注入

首先，熔断（Circuit Breaker）是一种保护机制，当下游服务出现故障或响应过慢时，上游服务会主动停止调用下游服务，以避免整个系统被拖垮。在Istio中，熔断器有三种状态：关闭、打开和半开。默认情况下，熔断器处于关闭状态，即正常调用下游服务。当下游服务连续失败次数达到预设的阈值时，熔断器会打开，此时上游服务会直接返回一个错误，不再调用下游服务。经过一段时间后，熔断器会进入半开状态，允许部分请求通过，以检查下游服务是否已恢复正常。

故障注入（Fault Injection）则是Istio提供的一种评估系统可靠性的有效方法。通过故意在服务调用过程中制造一些故障，可以验证服务的容错能力。例如，可以注入HTTP延迟故障，模拟网络延迟的情况，以测试服务在延迟条件下的表现。故障注入有助于发现潜在的问题并优化系统的故障处理能力。

在Istio中，熔断和故障注入都是可配置的，可以根据实际需求和场景进行调整。通过合理使用这两种技术，可以提高微服务的稳定性和可靠性，确保系统在面临故障、潜在峰值或其他未知网络因素影响时能够保持正常运行。

需要注意的是，熔断和故障注入都是针对微服务架构中的服务调用链路进行优化的手段。在微服务架构中，服务之间的调用链路可能变得很长，一个服务的故障可能会影响到整个链路上的其他服务。因此，需要合理设计服务拆分和调用关系，以减少级联故障的风险。同时，也需要结合其他技术手段，如限流、降级等，来共同保障微服务的稳定性和可用性。

# 11-istio流量镜像、可观测性

首先，流量镜像（Traffic Mirroring）是Istio提供的一种流量管理技术，它允许将服务的入站或出站流量复制到另一个服务实例上，通常用于测试、调试或审计等场景。这种机制可以确保在不影响正常业务流量的前提下，对新的服务版本或功能进行验证。通过流量镜像，运维人员可以在不中断现有服务的情况下，观察新服务的行为和性能，从而更加安全地进行服务升级和变更。

其次，Istio的可观测性是其另一个关键特性。通过收集和分析服务网格中的遥测数据，Istio为运维人员提供了深入的服务行为洞察。这些数据包括指标（Metrics）、日志（Logs）和链路追踪（Tracing）等，可以帮助运维人员了解服务的性能、稳定性和调用关系。例如，指标可以帮助监控服务的响应时间、吞吐量等关键指标；日志可以记录服务的运行过程和异常信息；链路追踪则可以展示服务之间的调用链路和依赖关系。这些信息对于故障排查、性能优化和系统维护都至关重要。

在Istio中，流量镜像和可观测性通常是协同工作的。通过流量镜像，运维人员可以将流量复制到测试环境，并利用Istio的可观测性特性对测试环境进行监控和分析。这样，他们可以在不影响生产环境的情况下，对服务进行验证和优化。同时，通过收集和分析生产环境中的遥测数据，运维人员可以及时发现和解决潜在的性能问题和故障。

总之，Istio的流量镜像和可观测性特性为微服务架构提供了强大的支持和保障。它们不仅提升了服务的可靠性和稳定性，还降低了运维的复杂性和风险。随着微服务架构的广泛应用和发展，这些特性将越来越成为企业构建高质量、高可靠性服务的关键技术。

# k8s中容器如何共享宿主机的解析

在 Kubernetes 中，容器可以通过与宿主机共享网络命名空间来共享宿主机的解析。当容器与宿主机共享网络命名空间时，它们将共享相同的网络配置，包括解析配置。

一种常见的方式是使用 Kubernetes 中的 hostNetwork 属性。当将 hostNetwork 设置为 true 时，Pod 将使用宿主机的网络命名空间，这意味着容器将与宿主机共享网络栈和解析配置。例如：

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  hostNetwork: true
  containers:
  - name: mycontainer
    image: nginx

yaml复制代码
```

在这个示例中，mypod Pod 中的 mycontainer 容器将与宿主机共享网络命名空间，包括宿主机的解析配置。这意味着容器可以访问宿主机上的 DNS 解析服务以解析域名。

需要注意的是，使用 hostNetwork 可能会降低容器的隔离性，并增加安全风险，因为容器将直接访问宿主机的网络。因此，在使用 hostNetwork 时需要谨慎考虑安全性和隔离性问题。

# label和annotation 的区别是啥。

Annotations和Labels之间的实际区别实际上很简单:Labels（标签）适用于 Kubernetes，而Annotations（注解）适用于人类。

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1712914439910-deb7dd1d-ae0d-4907-9f2f-8f30292ac21b.png)

# helm

小结

1、Helm 让我们能够像 apt 管理 deb 包那样安装、部署、升级和删除容器化应用。

2、Helm 由客户端和 Tiller 服务器组成。客户端负责管理 chart，服务器负责管理 release。

3、chart 是 Helm 的应用打包格式，它由一组文件和目录构成。其中最重要的是模板，模板中定义了 Kubernetes 各类资源的配置信息，Helm 在部署时通过 values.yaml 实例化模板。

4、Helm 允许用户开发自己的 chart，并为用户提供了调试工具。用户可以搭建自己的 chart 仓库，在团队中共享 chart。

5、Helm 帮助用户在 Kubernetes 上高效地运行和管理微服务架构应用，Helm 非常重要。

# logpilot 

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1713937492722-ee5a29e9-9773-4e8f-bb83-19de95ee7291.png)

# K8S 直接删错 namespace  pod  会 删除不  ，该怎么处理

在Kubernetes中，如果你直接删除一个Namespace，该Namespace下的所有资源，包括Pods、Deployments、Services、ConfigMaps、Secrets等，都会被级联删除。这是因为在Kubernetes的设计中，Namespace是一个高级的分组机制，用来隔离不同的资源集合。当一个Namespace被删除时，该Namespace内所有的子资源都会被视为不再需要，并随之被清理。

因此，如果你不小心删除了一个错误的Namespace，那么该Namespace内所有的Pod也会被自动删除，且这一过程通常是不可逆的。所以在执行kubectl delete namespace <namespace-name>命令前，务必确认无误，以避免不必要的数据丢失。如果确实发生了误删，恢复数据可能需要依赖于备份或从其他副本（如果有的话）进行恢复。

# k8s 的QOS 服务级别 是什么

Kubernetes 中的 QoS（Quality of Service）服务级别是指为 Pod 分配资源的优先级和限制条件。它确保在资源紧张的情况下，高优先级的 Pod 能够获得足够的资源，而低优先级的 Pod 可能会被限制或驱逐。

Kubernetes 主要定义了三种 QoS 级别：

1. **Guaranteed**：Pod 中的每个容器都必须有内存和 CPU 的限制与请求，且值必须相等。如果一个容器只指明了 limit 而未设定 request，则 request 的值等于 limit 值。这种级别的 Pod 具有最高的优先级，它们会被优先分配资源，并且在资源不足时不会被驱逐。
2. **Burstable**：Pod 中至少有一个容器的内存或 CPU 请求与限制不满足 Guaranteed 级别要求，即内存或 CPU 的值设置不同。这种级别的 Pod 具有中等优先级，它们会在 Guaranteed 级别 Pod 之后获得资源分配，但在资源紧张时可能会被限制或驱逐。
3. **BestEffort**：容器没有任何内存或 CPU 的限制或请求。这种级别的 Pod 优先级最低，它们会在其他级别 Pod 之后获得资源分配，并且在资源紧张时最容易被限制或驱逐。

通过设置 QoS 级别，Kubernetes 可以根据 Pod 的重要性和资源需求，合理地分配和管理资源，以确保整个系统的稳定性和可靠性。同时，QoS 级别也可以帮助用户更好地控制和优化应用程序的性能。

## K8S 产线下节点的流程是什么 

## K8S 产线上节点的流程是什么 

## 在K8S 中POD 的根容器是什么什么 ，它的作用是什么 

## 一旦产线 上节点出现 POD 的 网络地址 回环 该如何处理 

## 如果在上节点了时候，我把 csr 删掉了 ，该如何处理

## 

## pod  长期不更新，导致pod 过于集中，该如何处理

## 

## 一个K8S 的 无头服务用在哪里

# 在k8s集群外部，突然之间无法访问到pod,请说一下你的排查思路？

 我会根据业务的数据流走向逐一排查问题，比如访问k8s的pod前面有个lb是否是否能够到达lb,如果lb能够访问且处于正常工作状态，再去检查后端的ingress,若正常在检查

svc,查看svc的ep资源列表是否有pod列表，如 果没有，就说明可能pod是处于不健康状态，查看pod的运行情况。

# 如何扩容和缩容k8s集群

以kubeadm为例，扩容节点join命令即可，缩

如何扩容和缩容k8s集群 容，先将节点标记为不可调度（drain),给节点打污点并立刻驱逐ds资源的pod,删除节点，给删除的节点重新安装操作系统。

# jenkins如何集成k8s集群

开发人员将代码推送到gitlab.jenkins通过

 webhook拉取aitlab代码到本地并编译镜像推送

到harbor仓库，然后更新k8s的deployment镜像

即可。

# 如何收集kubernetes集群日志

1.业务容器直接将日志写入kafka集群，用户从

kafka集群拉取数据写入es,并通过kibana展示

数据。

2.部署是一个边车（sidecar)容器和业务容

 器共享网络空间和存储空间，通过共享存储将日

志采集并推送到kafka集群，用户从kafka集群拉

取数据写入es,并通过kibana展示数据。

3.使用damnset资源在每个节点部署一个filebeat日志采

集工具，将所有的日志采集并写入到kafka集群

或者es集群。

![img](https://cdn.nlark.com/yuque/0/2024/png/35538885/1717073586834-44d56142-31ef-4e32-b72f-16678a8ae893.png?x-oss-process=image%2Fformat%2Cwebp)

# k8s集群卡顿可能是什么问题

k8s集群卡顿  master集群有问题卡 负载高  etcd集群有点问题  可能打到etcd 某个节点 读写io压力巨大  然后正好访问该节点 访问慢 或者拿不到  返回数据整个链路都慢了

在Kubernetes（k8s）中，worker节点加入集群的过程通常涉及一个称为“Bootstrap”的阶段，这个阶段主要负责初始化worker节点并与集群的control plane建立安全的通信。Bootstrap阶段的具体步骤大致如下：

1. **生成Bootstrap Token**：

- - 在集群的master节点上，使用kubeadm工具生成一个Bootstrap Token。这个Token是一个短期的认证凭据，允许新节点在加入集群时进行身份验证，并且可以携带角色绑定信息以限制其权限。

1. **创建Bootstrap ConfigMap**：

- - kubeadm init或相关命令会创建一个Bootstrap ConfigMap，其中包含了worker节点加入集群所需的配置信息，比如API服务器地址、CA证书、Bootstrap Token等。这个ConfigMap通常存储在kube-system命名空间中。

1. **分发Bootstrap配置**：

- - 在worker节点上，需要获取Bootstrap ConfigMap的内容。这通常通过将kubelet配置文件（如bootstrap.kubeconfig）复制到worker节点实现。该文件包含了连接到API服务器所需的认证信息和TLS密钥。

1. **使用Bootstrap Token加入集群**：

- - 在worker节点上执行kubeadm join命令，该命令需要Bootstrap Token作为参数。kubeadm join命令会读取Bootstrap ConfigMap中的信息，并使用Token进行身份验证，然后下载必要的组件（如kubelet配置、Flannel或Calico网络插件配置等）。

1. **TLS Bootstrapping（可选）**：

- - 如果启用了TLS Bootstrapping机制，worker节点会在加入集群时请求一个新的节点证书。Bootstrap Token除了用于认证外，还允许节点发起证书签名请求(CSR)，master节点上的kube-controller-manager会自动审批这些请求并签发证书给worker节点。这样，worker节点就能以加密的方式与API服务器通信。

1. **安装并启动kubelet和kube-proxy**：

- - kubeadm join命令还会确保kubelet和kube-proxy服务在worker节点上被正确安装并启动，它们是worker节点上的核心组件，分别负责运行Pods和维护网络规则。

1. **完成Bootstrap并加入集群**：

- - 一旦kubelet成功连接到API服务器并完成了自我配置，worker节点就被认为已成功加入集群，可以开始接收并运行由调度器分配的Pods。

整个Bootstrap过程是自动化且安全的，确保了worker节点能够安全地加入集群，并准备就绪以参与负载分担。