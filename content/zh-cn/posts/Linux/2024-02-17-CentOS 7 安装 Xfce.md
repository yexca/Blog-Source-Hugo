---
slug: 153
# layout: post
title: 'CentOS 7 安装 Xfce'
author: yexca
date: 2024-02-17T22:51:34+08:00
# permalink: /archives/153
categories:
    - 折腾经验
tahs:
    - Linux
    - CentOS
---

~~这是什么时候写的文章啊（~~

## 查看是否有 Xfce 组

```bash
yum grouplist
```

如果没有，需要安装额外包 yum 源

```bash
yum install epel-release -y
```

## 安装 X Window system

```bash
yum groupinstall "X Window system"
```

## 安装 Xfce

```bash
yum groupinstall xfce
```

## 安装中文字体 (楷体)

```bash
yum install cjkuni-ukai-fonts
```

## 进入 Xfce 桌面

```bash
systemctl isolate graphical.target
```

## 参考文章

[CentOS 7安装Xfce桌面环境过程_qq_28641401的博客-CSDN博客](https://blog.csdn.net/qq_28641401/article/details/99428192)
