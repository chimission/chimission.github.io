---
title: "利用互联网资源免费部署一个全站 https blog"
date: 2022-06-27T23:09:19+08:00
draft: false
author: "chimission"
categories: ["wirte"]
tags: ["github"]
archives: ['2022-06']
comments: false
description: "使用github pages 构建自己的博客笔记"
url: "/posts/create_blog_with_github_pages/"
---
>介绍了一下此blog是怎么被搭建出来的  &nbsp; :-) 
 <!--more-->

### 起因  
想要搭建一个独立域名的博客用来记录和分享的念头在脑中存在很久了，最近工作不忙，趁机赶紧做起来。  

### 技术选型
需求很明确：容易部署、专注写作、低成本。经过一些调研我最终选择了以下的技术栈  
* [Github Pages](https://docs.github.com/en/pages) *交友网站*
* [Hugo](https://github.com/gohugoio/hugo) *go语言框架*
* [Markdown](https://markdown.lovejade.cn/) *感谢这位作者，为我写作减轻了不少心智负担*🎉️ 
* [七牛云](https://www.qiniu.com/) *感谢七牛提供的免费存储额度*

#### Github Pages
`Github Pages` 是一个可以从自己的 `Github` 源码仓库中直接生成页面的静态站点托管服务，不过它只能托管静态站点，不能运行服务器端源码，比如 `Python` 或者 `Ruby`。最主要的是，它是免费的，而且支持个人域名和`HTTPS`。静态站点可以让使用者更专注于写作本身。部署也很方便，最主要的时间成本是在前期git项目配置上，当项目配置好后，更新一篇文章只需要向关联的仓库push代码即可。  

更多详情可以访问[官方文档](https://docs.github.com/en/pages)。  

#### Hugo
选定了 `Github Pages`，那么就只能选择静态web框架，我之前并没有使用静态web框架的经验，所以对于框架的选择我其实没有什么特别要求，无论选择什么框架我都要从头开始学习，所以选一个社区活跃用户多的即可。因为最近在搞一些go语言的东西，最终我选择了使用go语言编写的 `Hugo`。  

更多详情可以访问[官方文档](https://gohugo.io/about/what-is-hugo/)。 

#### Markdown
这没什么好讲的了，任何一个合格的开发都应该学过这种文本格式（应该是的吧），`Hugo`对`Markdown`的支持也非常好。

### 部署
#### 创建github仓库
1. 首先在自己的github账号里创建一个仓库，起名 `xxx.github.io`，这个名字就是`github`默认赋予你的博客域名，所以至少要想一个让自己满意的名字。  

2. 创建好仓库之后点击 `Setting`-> `Page` 在 `Source` 栏选择网页对应的分支和文件夹，这里我选的是`master`分之和`docs/`目录。如果你使用的默认配置不做更改，在保存之后`github`会自动让你提交一个`commit`，提交之后就可以通过`https://xxx.github.io`这个域名访问网站了，`github`默认为使用者提供了一个首页。`github`甚至为我们提供了https，并且我们什么也不需要做 如果你不想继续折腾，到这一步网站就可以用了，后续编辑根目录下的`index.md`就可以同步更新网站了![](https://images.chimission.cn/blog/github_pages_setting.png)  

3. 如果你有自己的域名，不想使用github默认分配(因为你觉得不够coooool)，就可以在`Custom domain`里填写自己的域名来替换`github`默认提供给你的域名 `xxx.github.io`，比如这里我填写的 `notes.chimission.cn`, 保存之后 `github`会在项目目录下生成一个 `CNAME` 文件， 文件内容就是自定义域名字符串。  

4. 如果执行了`第三步`，那还需要在你的`域名服务商`那里添加一条`CNAME`记录， 使你的域名指向`github`为你分配的默认域名`xxx.github.io`。 ![](https://images.chimission.cn/blog/dns_cname.png)  
比如我这里创建了一条`notes`的记录，指向 `chimission.github.io`，保存之后就可以通过`http://notes.chimission.cn`来访问博客了(注意这里还没有开启https)。如果我们想开启https，也不用担心，在`Setting`-> `Page`页面勾选`Enforce Https`后，`github`会自动为我们申请证书，这一切都不需要我们介入。等待一段时间后就可以通过`https://notes.chimission.cn`来访问网站了。  
更多Https详情可以访问[官方文档](https://docs.github.com/cn/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)。   

#### 使用Hugo进行写作
1. 部署环境搭建好了，现在我们只要修改了仓库里的文件，`github` 就会自动触发更新部署。但是手写html比较反人类，借助静态web框架可以为我们节省大量时间，让我们的注意力专注于写作本身。 

2. `Hugo` 的 [github主页](https://github.com/gohugoio/hugo) 详细介绍了安装方式，根据自己的环境安装即可。 

3. 执行 `hugo new site xxx` 新建一个项目，进入到这个目录，把这个项目和上一节的`github`仓库关联。   

4. 编辑 `config.toml`文件，添加 `publishDir = "docs"`，因为我在上一节设置的`github pages`部署目录是 `docs/`，所以我需要设置一下使 `Hugo` 的输出的html目录为`docs/`，这里因人而异。 

5. 执行 `hugo new posts/firs_notes.md` hugo会在post目录下新建一个md文件，编辑这个文件随意写点东西，然后执行 `hugo server -D` hugo会在本地`localhost`启动服务器，让使用者预览文章效果。并且 `hugo` 会监听文件变动，如果你更新了目录下的文件，`hugo`会实时更新渲染。 

6. 这里推荐一个[在线markdown编写网站](https://markdown.lovejade.cn/)，提供了很多快捷键和预览功能。并且还提供了富文本转换功能，可以将富文本内容粘贴到编辑框内，网站会自动将其转换为md语言，非常不错的功能。

7. 当你结束完写作，想要推送到 `github` 更新网站时， 执行 `hugo -D`， hugo会编译post目录下的文件，在 `docs/` 目录下生成对应的html文件。最后将生成后的代码push到远程的`github`仓库中，稍微等一会，你的网站内容就更新了。

关于`Hugo`的更多使用可以访问[这里](https://gohugo.io/)

#### 对象存储
还有一个关键问题没有讨论 **如何给文章配图**。  
图片一般不会保存在本地，因为相较于文本文件，图片存储占用太大了。最终我选择了七牛云，七牛云的对象存储服务有免费的10G额度，超过了才会收费，对于一个刚起步的网站来说， 七牛云提供的10G免费额度绰绰有余。还有个原因是因为我工作的老东家曾经是七牛云的客户，还算熟悉。 :-)  
并且七牛云的图片链接可以自定义为自己的域名，并且添加https，这里我们可以申请为期一年的免费证书来做https认证。（相应的七牛云也会生成一个CNAME记录，需要你在域名服务商那里做个配置，比如我这里用的是 `images.chimission.cn`）

#### 结果
最终结果产出了本篇文章， 本篇文章使用 `markdown` 编写，使用 `Hogo` 编译， 图片存储在 `七牛云`，整个项目部署在 `github` 上。

谢谢阅读!

