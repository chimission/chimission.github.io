---
title: "如何在k8s中使用自定义资源（1）"
date: 2022-08-04T17:58:17+08:00
draft: false
author: "chimission"
categories: ["k8s"]
tags: ["k8s"]
archives: ['2022', '2022-08']
comments: false
description: ""
url: "/posts/k8s_custom_controller_1/"
---
>文本是系列文章第一篇，讲述如何在k8s中使用自定义资源
 <!--more-->

### 资源是什么
我们应该先明确一个概念，在 `k8s` 中 `资源` 是什么？ 尽管有很多人曾经编写过 `自定义资源`， 但是对于什么是`资源`并没有很清楚的认知。在[官网中](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)的定义  
> A resource is an endpoint in the Kubernetes API that stores a collection of API objects of a certain kind; for example, the built-in pods resource contains a collection of Pod objects.  

> 资源（Resource） 是 Kubernetes API 中的一个端点， 其中存储的是某个类别的 API 对象 的一个集合。 例如内置的 pods 资源包含一组 Pod 对象。 

`k8s` 中的所有内容都被抽象为 `资源`，如 `Pod`、`Service`、`Node` 等都是资源。`对象` 是 `资源`的实例，是持久化的实体。如某个具体的 `Pod`、某个具体的 `Node`。`k8s` 使用这些实体去表示整个集群的状态。对象的创建、删除、修改都是通过  `Kubernetes API` ，也就是  `Api Server` 组件提供的 `API` 接口，这些是 `RESTful` 风格的 `Api`，与 `k8s` 的 `万物皆对象` 理念相符。命令行工具 `kubectl`，实际上也是调用 `kubernetes api`。`K8s` 中的资源类别有很多种，`kubectl` 可以通过配置文件来创建这些 `对象`，配置文件更像是描述对象 `属性` 的文件，配置文件格式可以是 `JSON` 或 `YAML`。

### 自定义资源是什么
> A custom resource is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation. It represents a customization of a particular Kubernetes installation. However, many core Kubernetes functions are now built using custom resources, making Kubernetes more modular.  

> 自定义资源（Custom Resource） 是对 Kubernetes API 的扩展，不一定在默认的 Kubernetes 安装中就可用。自定义资源所代表的是对特定 Kubernetes 安装的一种定制。 不过，很多 Kubernetes 核心功能现在都用定制资源来实现，这使得 Kubernetes 更加模块化。