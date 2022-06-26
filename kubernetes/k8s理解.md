
# 容器编排与kubernetes作业管理

## Pod

### 为什么需要Pod，Pod设计背后的考虑

1. 容器进程组

容器是单进程模型。并不是说容器只有一个进程，惹事只容器没有管理多个进程的能力。PID=1进程是容器进程本身，而其他进程都是PID=1进程的子进程。可是，用户编写的应用，并不能够像正常操作系统里的 init 进程或者 systemd 那样拥有进程管理的功能。比如，你的应用是一个 Java Web 程序（PID=1），然后你执行 docker exec 在后台启动了一个 Nginx 进程（PID=3）。可是，当这个 Nginx 进程异常退出的时候，你该怎么知道呢？这个进程退出后的垃圾收集工作，又应该由谁去做呢？

像这样容器间的紧密协作，我们可以称为“超亲密关系”。这些具有“超亲密关系”容器的典型特征包括但不限于：互相之间会发生直接的文件交换、使用 localhost 或者 Socket 文件进行本地通信、会发生非常频繁的远程调用、需要共享某些 Linux Namespace（比如，一个容器要加入另一个容器的 Network Namespace）等等。这也就意味着，并不是所有有“关系”的容器都属于同一个 Pod。比如，PHP 应用容器和 MySQL 虽然会发生访问关系，但并没有必要、也不应该部署在同一台机器上，它们更适合做成两个 Pod。

首先，关于 Pod 最重要的一个事实是：它只是一个逻辑概念。也就是说，Kubernetes 真正处理的，还是宿主机操作系统上 Linux 容器的 Namespace 和 Cgroups，而并不存在一个所谓的 Pod 的边界或者隔离环境。那么，Pod 又是怎么被“创建”出来的呢？答案是：Pod，其实是一组共享了某些资源的容器。具体的说：Pod 里的所有容器，共享的是同一个 Network Namespace，并且可以声明共享同一个 Volume（MNT默认不共享）

2. init 容器

完成Pod进程组间关系初始化，如设置共享Namespace，容器启动顺序等

3. 进程组例子

war包与web服务器；容器日志收集

4. Pod 才是真正扮演了虚拟机的角色

docker exec -it <container-name> bash, 是指在容器进程下起了一个 tty 子进程

凡是跟容器的 Linux Namespace 相关的属性，也一定是 Pod 级别的。这个原因也很容易理解：Pod 的设计，就是要让它里面的容器尽可能多地共享 Linux Namespace，仅保留必要的隔离和限制能力。这样，Pod 模拟出的效果，就跟虚拟机里程序间的关系非常类似了。

### Pod中容器共享所有的空间吗

但不要忘记，Pod 的另一个重要特性是，它的所有容器都共享同一个 Network Namespace。这就使得很多与 Pod 网络相关的配置和管理，也都可以交给 sidecar 完成，而完全无须干涉用户容器。这里最典型的例子莫过于 Istio 这个微服务治理项目了。

但其他的 PID namespace 等是不共享的

默认共享的是UTS（主机名）、IPC（进程间通信）、NET（网络）、USER（用户、用户组）

默认不共享的namespace空间：
MNT（mount挂载）、PID（进程ID）、cgroup（CGroup 根目录）

### Pod 生命周期

1. lifecycle 钩子

2. pod.status.phase

3. 容器健康

phase 为 running，但实际上服务不可用，需要使用

- 容器健康探针

探针作用类型 存活livenessProbe 和 就绪readinessProbe 启动StartupProbe

> kubelet 使用 livenessProbe 来确定什么时候要重启容器。 例如，存活探测器可以探测到应用死锁（应用程序在运行，但是无法继续执行后面的步骤）情况。 重启这种状态下的容器有助于提高应用的可用性，即使其中存在缺陷。

> kubelet 使用 readinessProbe 可以知道容器何时准备好接受请求流量，当一个 Pod 内的所有容器都就绪时，才能认为该 Pod 就绪。 这种信号的一个用途就是控制哪个 Pod 作为 Service 的后端。 若 Pod 尚未就绪，会被从 Service 的负载均衡器中剔除。

> kubelet 使用启动探测器来了解应用容器何时启动。 如果配置了这类探测器，你就可以控制容器在启动成功后再进行存活性和就绪态检查，确保这些存活、就绪探测器不会影响应用的启动。 启动探测器可以用于对慢启动容器进行存活性检测，避免它们在启动运行之前就被杀掉。



支持http探测 
执行命令command探测 
tcp探测端口

- 异常恢复restartPolicy

它是 Pod 的 Spec 部分的一个标准字段（pod.spec.restartPolicy），默认值是 Always，即：任何时候这个容器发生了异常，它一定会被重新创建，注意不是重启容器


### PodPreset（Pod 预设置）的功能 已经出现在了 v1.11 版本的 Kubernetes 中。

Pod yaml 文件默认值。只针对 Pod 资源生效，而不会改变 Deployment 资源定义

### Node 与 Pod 交互投射的特殊资源，projected volume

其实，Secret、ConfigMap，以及 Downward API 这三种 Projected Volume 定义的信息，大多还可以通过环境变量的方式出现在容器里。但是，通过环境变量获取这些信息的方式，不具备自动更新的能力。所以，一般情况下，我都建议你使用 Volume 文件的方式获取这些信息。

1. Secret；

是帮你把 Pod 想要访问的加密数据，存放到 Etcd 中。然后，你就可以通过在 Pod 的容器里挂载 Volume 的方式，访问到这些 Secret 里保存的信息了。

2. ConfigMap；

与 Secret 类似的是 ConfigMap，它与 Secret 的区别在于，ConfigMap 保存的是不需要加密的、应用所需的配置信息。而 ConfigMap 的用法几乎与 Secret 完全相同：你可以使用 kubectl create configmap 从文件或者目录创建 ConfigMap，也可以直接编写 ConfigMap 对象的 YAML 文件。

3. Downward API；

Downward API，它的作用是：让 Pod 里的容器能够直接获取到这个 Pod API 对象本身的信息。


    1. 使用fieldRef可以声明使用:
    spec.nodeName - 宿主机名字
    status.hostIP - 宿主机IP
    metadata.name - Pod的名字
    metadata.namespace - Pod的Namespace
    status.podIP - Pod的IP
    spec.serviceAccountName - Pod的Service Account的名字
    metadata.uid - Pod的UID
    metadata.labels['<KEY>'] - 指定<KEY>的Label值
    metadata.annotations['<KEY>'] - 指定<KEY>的Annotation值
    metadata.labels - Pod的所有Label
    metadata.annotations - Pod的所有Annotation

    2. 使用resourceFieldRef可以声明使用:
    容器的CPU limit
    容器的CPU request
    容器的memory limit
    容器的memory request

4. ServiceAccountToken


## 容器(进程组组Pod)编排

### Deployment 与 ReplicaSet

副本控制: RS -> Pod

滚动更新: Deployment -> RS -> Pod

![关系图](https://static001.geekbang.org/resource/image/bb/5d/bbc4560a053dee904e45ad66aac7145d.jpg)

### StatefulSet 有状态服务控制器

状态的抽象化后的分类：
    1. 拓扑状态。SS 通过 headless service 创建 DNS 记录解析到 POD。 保证域名后面解析 Pod 类型不变。另外主从，主备，一主多从结构 等拓扑结构
    2. 存储状态

### DaemonSet: Node 守护 Pod

DaemonSet 的主要作用，是让你在 Kubernetes 集群里，运行一个 Daemon Pod。 所以，这个 Pod 有如下三个特征：

    这个 Pod 运行在 Kubernetes 集群里的每一个节点（Node）上；

    每个节点上只有一个这样的 Pod 实例；多的Pod会被删掉，因此 DaemonSet 控制器直接操作 Pod，不需要可以动态调整Pod的能力，因此DaemonSet 不需要 RS。升级、回滚的版本控制能力，是通过 ControllerRevision 来实现（专门用来记录某种 Controller 对象的版本）。

    当有新的节点加入 Kubernetes 集群后，该 Pod 会自动地在新节点上被创建出来；而当旧节点被删除后，它上面的 Pod 也相应地会被回收掉。

这个机制听起来很简单，但 Daemon Pod 的意义确实是非常重要的。我随便给你列举几个例子：

    各种网络插件的 Agent 组件，都必须运行在每一个节点上，用来处理这个节点上的容器网络；

    各种存储插件的 Agent 组件，也必须运行在每一个节点上，用来在这个节点上挂载远程存储目录，操作容器的 Volume 目录；

    各种监控组件和日志组件，也必须运行在每一个节点上，负责这个节点上的监控信息和日志搜集。

更重要的是，跟其他编排对象不一样，DaemonSet 开始运行的时机，很多时候比整个 Kubernetes 集群出现的时机都要早。

在 Kubernetes 项目中，当一个节点的网络插件尚未安装时，这个节点就会被自动加上名为node.kubernetes.io/network-unavailable的“污点”。DaemonSet Pod 模板加上”污点容忍“而通过这样一个 Toleration，调度器在调度这个 Pod 的时候，就会忽略当前节点上的“污点”，从而成功地将网络插件的 Agent 组件调度到这台机器上启动起来。


### 离线任务 Job 与 CronJon


# kubernetes API 

patch API 比如合并 istio envoy 容器与业务容器到 Pod 定义中


# 容器网络

## service 到 Pod

1. service VIP.  service vip -> pod IP

2. sercie DNS. 

    DNS -> service vip -> pod ip
    DNS -> pod -> ip;    也就是 headless service

## service 与 ingress

service 后面是多个 Pod，属于同个服务的不同节点

ingress 后面则是多个 service，反向代理到多个服务的 service

ingress 和 Nginx 转发规则一样，工作在应用层（HTTP，gRPC）第七层，根据host、path来转发到不同服务；
而service是工作在第四层网络层（IP），根据IP端口路由到不同网络节点（网卡）

# 作业调度与资源管理

## 资源模型

资源分配都是对 container 而言，Pod 在k8s体系中是一组 container 的关系的逻辑概念

在 Kubernetes 中，像 CPU 这样的资源被称作“可压缩资源”（compressible resources）。它的典型特点是，当可压缩资源不足时，Pod 只会“饥饿”，但不会退出。

而像内存这样的资源，则被称作“不可压缩资源（incompressible resources）。当不可压缩资源不足时，Pod 就会因为 OOM（Out-Of-Memory）被内核杀掉。

CPU 设置的单位是“CPU 的个数”。比如，cpu=1 指的就是，这个 Pod 的 CPU 限额是 1 个 CPU。

当然，具体“1 个 CPU”在宿主机上如何解释，是 1 个 CPU 核心，还是 1 个 vCPU，还是 1 个 CPU 的超线程（Hyperthread），完全取决于宿主机的 CPU 实现方式。

Kubernetes 只负责保证 Pod 能够使用到“1 个 CPU”的计算能力。此外，Kubernetes 允许你将 CPU 限额设置为分数，比如在我们的例子里，CPU limits 的值就是 500m。所谓 500m，指的就是 500 millicpu，也就是 0.5 个 CPU 的意思。这样，这个 Pod 就会被分配到 1 个 CPU 一半的计算能力。当然，你也可以直接把这个配置写成 cpu=0.5。但在实际使用时，我还是推荐你使用 500m 的写法，毕竟这才是 Kubernetes 内部通用的 CPU 表示方式。

### request 与 limit

在调度的时候，kube-scheduler 只会按照 requests 的值进行计算。而在真正设置 Cgroups 限制的时候，kubelet 则会按照 limits 的值来进行设置。

Kubernetes 为 Pod 设置这样三种 QoS 类别，具体有什么作用呢？实际上，QoS 划分的主要应用场景，是当宿主机资源紧张的时候，kubelet 对 Pod 进行 Eviction（即资源回收）时需要用到的。

- Guaranteed
当 Pod 里的每一个 Container 都同时设置了cpu内存 requests 和 limits，并且 requests 和 limits 值相等的时候，这个 Pod 就属于 Guaranteed 类别；

- Burstable
不满足 Guaranteed 的条件，但至少有一个 Container 设置了 requests。那么这个 Pod 就会被划分到 Burstable 类别。

- BestEffort
一个 Pod 既没有设置 requests，也没有设置 limits，那么它的 QoS 类别就是 BestEffort。

QoS 影响的是Node资源不足时，Pod驱逐策略，优先驱逐（删除）低QoS的Pod


# 声明式 API ，kubernetes 编程

能力的关键，允许 Patch 操作

Istio 为例

envoy 容器，C++网络代理，运行在每一个 service mesh 面板 Pod 中。

### informer

25 | 深入解析声明式API（二）：编写自定义控制器: https://time.geekbang.org/column/article/42076

在 Kubernetes 项目中，一个 API 对象在 Etcd 里的完整资源路径，是由：Group（API 组）、Version（API 版本）和 Resource（API 资源类型）三个部分组成的。

所谓的 Informer，就是一个自带缓存和索引机制，可以触发 Handler 的客户端库。

这个本地缓存在 Kubernetes 中一般被称为 Store，索引一般被称为 Index。

Informer 使用了 Reflector 包，它是一个可以通过 ListAndWatch 机制获取并监视 API 对象变化的客户端封装。
Reflector 和 Informer 之间，用到了一个“增量先进先出队列”(delta FIFO queue)进行协同。而 Informer 与你要编写的控制循环之间，则使用了一个工作队列(work queue)来进行协同。

![](https://static001.geekbang.org/resource/image/32/c3/32e545dcd4664a3f36e95af83b571ec3.png)

在实际应用中，除了控制循环之外的所有代码，实际上都是 Kubernetes 为你自动生成的，即：pkg/client/{informers, listers, clientset}里的内容。

### Operator 与 自定义controller

operator 的工作原理，实际上是利用了 Kubernetes 的自定义 API 资源（CRD），来描述我们想要部署的“有状态应用”；

然后在自定义控制器里，根据自定义 API 对象的变化，来完成具体的部署和运维工作。所以，编写一个 Etcd Operator，与我们前面编写一个自定义控制器的过程，没什么不同。

根据一些 Kubernetes 官方文档介绍， Operator 是用来扩展 Kubernetes，用于管理应用程序和组件的。就我个人的理解来说，Operator 是一种特别的 Controller，也就是说本质上 Operator 和 Controller 都是一样的，都是基于 Kubernetes 的资源和控制器概念之上构建，但是不同之处在于，Operator 包含了应用程序特定的领域知识，其实也可以说是封装了运维人员对于特定应用程序的运维经验。

例如通过 CRD 定义一个 CRD 来定义 Nginx 的配置不算 Operator，顶多算 Controller；但是，如何让 Nginx 保持高可用的 Controller，这就是个 Operator，因为它包含了保证 Nginx 高可用的业务逻辑，这封装了运维经验。此外，比较常见的还有管理应用状态的 Operator，例如 prometheus operator，它既要管理 promehteus 的配置，还需要管理 prometheus 自身的运行以及存储等。

# 存储

## PVC、PV 与 StorageClass

PVC 存储接口定义，PV 存储具体实现。Pod 对接 PVC，具体存储由 PV 决定

PVC 其实就是一种特殊的 Volume。只不过一个 PVC 具体是什么类型的 Volume，要在跟某个 PV 绑定之后才知道

- storageClass 
是 PV 的模板，自动创建PV，

一个大规模的 Kubernetes 集群里很可能有成千上万个 PVC，这就意味着运维人员必须得事先创建出成千上万个 PV。更麻烦的是，随着新的 PVC 不断被提交，运维人员就不得不继续添加新的、能满足条件的 PV，否则新的 Pod 就会因为 PVC 绑定不到 PV 而失败。在实际操作中，这几乎没办法靠人工做到。

具体地说，StorageClass 对象会定义如下两个部分内容：

- 第一，PV 的属性。比如，存储类型、Volume 的大小等等。

- 第二，创建这种 PV 需要用到的存储插件。

比如，Ceph 等等。有了这样两个信息之后，Kubernetes 就能够根据用户提交的 PVC，找到一个对应的 StorageClass 了。然后，Kubernetes 就会调用该 StorageClass 声明的存储插件，创建出需要的 PV。

# 容器监控与日志

## Prometheus

![系统构成](https://static001.geekbang.org/resource/image/2a/d3/2ada1ece66fcc81d704c2ba46f9dd7d3.png)

metrics: 监控指标

Pull模式的特点
1. 被监控方(k8s)提供一个server，并负责维护
2. 监控方(prometheus)控制采集频率

应用需要实现/metrics， 来响应Prometheus的数据采集请求。

## custom metrics 与 HPA

## 日志收集与管理

而对于一个容器来说，当应用把日志输出到 stdout 和 stderr 之后，容器项目在默认情况下就会把这些日志输出到宿主机上的一个 JSON 文件里。这样，你通过 kubectl logs 命令就可以看到这些容器的日志了。

Kubernetes 项目本身，主要为你推荐了2种日志方案。

- 第一种，在 Node 上部署 logging agent，将日志文件转发到后端存储里保存起来。

这里的核心就在于 logging agent ，它一般都会以 `DaemonSet` 的方式运行在节点上，然后将宿主机上的容器日志目录挂载进去，最后由 logging-agent 把日志转发出去。

当容器的日志只能输出到某些文件里的时候，我们可以通过一个 sidecar 容器把这些日志文件重新输出到 sidecar 的 stdout 和 stderr 上，缺点是会浪费磁盘，日志存了两份

第三种方案，就是通过一个 sidecar 容器，直接把应用的日志文件发送到远程存储里面去。

