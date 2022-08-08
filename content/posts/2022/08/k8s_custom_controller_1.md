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

> 自定义资源（Custom Resource） 是对 Kubernetes API 的扩展，不一定在默认的 Kubernetes 安装中就可用。自定义资源所代表的是对特定 Kubernetes 安装的一种定制。 不过，很多 Kubernetes 核心功能现在都用自定义资源来实现，这使得 Kubernetes 更加模块化。

### 创建自定义资源
#### 创建CustomResourceDefinition（CRD）
首先我们需要编写一个 `CRD` 文件，顾名思义，CRD是 `ustomResource`的定义文件，它规定了要创建的自定义资源的各种属性。  

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # 名字必需与下面的 spec 字段匹配，并且格式为 '<名称的复数形式>.<组名>'
  name: ufos.ufocontroller.chimission.io
spec:
  # 组名称，用于 REST API: /apis/<组>/<版本>
  group: ufocontroller.chimission.io
  # 列举此 CustomResourceDefinition 所支持的版本
  versions:
    - name: v1
      # 每个版本都可以通过 served 标志来独立启用或禁止
      served: true
      # 其中一个且只有一个版本必需被标记为存储版本
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                deploymentName:
                  type: string
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 10
            status:
              type: object
              properties:
                availableReplicas:
                  type: integer
  # 可以是 Namespaced 或 Cluster
  scope: Namespaced
  names:
    # 名称的复数形式，用于 URL：/apis/<组>/<版本>/<名称的复数形式>
    plural: ufos
    # 名称的单数形式，作为命令行使用时和显示时的别名
    singular: ufo
    # kind 通常是单数形式的帕斯卡编码（PascalCased）形式。你的资源清单会使用这一形式。
    kind: Ufo
    # shortNames 允许你在命令行使用较短的字符串来匹配资源
    shortNames:
    - uu
```  
这里我们定义了一个名字叫 `Ufo` 的资源类型 :-) ， 它有两个属性字段， `deploymentName`, `replicas`。后面我会用这个自定义资源来控制和 `deploymentName` 同名的 `deployment`，将其下的 `Pod` 数量调整到 `replicas` 指定数量。

然后创建它：
>kubectl apply -f resourcedefinition.yaml  

这样一个新的受 `namespace` 约束的 RESTful API 端点会被创建：
>/apis/ufocontroller.chimission.io/v1/namespaces/*/ufo/...

此 URL 自此可以用来创建和管理定制对象。对象的 `kind` 将是来自你上面创建时 所用的 `spec` 中指定的 `Ufo`。
创建端点的操作可能需要几秒钟。你可以监测你的 `CustomResourceDefinition` 的 `Established` 状况变为 `true`。

看下结果： 

![结果](https://images.chimission.cn/blog/get_ufo.png)  

#### 创建CustomResource（CR）
有了CRD之后就可以创建CR了

TODO 

