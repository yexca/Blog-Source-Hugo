---
slug: 71
title: 'Fedora 安装 Komga 折腾记录'
date: '2022-09-14T18:47:16+08:00'
author: yexca
# layout: post
# permalink: /archives/71
views:
    - '472'
categories:
    - 日常
    - 折腾记录
tags:
    - Docker
---

## 引言

为了在内网更方便得看漫画

## IP 设置

路由器为 OpenWRT 系统

### 路由器设置

在 *网络- IP/MAC 绑定* 将电脑绑定一个固定的 IP

### Fedora 设置

因为我设置的 IP 与自动分配的不一致 (有线连接) ，固需要手动修改

在 *网络-设置* 的 *身份* 加上 *MAC 地址*，*IPv4* 改为 *手动*，地址依次为 *IP、255.255.255.255、路由器 IP*，*DNS* 加上 *路由器 IP*，未取消勾选自动

### 域名劫持

尽管可以通过 IP 直接访问，但是有一个域名会更加好记吧

在路由器 *网络-主机名* 的 *主机名* 处填入域名，*IP 地址* 处填入电脑的 IP

## 安装 Docker

安装了有 GUI 的 Docker Desktop

### 设置仓库

```bash
dnf -y install dnf-plugins-core
```

```bash
sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
```

### 下载 RPM 包

在 [官网 Download 处](https://docs.docker.com/desktop/install/linux-install/) 下载

下载完成后直接双击安装

## 安装 Komga

### Docker 设置

* 文件共享设置

在 Docker Desktop 的 *设置-Resources-File sharing* 添加漫画路径

*注：如果共享的目录在下次启动不存在 (未挂载) ，docker 将无法正常启动*

* 网络设置

不清楚是否非必须，在 *设置-Resources-Network* 设置为了自己的网段

### 命令行安装

直接在 shell 敲入

```bash
docker run \
  --name=komga \
  --user 1000:1000 \
  -p 2333:8080 \
  -v /home/yexca/komga/config:/config \
  -v /home/yexca/komga/data:/data \
  --restart unless-stopped \
  gotson/komga:latest
```

* -p

前一个为本机映射端口，后一个为容器

* -v

文件映射，将本机的目录 (/home/yexca/komga/config) 映射到容器的 (/config)

注：无法映射本机的隐藏文件 (以 `.` 开头的文件)

### GUI 安装

在上一步做完后 Docker Desktop 的 *Imags* 会多出一个 *gotson/komga*，点击 *run*，然后配置

* 第一行：名字

* Ports：映射到本机的端口，比如 80，这样就可以直接域名访问了

* Volumes：路径映射

* Environment variables：环境变量，此处用不到

### 检测是否运行

使用命令查看

```bash
docker ps -a
```

## 防火墙配置

打开端口

```bash
firewall-cmd --zone=public --add-port=80/tcp
```

加载配置

```bash
firewall-cmd --reload
```

查看端口打开状态

```bash
firewall-cmd --zone=public --query-port=80/tcp
```

可能需要添加服务

```bash
firewall-cmd --add-service=http
```

实在不能开就用 GUI 吧 ~~(一开始就用会更快吧)~~

```bash
sudo yum install firewall-config
```

## 参考文章

[Install Docker Desktop on Fedora - Docker Documentation](https://docs.docker.com/desktop/install/fedora/)

[【Docker】Error response from daemon: invalid mount config for type "bind": bind source path does not exist - Qiita](https://qiita.com/ucan-lab/items/7c0ca7db70deb56ad4fa)

[Run with Docker - Komga](https://komga.org/installation/docker.html#version-tags)

[简约但绝不简单的Komga-老苏的blog](https://laosu.ml/2021/08/02/%E7%AE%80%E7%BA%A6%E4%BD%86%E7%BB%9D%E4%B8%8D%E7%AE%80%E5%8D%95%E7%9A%84Komga/)

[fedora 28 , firewalld 防火墙控制，firewall-cmd 管理防火墙规则 - xuyaowen - 博客园](https://www.cnblogs.com/xuyaowen/p/linxu_firewalld.html)

[Fedora防火墙配置 - 上官飞鸿 - 博客园](https://www.cnblogs.com/jackadam/p/9483381.html)

[原神自动签到(Linux服务器Docker) - yexca'Blog](http://blog.yexca.net/archives/47)

[Fedora 打开8080端口_chunqi zhi的博客-CSDN博客](https://blog.csdn.net/zhichunqi/article/details/80488567)