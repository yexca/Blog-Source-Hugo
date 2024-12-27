---
slug: 63
title: 'Fedora 安装 java8(Oracle JDK)'
date: '2022-09-02T17:37:51+08:00'
author: yexca
# layout: post
# permalink: /archives/63
views:
    - '752'
categories:
    - Linux
    - LinuxTools
tags:
    - Fedora
    - Java
---

## 引言

尽管 Fedora 系统自带 java 环境，不过是 OpenJDK 。有时候还是需要使用 Oracle 的

## 下载

进入官网下载：[Java Downloads | Oracle](https://www.oracle.com/java/technologies/downloads/) (下载需要登陆)

找到 *java8-Linux* ，下载 *x64 Compressed Archive* (64位的压缩包版本)

本文章写时文件名为 *jdk-8u341-linux-x64.tar.gz*

## 移到相应目录

1. 首先创建一个 java的 目录，在 */usr/local* 中

```bash
sudo mkdir -p /usr/local/java
```

2. 复制文件到此目录

假设下载的文件在 *~/Downloads* ，进入下载目录

```bash
cd Downloads
```

然后复制到上述目录

```bash
sudo cp -r jdk-8u341-linux-x64.tar.gz /usr/local/java
```

## 解压缩安装文件

3. 切换到 java 目录

```bash
cd /usr/local/java
```

4. 解压缩安装文件

```bash
sudo tar xvzf jdk-8u341-linux-x64.tar.gz
```

## 配置$PATH

5. 在 */etc/profile* 末尾添加以下内容

```bash
JAVA_HOME=/usr/local/java/jdk1.8.0_341
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export JAVA_HOME
export PATH
```

## 更新可用 java 版本列表

6. 直接运行以下命令

```bash
sudo update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/jdk1.8.0_341/bin/java" 1
```

```bash
sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.8.0_341/bin/javac" 1
```

```bash
sudo update-alternatives --install "/usr/bin/javaws.itweb" "javaws.itweb" "/usr/local/java/jdk1.8.0_341/bin/javaws.itweb" 1
```

## 生效配置文件

7. 首先重新加载系统范围的 PATH 文件

```bash
source /etc/profile
```

8. 重启系统

```bash
reboot
```

## 切换 java 版本

可以运行命令查看 java 版本

```bash
java -version
```

9. 使用以下指令切换

```bash
sudo alternatives --config java
```

当前使用的 java 版本前会有`+`，找到相应版本，输入数字选择即可

## 参考文章

[如何在Fedora{OpenJDK 和 Oracle JDK}上安装Java？](https://www.lsbin.com/9422.html)