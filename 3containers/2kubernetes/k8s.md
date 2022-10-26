教程：https://kuboard.cn/learning/



# 概况

### Kubernetes的概念

Cluster、Master、Node

Pod：Pod是kubernets的最小工作单元。每个pod包含一个或多个容器。pod中的容器会做为一个整体被master调度到一个node上运行

Controller：管理 Pod

​	Deployment：是最常用的 Controller

​	DaemonSet、StatefuleSet、Job

Service：Service 为 Pod 提供了负载均衡。Kubernetes 运行容器（Pod）与访问容器（Pod）这两项任务分别由 Controller 和 Service 执行



### Kubernetes的功能

服务发现和负载均衡

存储编排

自动发布和回滚

自愈

密钥及配置管理



### Kubernetes的边界

不限制应用程序的类型

不提供应用程序级别的服务

不提供或者限定配置语言	可选的有 helm/kustomize/kubectl/kubernetes dashboard/kuboard/octant/k9s

不部署源码、不编译或构建应用程序	可选的有Jenkins / Gitlab Runner / docker registry / harbour

不限定日志、监控、报警的解决方案	可选的有 ELK / Prometheus / Graphana / Pinpoint / Skywalking 

不提供或限定任何机器的配置、维护、管理或自愈的系统	可选的组件有 puppet、ansible、open stack



### Kubernetes组件

#### Master组件

Master组件可以运行于集群中的任何机器上。但是，为了简洁性，通常在同一台机器上运行所有的 master 组件，且不在此机器上运行用户的容器



##### kube-apiserver

此 master 组件提供 Kubernetes API。这是Kubernetes控制平台的前端（front-end），可以水平扩展（通过部署更多的实例以达到性能要求）。kubectl / kubernetes dashboard / kuboard 等Kubernetes管理工具就是通过 kubernetes API 实现对Kubernetes 集群的管理



##### etcd

支持一致性和高可用的名值对存储组件，Kubernetes集群的所有配置信息都存储在 etcd 中。请确保您 [备份](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)[ ](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster) 了 etcd 的数据。关于 etcd 的更多信息，可参考 [etcd 官方文档](https://etcd.io/docs/)



##### kube-scheduler

此 master 组件监控所有新创建尚未分配到节点上的 Pod，并且自动选择为 Pod 选择一个合适的节点去运行



##### kube-controller-manager

此 master 组件运行了所有的控制器

- 节点控制器： 负责监听节点停机的事件并作出对应响应
- 副本控制器： 负责为集群中每一个 副本控制器对象（Replication Controller Object）维护期望的 Pod 副本数
- 端点（Endpoints）控制器：负责为端点对象（Endpoints Object，连接 Service 和 Pod）赋值
- Service Account & Token控制器： 负责为新的名称空间创建 default Service Account 以及 API Access Token



#### Node 组件

Node 组件运行在每一个节点上（包括 master 节点和 worker 节点），负责维护运行中的 Pod 并提供 Kubernetes 运行时环境



##### kubelet

此组件是运行在每一个集群节点上的代理程序。它确保 Pod 中的容器处于运行状态



##### kube-proxy

[kube-proxy](https://kuboard.cn/learning/k8s-intermediate/service/service-details.html#虚拟-ip-和服务代理) 是一个网络代理程序，运行在集群中的每一个节点上，是实现 Kubernetes Service 概念的重要部分



##### 容器引擎

容器引擎负责运行容器。Kubernetes支持多宗容器引擎：[Docker](http://www.docker.com/)[ ](http://www.docker.com/)、[containerd](https://containerd.io/)[ ](https://containerd.io/)



#### Addons

Addons 使用 Kubernetes 资源（DaemonSet、Deployment等）实现集群的功能特性。由于他们提供集群级别的功能特性，addons使用到的Kubernetes资源都放置在 `kube-system` 名称空间下。



##### DNS



##### Web UI（Dashboard）

[Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)[ ](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) 是一个Kubernetes集群的 Web 管理界面。用户可以通过该界面管理集群。



#####  Kuboard

[Kuboard](https://kuboard.cn/install/install-dashboard.html) 是一款基于Kubernetes的微服务管理界面，相较于 Dashboard，Kuboard 强调

- 无需手工编写 YAML 文件
- 微服务参考架构
- 上下文相关的监控
- 场景化的设计 
  - 导出配置
  - 导入配置



##### ContainerResource Monitoring



#####  Cluster-level Logging



# 基础

**Master 负责管理集群** 负责协调集群中的所有活动，例如调度应用程序，维护应用程序的状态，扩展和更新应用程序

**Worker节点(即图中的Node)是VM(虚拟机)或物理计算机，充当k8s集群中的工作计算机。** 每个Worker节点都有一个Kubelet，它管理一个Worker节点并与负责与Master节点通信。该Worker节点还应具有用于处理容器操作的工具，例如Docker



一个deployment，对应一个service ，部署多个pod，每个pod可以有多个容器，对应多个image。

版本更新是pod的变化（通过该变deploymen实现）

对象是k8s的实例，包括一些核心的对象和创建的对象，deployment是对象的一个，service也是，可以同过yaml文件的形式创建



准备：按照官网安装了k8s与kuboard



### 部署Deployment



Deployment 处于 master 节点上，通过发布 Deployment，master 节点会选择合适的 worker 节点创建 Container，Container 会被包含在 Pod 里



在 k8s 集群中发布 Deployment 后（在master节点）

Deployment 将指示 k8s 如何创建和更新应用程序（docker image）的实例（docker container），

master 节点将应用程序实例（被Pod包装的 ）调度到集群中的具体的节点上



创建应用程序实例后，Kubernetes Deployment Controller 会持续监控这些实例。如果运行实例的 worker 
节点关机或被删除，则 Kubernetes Deployment Controller 将在群集中资源最优的另一个 worker 
节点上重新创建一个新的实例



#### 使用 kubectl部署

编写yaml文件

```bash
apiVersion: apps/v1	#与k8s集群版本有关，使用 kubectl api-versions 即可查看当前集群支持的版本
kind: Deployment	#该配置的类型，我们使用的是 Deployment
metadata:	        #译名为元数据，即 Deployment 的一些基本属性和信息
  name: nginx-deployment	#Deployment 的名称
  labels:	    #标签，可以灵活定位一个或多个资源，其中key和value均可自定义，可以定义多组，目前不需要理解
    app: nginx	#为该Deployment设置key为app，value为nginx的标签
spec:	        #这是关于该Deployment的描述，可以理解为你期待该Deployment在k8s中如何使用
  replicas: 1	#使用该Deployment创建一个应用程序实例
  selector:	    #标签选择器，与上面的标签共同作用，目前不需要理解
    matchLabels: #选择包含标签app:nginx的资源
      app: nginx
  template:	    #这是选择或创建的Pod的模板
    metadata:	#Pod的元数据
      labels:	#Pod的标签，上面的selector即选择包含标签app:nginx的Pod
        app: nginx
    spec:	    #期望Pod实现的功能（即在pod中部署）
      containers:	#生成container，与docker中的container是同一种
      - name: nginx	#container的名称
        image: nginx:1.7.9	#使用镜像nginx:1.7.9创建container，该container默认80端口可访问

```

应用 YAML 文件

```shell
kubectl apply -f nginx-deployment.yaml
```

查看结果

```shell
kubectl get deployments
kubectl get pods
```

删除

```sh
kubectl delete -f https://kuboard.cn/statics/learning/obj/deployment.yaml
```



#### 使用 Kuboard部署



### 查看pods/Nodes

#### pods

Pod 容器组** 是一个k8s中一个抽象的概念，用于存放一组 container（可包含一个或多个 container 容器，即图上正方体)，以及这些 container （容器）的一些共享资源。这些资源包括：

- 共享存储，称为卷(Volumes)
- 网络，每个 Pod（容器组）在集群中有个唯一的 IP，pod（容器组）中的 container（容器）共享该IP地址
- container（容器）的基本信息，例如容器的镜像版本，对外暴露的端口等





Pod是短暂的，Kubernetes支持 ***卷*** 的概念，因此可以使用持久化的卷类型



每个Pod都与运行它的 worker 节点（Node）绑定，并保持在那里直到终止或被删除。如果节点（Node）发生故障，则会在群集中的其他可用节点（Node）上运行相同的 Pod（从同样的镜像创建 Container，使用同样的配置，IP 地址不同，Pod 名字不同）



#### Node

每个 Kubernetes Node（节点）至少运行：

- Kubelet，负责 master 节点和 worker 节点之间通信的进程；管理 Pod（容器组）和 Pod（容器组）内运行的 Container（容器）。
- 容器运行环境（如Docker）负责下载镜像、创建和运行容器等。



#### 使用kubectl排除故障

```sh
kubectl get deployments
kubectl get pods
kubectl get nodes

# 查看所有名称空间的 Deployment
kubectl get deployments -A

# 查看 kube-system 名称空间的 Deployment
kubectl get deployments -n kube-system
```

```shell
#一般资源 后面加 get那得到的名称
kubectl describe pod nginx-deployment-59bd9cff8-vkxtt

kubectl describe deployment nginx
kubectl describe deployment nginx-deployment
```

```shell
#pod日志
kubectl logs -f nginx-deployment-59bd9cff8-vkxtt
```

```shell
#在pod中的容器环境内执行命令
#kubectl exec Pod名称 操作命令
kubectl exec -it nginx-deployment-59bd9cff8-vkxtt /bin/bash
```



#### 使用kuboard排除故障



### 发布服务 Service

由于 Kubernetes 集群中每个 Pod（容器组）都有一个唯一的 IP 地址（即使是同一个 Node 上的不同 Pod），我们需要一种机制，为前端系统屏蔽后端系统的 Pod（容器组）在销毁、创建过程中所带来的 IP 地址的变化



Service是一个抽象层，它通过 LabelSelector 选择了一组 Pod（容器组），把这些 Pod 同一用一个IP封装，并支持负载均衡。



Deployment B 含有 LabelSelector 为 app=B

通过 Deployment B 创建的 Pod 包含标签为 app=B

在master节点的Service B 通过标签选择器 app=B 选择可以路由各个节点的 Pod



在创建Service的时候，通过设置配置文件中的 spec.type 字段的值，可以以不同方式向外部暴露应用程序：

- **ClusterIP**（默认）

  在群集中的内部IP上公布服务，这种方式的 Service（服务）只在集群内部可以访问到

- **NodePort**

  使用 NAT 在集群中每个的同一端口上公布服务。这种方式下，可以通过访问集群中任意节点+端口号的方式访问服务 `<NodeIP>:<NodePort>`。此时 ClusterIP 的访问方式仍然可用。

Labels（标签）可以在创建 Kubernetes 对象时附加上去，也可以在创建之后再附加上去。任何时候都可以修改一个 Kubernetes 对象的 Labels（标签）



#### 使用 kubectl部署

确定已经有一个app标签的deployment，并部署了pod

```sh
vim nginx-service.yaml
```

```shell
apiVersion: v1
kind: Service
metadata:
  name: nginx-service	#Service 的名称
  labels:     	#Service 自己的标签
    app: nginx	#为该 Service 设置 key 为 app，value 为 nginx 的标签
spec:	    #这是关于该 Service 的定义，描述了 Service 如何选择 Pod，如何被访问
  selector:	    #标签选择器
    app: nginx	#选择包含标签 app:nginx 的 Pod
  ports:
  - name: nginx-port	#端口的名字
    protocol: TCP	    #协议类型 TCP/UDP
    port: 80	        #集群内的其他容器组可通过 80 端口访问 Service
    nodePort: 32600   #通过任意节点的 32600 端口访问 Service
    targetPort: 80	#将请求转发到匹配 Pod 的 80 端口
  type: NodePort	#Serive的类型，ClusterIP/NodePort/LoaderBalancer
```

```sh
kubectl apply -f nginx-service.yaml
```

```sh
kubectl get services -o wide
```



#### 使用 Kuboard部署



### Scaling（伸缩）应用程序

**伸缩** 的实现可以通过更改 nginx-deployment.yaml 文件中部署的 replicas（副本数）来完成

```
spec:
  replicas: 2    #使用该Deployment创建两个应用程序实例
```

```sh
kubectl apply -f nginx-deployment.yaml
```

```sh
watch kubectl get pods -o wide
```

### 执行滚动更新(Rolling Update)

**Rolling Update滚动更新** 通过使用新版本的 Pod 逐步替代旧版本的 Pod 来实现 Deployment 的更新，从而实现零停机。新的 Pod 将在具有可用资源的 Node（节点）上进行调度



更新完 Deployment 部署文件中的镜像版本后，master 节点选择了一个 worker 节点，并根据新的镜像版本创建 Pod。同时master 节点选择一个旧版本的 Pod 将其移除。Service A 将新 Pod 纳入到负载均衡中，将旧Pod移除

重复以上步骤直到全部更新

#### 使用kubectl更新 

修改 nginx-deployment.yaml的image

```sh
kubectl apply -f nginx-deployment.yaml
```

```sh
watch kubectl get pods -l app=nginx
```



# 架构

### 节点

#### 节点状态

节点的状态包含如下信息：

- Addresses
- Conditions
- Capacity and Allocatable
- Info

```sh
kubectl get nodes -o wide
kubectl describe node <your-node-name>
```

https://kuboard.cn/learning/k8s-bg/architecture/nodes.html#conditions

#### 节点管理

添加节点、节点控制器、节点自注册、手动管理节点、

https://kuboard.cn/learning/k8s-bg/architecture/nodes-mgmt.html#%E8%8A%82%E7%82%B9%E6%8E%A7%E5%88%B6%E5%99%A8%EF%BC%88node-controller%EF%BC%89



### Master-Node之间的通信

https://kuboard.cn/learning/k8s-bg/architecture/com.html



### 控制器

在 Kubernetes 中，每个控制器至少追踪一种类型的资源。这些资源对象中有一个 `spec` 字段代表了目标状态。资源对象对应的控制器负责不断地将当前状态调整到目标状态。

理论上，控制器可以自己直接执行调整动作，然而，在Kubernetes 中，更普遍的做法是，控制器发送消息到 API Server，而不是直接自己执行调整动作

#### 通过APIServer进行控制

#### 直接控制

某些特殊的控制器需要对集群外部的东西做调整。例如，您想用一个控制器确保集群中有足够的节点，此时控制器需要调用云供应商的接口以创建新的节点或移除旧的节点



# 高级操作

### Kubernetes对象介绍

Kubernetes对象指的是Kubernetes系统的持久化实体，所有这些对象合起来，代表了你集群的实际情况。常规的应用里，我们把应用程序的数据存储在数据库中，Kubernetes将其数据以Kubernetes对象的形式通过
api server存储在 etcd 中。具体来说，这些数据（Kubernetes对象）描述了：

- 集群中运行了哪些容器化应用程序（以及在哪个节点上运行）
- 集群中对应用程序可用的资源
- 应用程序相关的策略定义，例如，重启策略、升级策略、容错策略
- 其他Kubernetes管理应用程序时所需要的信息
- 

一个Kubernetes对象代表着用户的一个意图（a record of intent），一旦您创建了一个Kubernetes对象，Kubernetes将持续工作，以尽量实现此用户的意图。创建一个 Kubernetes对象，就是告诉Kubernetes，您需要的集群中的工作负载是什么（集群的 **目标状态**）



操作 Kubernetes 对象（创建、修改、删除）的方法主要有：

- [kubectl](https://kuboard.cn/install/install-kubectl.html) 命令行工具
- [kuboard](https://kuboard.cn/install/install-k8s-dashboard.html) 图形界面工具

kubectl、kuboard 最终都通过调用 [kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)[ ](https://kubernetes.io/docs/concepts/overview/kubernetes-api/) 来实现对 Kubernetes 对象的操作。您也可以直接在自己的程序中调用 Kubernetes API，此时您可能要有用到 用到 [Client Libraries](https://kubernetes.io/docs/reference/using-api/client-libraries/)



每一个 Kubernetes 对象都包含了两个重要的字段：

- `spec` 必须由您来提供，描述了您对该对象所期望的 **目标状态**
- `status` 只能由 Kubernetes 系统来修改，描述了该对象在 Kubernetes 系统中的 **实际状态**



不同类型的 Kubernetes，其 `spec` 对象的格式不同（含有不同的内嵌字段），通过 [API 手册](https://kubernetes.io/docs/reference/#api-reference)[ ](https://kubernetes.io/docs/reference/#api-reference) 可以查看 Kubernetes 对象的字段和描述。例如，假设您想了解 Pod 的 `spec` 定义，可以在 [这里](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/#podspec-v1-core)[ ](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/#podspec-v1-core)找到，Deployment 的 `spec` 定义可以在 [这里](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/#deploymentspec-v1-apps)[ ](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/#deploymentspec-v1-apps) 找到。

### 管理Kubernetes对象

kubectl 命令行工具支持多种途径以创建和管理 Kubernetes 对象。本文档描述了3种不同的方式。更多的细节，请参考 [Kubectl book](https://kubectl.docs.kubernetes.io/)。同一个Kubernetes对象应该只使用一种方式管理，否则可能会出现不可预期的结果

| 管理方式         | 操作对象                     | 推荐的环境 | 参与编辑的人数 | 学习曲线 |
| ---------------- | ---------------------------- | ---------- | -------------- | -------- |
| 指令性的命令行   | Kubernetes对象               | 开发环境   | 1+             | 最低     |
| 指令性的对象配置 | 单个 yaml 文件               | 生产环境   | 1              | 适中     |
| 声明式的对象配置 | 包含多个 yaml 文件的多个目录 | 生产环境   | 1+             | 最高     |

#### 指令性的命令行

当使用指令性的命令行（imperative commands）时，用户通过向 `kubectl` 命令提供参数的方式，直接操作集群中的 Kubernetes 对象。此时，用户无需编写或修改 `.yaml` 文件。

这是在 Kubernetes 集群中执行一次性任务的一个简便的办法。由于这种方式直接修改 Kubernetes 对象，也就无法提供历史配置查看的功能

```sh
kubectl run nginx --image nginx
```

#### 指令性的对象配置

创建

```
kubectl create -f nginx.yaml
```

删除

```sh
kubectl delete -f nginx.yaml
```

替换  （yaml定义了对象及对应关系，相当于删除再创建）

```sh
kubectl replace -f nginx.yaml
```

#### 声明式的对象配置

当使用声明式的对象配置时，用户操作本地存储的Kubernetes对象配置文件，然而，在将文件传递给 kubectl 
命令时，并不指定具体的操作，由 kubectl 自动检查每一个对象的状态并自行决定是创建、更新、还是删除该对象。使用这种方法时，可以直接针对一个或多个文件目录进行操作

声明式对象配置使用 `patch` API接口，此时会把变化的内容更新进去，而不是使用 `replace` API接口，该接口替换整个 spec 信息

```sh
kubectl diff -R -f configs/
kubectl apply -R -f configs/
```

### 名称空间介绍

查看名称空间

`kubectl get namespaces`

设定namespace

`--namespace`

设置名称空间偏好

```sh
kubectl config set-context --current --namespace=<您的名称空间>
kubectl config view --minify | grep namespace:
```

大部分的 Kubernetes 对象（Pod、Service、Deployment、StatefulSet等）都必须在名称空间里。但是某些更低层级的对象，是不在任何名称空间中的，例如 [nodes](https://kuboard.cn/learning/k8s-bg/architecture/nodes.html)、[persistentVolumes](https://kuboard.cn/learning/k8s-intermediate/persistent/pv.html)、[storageClass](https://kuboard.cn/learning/k8s-intermediate/persistent/storage-class.html) 等

```sh
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```

### 使用名称空间

查看

```sh
kubectl get namespaces
kubectl describe namespaces <name>
```

创建

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <名称空间的名字>
```

```sh
kubectl create -f ./my-namespace.yaml
```

```sh
#或者
kubectl create namespace <名称空间的名字>
```

删除

```sh
kubectl delete namespaces <名称空间的名字>
```

切分集群

https://kuboard.cn/learning/k8s-intermediate/obj/namespace-op.html#%E4%BD%BF%E7%94%A8%E5%90%8D%E7%A7%B0%E7%A9%BA%E9%97%B4%E5%88%87%E5%88%86%E9%9B%86%E7%BE%A4





