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

作者目前工作中所维护的服务大部分跑在k8s上，在监控方面使用了官方推荐的技术栈`prometheus`和`grafana`。`prometheus`负责收集数据，`grafana`负责展示数据  

![监控面板](https://images.chimission.cn/blog/granfa.jpg)  

另外我们还集成了钉钉机器人，能够在`prometheus`采集到不正常的数据时能通过钉钉及时通知到对应的开发同事，例如下图，`prometheus`在`k8s`的某个`namespace`下给定的内存已经快被使用完之前发送了一条消息到我们的报警群，提醒我同事该为它加内存资源了:-)。 


![报警](https://images.chimission.cn/blog/k8s_dingding.jpg)   

接下来让我们用一个demo项目来演示这套系统是如何运作的。

未完待续