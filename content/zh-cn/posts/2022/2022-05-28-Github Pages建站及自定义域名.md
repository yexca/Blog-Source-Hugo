---
slug: 44
title: 'Github Pages建站及自定义域名'
date: '2022-05-28T15:30:43+08:00'
author: yexca
# layout: post
# permalink: /archives/44
views:
    - '355'
categories:
    - 网站建设
tags:
    - Github
---

## 引言

最近基于Github Pages整了一个[个人页面](https://git.yexca.xyz)，记录一下如何使用Github Pages建站以及自定义域名

本文没有建站系统等，~~因为我也就写了一个Markdown文件~~

## 创建Github仓库

首先需要注册一个[Github](https://github.com)账号，登录后[创建一个新仓库](https://github.com/new)

其中**Repository name**为`username.github.io`，例如我的Github用户名为`yexca`，则输入`yexca.github.io`

## Git环境安装

Windows环境直接从Git官网直接[下载安装程序](https://git-scm.com/downloads)即可

安装完成后，打开Git Bash，在命令行输入以下代码

```bash
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

其中`“Your Name”`替换为您的姓名，`“email@example”`替换为您的邮箱

例如我的

```bash
$ git config --global user.name "yexca"
$ git config --global user.email "yexca@duck.com"
```

## Github Desktop

### 安装

如果您熟悉Git的操作~~熟悉Git操作怎么会来看我的文章~~，这一步可以跳过

直接进入[Github Desktop官网](Https://desktop.github.com)下载安装即可

### 克隆仓库

打开Github Desktop后登录Github账号选择一个空文件夹将上一步创建的仓库克隆到本地

然后软件会出现一个仓库变动界面，右方会有一些快捷操作

![软件界面](https://cdn.jsdelivr.net/gh/yexca/picx-images-hosting@master/2022/05-GithubPages建站/image.43qnbq0gw800.webp)

这里我使用VS Code，点击“Open in Visual Studio Code”在VS Code打开

### 建立网站

这里直接创建一个`README.md`文件使用[Markdown编辑](https://blog.yexca.net/archives/43)(~~这里顺便放一个我写的Markdown笔记~~)

编辑完成并保持后在`Github Desktop`点击`Commit to main`，然后点击右方`Push origin`即可

至此访问`username.github.io`即可看到网站内容~~如果没看到请等一段时间~~

## 自定义域名

### Github Pages

进入刚刚建立的仓库页面，点击`Settings`，左侧找到`Pages`，在`Custom domain`处输入自定义域名然后点击`Save`

注：在这里可以进行Jekyll建站主题的选择

### DNS

在域名的DNS解析处添加一个`CNAME`类型解析，将域名指向`username.github.io`，其中`username`为您的Github用户名

### HTTPS

这里我用Github的不知为何没成功，于是使用[Cloudflare](https://cloudflare.com/zh-cn/)

在DNS解析处启用代理，然后在`SSL/TLS`的`边缘证书`处将`始终使用HTTPS`打开即可

## 其他建站

因无博客需求，我只是写一个简单的文件，如果是建立博客之类的可以使用一些建站工具

* [Jekyll](http://jekyllrb.com/) Github官方支持的建站
* [VuePress中文网 ](http://caibaojian.com/vuepress/) Markdown推荐

* Gitbook 适合建立文档类网站
* [LOFFER](https://fromendworld.github.io/LOFFER/)
* [Gridea](https://gridea.dev/) 一个静态博客写作客户端
* [Hexo](https://hexo.io/zh-cn/) 快速、简洁且高效的博客框架
* [Hugo](https://gohugo.io/)

## 参考文章

[GitHub Pages 快速入门 - GitHub Docs](https://docs.github.com/cn/pages/quickstart)

[GitHub Pages博客：自定义域名，HTTPS，CAA — 浮云的博客](https://last2win.com/2020/02/21/github-pages-https/)

[GitHub Pages 搭建教程](https://sspai.com/post/54608)

[安装Git - 廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600/896067074338496)
