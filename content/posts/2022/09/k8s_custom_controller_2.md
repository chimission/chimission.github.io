---
title: "如何在k8s中使用自定义资源（2）原理"
date: 2022-09-18T09:59:55+08:00
draft: false
author: "chimission"
categories: [""]
tags: ["k8s"]
archives: ['2022', '2022-09']
comments: false
description: ""
url: "/posts/k8s_custom_controller_2/"
---
>文本是系列文章第二篇，讲述如何在k8s中使用自定义资源
 <!--more-->
上一篇中我们创建了一个 CRD 和与之对应的 CR，但是只创建对象并没有什么用，还需要写一个 controller 来控制自定义资源的逻辑。k8s 已经内置了很多个 controller，由 controller-manager这个组件统一管理，例如 pod-controller，在我们向 k8s 提交一个pod对象的信息后，pod-controller 就会察觉到这一动作的发生，进而检查提交的 pod 的期望状态和 pod 的实际状态符不符合， 如果不符合就会调整到期望状态。
 
### k8s架构
![架构图](https://images.chimission.cn/blog/k8s_jiagou.png)  

在正式编写控制器之前让我们简单过一下 k8s 的控制器原理 ,这里我们主要关心 master 上的 controller-manager 组件和 apiserver组件， 它位于上图右侧主控节点。  apiserver 是整个集群统一入口，提供了http RESTFUl api 的方式，所有其他组件都通过 apiserver 进行交互。  
controller-manager 负责管理一系列的具体的 controller， 例如内置的 pod-controller，deployment-controller，job-controller等等，基本上在 k8s 中每一个内置的 wokeload 都会有一个相对应的 controller，用来管理自身的状态。

### 通信过程 精简版
首先让我们从整体上看下 controller 是如何从 k8s 那里获取关心的对象信息的  

![精简版](https://images.chimission.cn/blog/%E7%B2%BE%E7%AE%80%E7%89%88.png)  

首先左上角 k8s 讲集群中的对象信息序列化保存到 etcd 中，然后是 右边的 apiserver。  
上面我们提到过 apiserver 是整个集群统一入口，提供了http RESTFUl api 的方式，所有其他组件都通过 apiserver 进行交互，不例外的，所有 controller 都要和 apiserver 通信， 然后由 apiserver 去 etcd 中才可以获取到内部对象信息。 
可以看到， controller 和 apiserver 中间隔着 clinet-go， client-go 是 controller-manager 的组件，最初集成在代码里，后来被抽出来作为单独的sdk，给用户自定义controller提供统一的api，使得用户自定义controller和系统提供的 controller（ Deployment Controller等等）实现方式基本一致。简单来说，clinet-go 就是官方抽象出来的标准工具包，集成了许多实现好的功能， 方便用户编写自己的自定义 controller。后续在写代码时会使用到它。

### 通信过程 详细版  

![详细版](https://images.chimission.cn/blog/%E8%AF%A6%E7%BB%86%E7%89%88.jpeg)

蓝色的是 client-go 已经提供好的功能，黄色的是用户需要自己写代码实现的地方。

#### informer
client-go 中的 api-server，负责各组件的数据传递， 和用户代码和client-go代码的连接
#### reflector
Reflector 用于监控（Watch）指定的 Kubernetes 资源，当监控的资源发生变化时，触发相应的变更事件，例如 Add 事件、Update 事件、Delete 事件，并将其资源对象存放到本地缓存 DeltaFIFO 中。
#### DeltaFIFO
DeltaFIFO 是一个生产者-消费者的队列，生产者是 Reflector，消费者是 Pop 函数，FIFO 是一个先进先出的队列，而 Delta 是一个资源对象存储，它可以保存资源对象的操作类型，例如 Add 操作类型、Update 操作类型、Delete 操作类型、Sync 操作类型等。
#### Indexer
Indexer 是 client-go 用来存储资源对象并自带索引功能的本地存储，Reflector 从 DeltaFIFO 中将消费出来的资源对象存储至 Indexer。Indexer 与 Etcd 集群中的数据保持完全一致。这样我们就可以很方便地从本地存储中读取相应的资源对象数据，而无须每次从远程 APIServer 中读取，以减轻服务器的压力。【几乎所有的controller都是通过本地缓存查询对象状态的】
#### ResourceEventHandlers
自己向client-go注册的回调函数，在命令式编程里，add、update、delete的回调函数基本一致，就是向 workqueue 队列里推入一个被操作对象的元素
#### workqueue
自己实现的队列， 功能和 DeltaFIFO 类似，用来保存事件对象，使用队列是为了防止自己写的代码处理速度跟不上client-go的发送速度，导致client-go卡顿。
#### Processitem/Handle Object
真正的自定义控制器的逻辑就在写在这里，命令式编程一般会从 workqueue 中读取对象key， 然后根据key从  Indexer 的cache 里获得被操作对象的状态， 对比时期状态和期望状态，不一样就改成一样的。

这里理论知识比较多，直接去查看源码显得有一定困难，我们可以用一个实际的示例来进行说明，比如现在我们删除一个 Pod，一个 Informer的执行流程是怎样的：

1. 首先初始化 Informer，Reflector 通过 List 接口获取所有的 Pod 对象

2. Reflector 拿到所有 Pod 后，将全部 Pod 放到 Store（本地缓存）中

3. 如果有人调用 Lister 的 List/Get 方法获取 Pod，那么 Lister 直接从 Store 中去拿数据

4. Informer 初始化完成后，Reflector 开始 Watch Pod 相关的事件

5. 此时如果我们删除 Pod1，那么 Reflector 会监听到这个事件，然后将这个事件发送到 DeltaFIFO 中

6. DeltaFIFO 首先先将这个事件存储在一个队列中，然后去操作 Store 中的数据，删除其中的 Pod1

7. DeltaFIFO 然后 Pop 这个事件到事件处理器（资源事件处理器）中进行处理

8. LocalStore 会周期性地把所有的 Pod 信息重新放回 DeltaFIFO 中去

### 结语
本文主要介绍了k8s 自定义controller的一些理论概念， 下一篇中我们会用go 实战编写一个controller来控制上一篇的CRD。
