[TOC]



# 服务service

## 概述

pod是k8s的核心资源之一，k8s提供了各种controller来保障pod的数量满足期望，用户不必操心pod可用性问题。因此构成应用的一组pod数量会处于动态平衡，一个pod故障会有另一个新的pod创建，应用的pod的IP是在变化的，应用难以直接通过Pod IP来对外提供服务。未解决pod访问的问题，k8s提供了类似于反向代理、负载均衡的一个抽象资源——service。

service为应用后端的pod提供了一个统一的入口，并能将请求进行均衡负载，分发到后端的pod上。

对于service的请求来源可以分为两大类：集群内和集群外。k8s分别提供了不同类型type的service，以便用户灵活选用。先介绍两种常用的类型ClusterIP、NodePort

ClusterIP：通过集群的内部 IP 暴露服务，选择该值，服务只能够在集群内部可以访问，这也是默认的 ServiceType。

NodePort：通过每个 Node 上的 IP 和静态端口（NodePort）暴露服务。NodePort 服务会路由到 ClusterIP 服务，这个 ClusterIP 服务会自动创建。通过请求`<NodeIP>:<NodePort>`，可以从集群的外部访问一个 NodePort 服务。



![img](app-config.assets/20180710114742a7760bf5-ca14-4d56-b747-4ec876736d16.png)

​														集群内通过ClusterIP类型的service来访问应用pod



NodePort示例：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx-app
  template:
    metadata:
      labels:
        app: my-nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-app-svc
  labels:
    name: my-nginx-app
spec:
  selector:			  #	通过selector和一组pod绑定				
    app: my-nginx-app
  type: NodePort      #这里代表是NodePort类型的
  ports:
  - port: 80          # 这里的端口和clusterIP供内部访问, 如10.97.114.36:80,。
    targetPort: 80    # 端口一定要和container暴露出来的端口对应
    protocol: TCP
    nodePort: 32143   # 此端口供集群外部访问
```



另外两种type的service：

LoadBalancer：使用云提供商的负载局衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 NodePort 服务和 ClusterIP 服务。

ExternalName：将service映射为一个域名。需要kube-dns的支持。



### service 负载分发策略

service除了反向代理了后端pod，还提供了负载均衡的策略，有两种：

RoundRobin：轮询模式，即轮询将请求转发到后端的各个pod上（默认模式）；

SessionAffinity：基于客户端IP地址进行会话保持的模式，第一次客户端访问后端某个pod，之后的请求都转发到这个pod上。



### service的服务发现

通过service访问pod，而无需知道后端pod的IP，那么service的访问地址怎么暴露出去被他人发现呢。对于service的发现，k8s提供了两种方式：

1. 环境变量：当创建一个Pod的时候，kubelet会在该Pod中注入集群内所有Service的相关环境变量。需要注意的是，要想一个Pod中注入某个Service的环境变量，则必须Service要先比该Pod创建。这一方式很缺乏灵活性。

2. DNS：官方推荐的方式，使用集群插件DNS服务器。 DNS 服务器监视着创建新 Service 的 Kubernetes API，从而为每一个 Service 创建一组 DNS 记录。 如果整个集群的 DNS 一直被启用，那么所有的 Pod 应该能够自动对 Service 进行名称解析。

例如，有一个名称为 "my-service" 的 Service，它在 Kubernetes 集群中名为 "my-ns" 的 Namespace 中，为 "my-service.my-ns" 创建了一条 DNS 记录。 在名称为 "my-ns" 的 Namespace 中的 Pod 应该能够简单地通过名称查询找到 "my-service"。 在另一个 Namespace 中的 Pod 必须限定名称为 "my-service.my-ns"。 这些名称查询的结果是 Cluster IP。

有了kube-dns，k8s还提供了一种type的service：ExternalName。可以通过域名访问的service



## service的细节

### endpoint

endpoint是k8s集群中的一个资源对象，存储在etcd中，用来记录一个service对应的所有pod的访问地址。

service配置selector，endpoint controller才会自动创建对应的endpoint对象；否则，不会生成endpoint对象.

其控制器endpoint controller是k8s集群控制器的其中一个组件，负责生成和维护所有endpoint对象的控制器，监听service和对应pod的变化：

- 监听到service被删除，则删除和该service同名的endpoint对象
- 监听到新的service被创建，则根据新建service信息获取相关pod列表，然后创建对应endpoint对象
- 监听到service被更新，则根据更新后的service信息获取相关pod列表，然后更新对应endpoint对象
- 监听到pod事件，则更新对应的service的endpoint对象，将podIP记录到endpoint中



### 外部服务的service

前面提到的service都是对集群内某个namespace服务的代理。而在某些场景中，将另一个集群或Namespace中的服务作为服务的后端，甚至将一个外部数据库用为后端服务进行连接。

这时可以通过创建一个无Label Selector的Service实现，以及单独创建存放访问地址的endpoint，从而可以将service的后端定向到本namespace外面。

这个时候需要手动创建存放后端访问信息的endpoint，并将其与service绑定，而service不需要有selector。一个例子：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 1.2.3.4 # 手动添加后端服务的地址
    ports:
      - port: 9376
```



### Headless Service

pod访问地址事实上是存放在endpoint中，存放IP地址、端口是endpoint最为核心的作用。

有些资源的地址并不是动态的，比如etcd、master的三大组件，如果需要外界想要通过访问集群API来获取这些实体资源的地址，并自由选择代理、负载均衡的方案，那么在这里实际上并不需要service，而只关心endpoint中的信息。

对于这类特别的场景，k8s也提供了headless service——ClusterIP为None的service。

对这类 Service 并不会分配 Cluster IP，kube-proxy也不会处理它们，而且平台也不会为它们进行负载均衡和路由，与k8s解耦。用户便可不依赖service，自行选择后端资源的代理、负载均衡策略。

一个headless service例子，将etcd集群的地址通过endpoint形式放入集群，外界可以访问api-server进而发现etcd的地址：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: etcd-k8s
  namespace: kube-system
  labels:
    k8s-app: etcd
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: port
    port: 2379
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: etcd-k8s
  namespace: kube-system
  labels:
    k8s-app: etcd
subsets:
- addresses:
  - ip: 22.22.3.231
    nodeName: etcd01
  - ip: 22.22.3.232
    nodeName: etcd02
  - ip: 22.22.3.233
    nodeName: etcd03
  ports:
  - name: port
    port: 2379
    protocol: TCP
```



### kube-proxy

访问service有两种形式：k8s内部从pod到service的访问，和外部通过service暴露的node port到pod的访问。

两种访问service的途径实际上是通过kube-proxy这一个k8s组件实现的。

具体内容放在k8s网络部分，这里不展开。



### ingress

通常情况下，service和pod的IP仅可在集群内部访问。集群外部的请求需要通过负载均衡转发到service在Node上暴露的NodePort上，然后再由kube-proxy将其转发给相关的Pod。

而Ingress就是为外部进入集群的请求提供路由规则的集合，如下图所示

```
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
```

Ingress可以给service提供集群外部访问的URL、负载均衡、SSL终止、HTTP路由等。可以为服务提供七层负载均衡。

为了配置这些Ingress规则，集群管理员需要自行部署Ingress的控制器 ingress controller，用以监听Ingress和service的变化，并根据规则配置负载均衡并提供访问入口。对于ingress controller，常见的实现方案有：

[Ingress NGINX](https://github.com/kubernetes/ingress-nginx): Kubernetes 官方维护的方案

[Traefik](https://github.com/containous/traefik): 是一套开源的 HTTP 反向代理与负载均衡器，而它也支援了 Ingress



Ingress和kube-proxy将放于k8s网络部分展开



## 小结

service是后端的反向代理和负载均衡。k8s的Service定义了一个服务的访问入口地址，前端的应用通过这个入口地址访问其背后的一组由Pod副本组成的实例，来自外部的访问请求被负载均衡到后端的各个容器应用上。

Service与其后端Pod副本之间则是通过Label Selector来实现绑定的。

service针对不同的访问来源，提供了不同的类型type，主要分为集群内ClusterIP，集群外NodePort等。

后端应用的访问信息存放在endpoint中，service通过与endpoint绑定达到代理后端的目标。

endpoint扩展了service后端服务的位置，后端可以在非同个namespace外，甚至可以在集群外。

如果只关心endpoint中后端的访问信息，不需要service提供后端的代理服务，可以使用headless service将后端的代理策略交由开发人员灵活实现。



# App配置

## ConfigMap

应用程序的运行可能会依赖一些配置，而这些配置又是可能会随着需求产生变化的。如果我们的应用程序的应用镜像和配置不分离，如果修改了某些配置项，就要重新构建镜像，十分不灵活。ConfigMap资源可以实现应用和配置的分离，避免因为修改配置而重新构建镜像。

ConfigMap 用于保存配置数据的键值对，可以用来保存单个属性，也可以用来保存配置文件。一个yaml文件示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: adapter-config
  namespace: monitoring
  
# 配置数据 
data: 

  # key-value形式的配置
  file_path: "/etc/config.d/..."   
  
  # 文件式的配置
  config.yaml: |				   
    resourceRules:
    	# ...
    memory:
    	# ...
```

- 常见创建形式

```
# 从文件创建
kubectl create -f file_name
# 从目录创建
kubectl create configmap my_config --from-file=/home/tianshuai/tls-config -n monitoring
```



- 通过ConfigMap挂载配置

常见一种方式，使用volume挂载ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vol-test-pod
spec:
  containers:
  - name: db
    # ...
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: my_config
      # 挂载部分数据
      # configMap：
      # name: my_config
      # items:
      # - key: config.data.my_key
      #   path: item/key              # 相对于mountPath的路径
```



需要说明两点：

1、ConfigMap必须在Pod之前创建

2、ConfigMap是限定namespace的资源，因此不能跨namespace使用ConfigMap



## Secret

对于有些敏感数据配置，如密码等，k8s提供了Secret资源来挂载。和ConfigMap很相似

```yaml
# my_secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: grafana-datasources
  namespace: monitoring
type: Opaque
data:
  my_key: my_value
  password: MWYyZDFlMmU2N2Rm
  username: YWRtaW4=
```



- 常见创建方式

```
# kubectl create secret -h
kubectl create -f my_secrets.yml

kubectl create secret generic helloworld-tls \
  --from-file=key.pem \
  --from-file=cert.pem
```



- 常见使用方式

挂载进目录

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: db
  name: db
spec:
  containers:
  - name: db
    # ...
    volumeMounts:
    - name: secrets
      mountPath: "/etc/secrets"
      readOnly: true
    # ...
  volumes:
  - name: secrets
    secret:
      secretName: mysecret
```



导出为容器运行时环境变量

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: db
  name: db
spec:
  containers:
  - name: db
    # ...
	env:
    - name: WORDPRESS_DB_USER
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: username
    - name: WORDPRESS_DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
```

