---
title: "聊一聊 输入 kubectl apply 时，k8s在背后做了什么"
date: 2022-07-04T17:59:07+08:00
draft: false
author: "chimission"
categories: ["k8s"]
tags: ["k8s"]
archives: ['2022', '2022-07']
comments: false
description: ""
url: "/posts/k8s_create_pod/"
---
>介绍了一下k8s是如何通过命令行启动pod :-)  这是一篇旧文，迁移到此
 <!--more-->

如果我们想在`kubernetes`集群中创建3个相同`nginx`镜像的`pod`，官方推荐的做法是定义一个更高一级抽象的`deployment`, 通过这个`deployment`来统一管理一组pod，并且使用声明式命令`apply`代替 指令式命令`create`来创建`deployment`对象
配置文件`deployment.yml`如下, 来自`kubernetes`官网

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: nginx-deployment
spec:
selector:
matchLabels:
app: nginx
replicas: 2 # tells deployment to run 2 pods matching the template
template:
metadata:
labels:
app: nginx
spec:
containers:

- name: nginx
  image: nginx:1.14.2
  ports:
- containerPort: 80
```
在 `shell` 中我们敲下 `kubectl apply -f deployment.yml` 后，`k8s` 就会为我们在 `etcd` 中生成`deployment` 和 `pod` 对象并将 `pod` 通过一系列的算法自动调度到集群节点上启动，这一切都是 `k8s` 自动帮我们完成的，我们只需要提供一份配置文件即可。`k8s` 在背后究竟是怎么做到的呢，让我们一探究竟。本篇文章省略了很多技术细节，一方面这些展开讲述细节需要大量的篇幅，另一方面作者也没有完全掌握这些细节，所以我们会放在后续的系列文章中继续探讨。  

## 构造请求  

`k8s` 集群使用 `REST API`进行通信, 请求会以`HTTP`的方式发给 `kube-apiserver` 进行处理。当我们是在`shell` 中输入 `kubectl apply -f deployment.yml`时，本地的 `kubectl` 会解析 `yaml` 中的配置根据其内容构造相应对象的 `HTTP` 请求参数。首先 `kubectl`会检查有没有语法错误（比如创建不支持的资源或使用格式错误的镜像名称），出现错误后会直接返回不会发送到 `kube-apiserver` 以节省网络负载。通过检查后 `kubectl` 就会构造出 `HTTP` 请求发送给 `kube-apiserver`。

## 2 认证，鉴权，准入控制


现在请求已经发送给了 `kube-apiserver`，`kube-apiserver` 接下来会判断这个请求的发起者是否合法，即请求发起者对应的用户信息是否存储 `k8s` 集群中，此过程称为**认证** `Authentication`。`k8s` 提供了多种[认证方式](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authentication)，这里我们不做过多的讨论，如果认证没有通过则会直接返回失败的错误信息，通过了就会进入一步 **鉴权**。

虽然我们的身份已经得到了 `k8s `的认可，但是身份 `identity` 和许可 `permission` 并不是一个概念，就像`mysql`账号有的有读写权限而有的只有读取权限一样。此时 `kube-apiserver` 会检查用户的权限是否可以进行相应的操作，对应我们文章中的命令就是创建 `deployment` 的权限，这里 `k8s` 也提供了[多种方式](https://kubernetes.io/zh/docs/reference/access-authn-authz/authorization/)进行鉴权不再赘述。

好了 kube-apiserver 确认请求发起者有相应的权限，这样就可以执行创建 `deployment` 的动作了吗。 很“不幸”还有最后一步， **准入控制** 。`Kubernetes` 准入控制器是控制和强制使用集群的一种插件。我们可以把它看作是拦截（已认证）API 请求的拦截器，它可以更改请求对象，甚至完全拒绝请求。这是可以配置的插件，也就是说你通过这套机制自己开发一套插件部署在集群中来控制请求的行为。`k8s`官方提供了很多[“内置”的准入控制器](https://kubernetes.io/zh/docs/reference/access-authn-authz/admission-controllers/)。  

## 3 etcd

终于我们的请求被验证通过，`kube-apiserver` 会在 `etcd` （服务发现的后端，存储了集群的状态及其配置）中创建我们的`Deployment`对象， 创建过程中出现的任何错误都会被捕获，最后 `kube-apiserver` 会构造 `HTTP` 响应返回给客户端，我们在输入完命下回车之后看到的信息就是 `kubectl` 得到 `HTTP` 响应解析后的信息。注意此时我们部署的 `Deployment` 对象现在虽然保存在于 `etcd` 中，但是它还没有被部署到真正的 `Node` 上。  

# 4 控制循环 （Control loops）

接下来的步骤对于请求调用者来说都是异步执行的，因为请求的响应已经在上一步得到了。

我们已经创建了`Deployment`，但是并没有创建涉及 `Deployment` 所依赖的资源拓扑（此例子中就是`ReplicaSet` 和`Pod` ）这其实是k8s通过内置控制器 `Controller`自动帮我们创建的。

`Controller` 是一个用于将系统状态从`当前状态`调谐到`期望状态`的异步脚本。所有内置的 `Controller` 都通过组件 `kube-controller-manager` 并行运行，每种 `Controller` 都负责一种具体的控制流程。

比如我们本次使用到的 `Deployment Controller`

当`k8s`在`etcd`中新创建了一个`Deployment`对象， `Deployment Controller`会监听（ **ListAndWatch** ）到这个事件之后然后检查`Deployment`这个对象的期望状态，和实际状态作对比，比如这次检查到相关联的对象`ReplicaSet`（因为本质上 `Deployment` 是通过控制 `ReplicaSet` 来控制 `Pod` 的）没有被创建，`Deployment Controller` 就会创建关联的 `ReplicaSet` ，创建 `ReplicaSet` 之后 `Deployment Controller` 的并不会检查对应管理的`Pod`，这是`ReplicaSet Controller`的工作。

`ReplicaSet Controller` 和 `Deployment Controller` 工作类似，`ReplicaSet Controller` 监听的是`ReplicaSet` 这个对象， 当`ReplicaSet` 被创建时就会检查这个 `ReplicaSet` 对象对应的期望状态，创建  `Pod` 对象。

这里也可以看出`Deployment`并不是直接管理`Pod`，而是通过 `ReplicaSet`，即 `Deployment `管理`ReplicaSet`， `ReplicaSet`管理`Pod`。

实际上 `Control loops` 的细节有很多，包括 实现监听的**Informer**机制，内部工作队列 `WorkQueue`， 本地缓存等等，如果全部展开如要大量的篇幅，而且作者也并没有完全掌握内部细节，我会在后续系列文章再次总结。

而此时我们也只是在 `etcd` 中创建了 `Deployment，ReplicaSet`， `Pod` 这3个对象，还没有在实际 `Node` 中部署。

## 5 调度 （Scheduler）

接下来到了调度环节。

当所有的 `Controller` 正常运行后，`etcd` 中就会保存一个 `Deployment`、一个 `ReplicaSet` 和 三个 `Pod`， 并且可以通过 `kube-apiserver` 查看到。这时如果你在`shell`里 `get pod`查看刚才的`pod`状态 你会看到`Pending`状态（调度中，即它们还没有被调度到集群中合适的 Node 上）。

`k8s`是依靠`Scheduler`这个组件完成调度操作的。`Scheduler` 组件运行在集群控制平面上，工作方式与其他 `Controller` 相同：监听事件并调谐状态。具体来说， `Scheduler` 的作用是过滤 `PodSpec` 中 `NodeName` 字段为空的 `Pod` 并尝试将其调度到合适的节点。`Scheduler` 会经过一系列的比如资源限制（cpu，内存）等算法首先选出一批符合条件的 `Node`， 然后通过第二轮算法（列如负载均衡情况）给 `Node` 打分，将 `Pod` 调度最高分的 `Node` 上，调度器就会将`Pod`对象的`nodeName`字段的值，修改为上述`Node`的名字。度器对一个 `Pod` 调度成功，实际上就是将它的 `spec.nodeName` 字段填上调度结果的节点名字。

不可避免的，这里也包含了很多细节，我们也会在后续文章中详细讨论。  

## 6 Kubelet

终于到了激动人心的真正的容器启动环节。

我们来总结一下已经完成的任务：

1. `HTTP` 请求通过了认证、授权和准入控制阶段；
2. 一个 `Deployment`、`ReplicaSet` 和三个 `Pod` 被持久化到 `etcd`；
3. 最后每个 `Pod` 都被调度到合适的节点。

到目前为止，所有的工作仅仅只是针对保存在 `etcd` 中的资源对象，接下来的步骤涉及到在工作节点之间运行具体的容器，这是分布式系统 `Kubernetes` 的关键因素。这些事情都是由 `Kubelet` 完成的。

在 `Kubernetes` 集群中，每个 `Node` 节点上都会启动一个 `Kubelet` 服务进程，该进程用于处理 `Scheduler` 下发到本节点的 `Pod` 并管理其生命周期。这意味着它将处理 `Pod` 与 `Container Runtime` 之间所有的转换逻辑，包括挂载卷、容器日志、垃圾回收等操作。

我们可以把 `Kubelet` 当成一种特殊的 `Controller`，它每隔 20 秒（可以自定义）向 `kube-apiserver` 查询 `Pod`，过滤 `NodeName` 与自身所在节点匹配的 `Pod` 列表。

当检测到新的 `pod` 对象还没有在 `Node` 上创建时，Kubelet进行一些前置操作，然后通过 `CRI（Container Runtime Interface）` 创建 `pause` 容器，通过 `CNI (Container Network Interface)` 为 `Pod` 设置网络，最后通过`CRI`拉取我们文件定义中的 `nginx` 镜像，创建并启动起来！

## 总结
![](https://images.chimission.cn/blog/k8s_create_pod.png)  
最终我们的集群上会运行三个容器，这三个容器可能分布在不同的Node上，而这一切只需要我们编写一份文章开头的yml配置文件，Amazing！