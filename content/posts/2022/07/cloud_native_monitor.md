---
title: "通过钉钉机器人自动化报警运维"
date: 2022-07-25T21:09:46+08:00
draft: false
author: "chimission"
categories: ["monitor"]
tags: ["prometheus"]
archives: ['2022', '2022-07']
comments: false
description: ""
url: "/posts/cloud_native_monitor/"
---
>prometheus+alertmanager+dingtalk 实现运维监控报警
 <!--more-->

作者目前工作中所维护的服务大部分跑在k8s上，在监控方面使用了官方推荐的技术栈 `prometheus` 和 `grafana` 。 `prometheus` 负责收集数据，`grafana` 负责展示数据  

![监控面板](https://images.chimission.cn/blog/granfa.jpg)  

另外我们还集成了钉钉机器人，能够在 `prometheus` 采集到不正常的数据时能通过钉钉及时通知到对应的开发同事，例如下图， `prometheus` 在 `k8s` 的某个 `namespace` 下给定的内存已经快被使用完之前发送了一条消息到我们的报警群，提醒我同事该为它加内存资源了:-)。 


![报警](https://images.chimission.cn/blog/k8s_dingding2.jpg)   



接下来让我们用一个demo项目来演示这套系统是如何运作的 [项目地址](https://github.com/chimission/alertdemo)  

## 项目结构 
这里我们使用了平铺的文件结构，最主要的原因也是比较简单，没有太多的组件，不如直接平铺直观一些。  

这是demo的整体系统架构图  

![系统架构](https://images.chimission.cn/blog/baojingjiagou.png)  

项目整体运行在 `docker` 上，负责收集数据的 `prometheus` 运行在docker里, 负责报警的`alertmanager` 运行在docker里，负责暴露机器数据的 `node_exporter` 也运行在docker里，然后使用 `docker-compose` 做容器管理，用 `python` 的web框架 `flask` 写了一个 `消息解析器` 作为中间人将报警信息转化为钉钉机器人可以识别的格式转发送给钉钉机器人，并且通过 `dockerfile`编译进docker里运行:-) 完全的docker化，只需要安装一个`docker`和 `docker-compose` 就可以部署整个demo，不过话说回来现在应该没有哪个机器上没有docker了吧:-)  

## 各组件作用

### docker
基础设施，用来运行各个服务。

### prometheus
Prometheus 是一个免费的开源软件，用于实时系统和事件监控和警报。2016 年成为云原生计算基金会的一个项目。目前已经成为了云原生时代监控方案的事实标准。Prometheus 擅长记录纯数字为主的时序数据。他不仅适合监控服务器的指标，还适合监控服务中频繁动态变化的指标。在微服务的架构中，他有一个很明显的优势，支持多维数据的收集和查询。使用 PromQL 查询数据  

### node_exporter
它负责收集数据将数据暴露给 prometheus，因为 prometheus 某些时候没有办法直接拿到想要的数据，所以需要第三方程序来辅助，node_exporter 就是这样的第三方程序。
它采集对应主机的 CPU、内存、磁盘等信息，暴露给 prometheus。

### alertmanager
alertmanager 主要用于接收 Prometheus 发送的告警信息，它支持丰富的告警通知渠道，而且很容易做到告警信息进行去重，降噪，分组，策略路由，是一款前卫的告警通知系统。  
Prometheus 生态中的警报是在 Prometheus Server 中计算警报规则(Alert Rule)并产生的，而所谓计算警报规则，其实就是周期性地执行一段 PromQL，得到的查询结果就是警报。  
只是，当 Prometheus Server 计算出一些警报后，它自己并没有能力将这些警报通知出去，只能将警报推给 Alertmanager，由 Alertmanager 进行发送。这个切分，一方面是出于单一职责的考虑，让 Prometheus "do one thing and do it well", 另一方面则是因为警报发送确实不是一件"简单"的事，需要一个专门的系统来做好它。可以这么说，Alertmanager 的目标不是简单地"发出警报"，而是"发出高质量的警报"。这里我们只是用它的基础功能，发送报警即可。

#### dingtalk
一个用python编写的消息解析器，alertmanager 目前不能直接集成钉钉机器人，需要存在一个中间件将 alertmanager 格式的消息转化成 钉钉机器人可以解析的格式。

## 各文件作用

### docker-compose.yml
容器部署文件，此文件定义了我们需要运行的容器。文件里定义了4个服务，分别是`dingtalk`消息解析器， `alertmanager` ， `prometheus` ，和 `node_exporter` ()，并且通过 `volumes` 参数将项目中的各种配置文件挂载到容器中，这样如果我们需要修改某个服务的配置文件，只需要修改项目目录下的文件即可而不用进入到容器当中修改。


TODO 未完待续
    