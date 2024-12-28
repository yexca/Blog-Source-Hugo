---
slug: 7
title: '关于在使用 navicat 连接 SQL server 的一些问题'
date: '2021-11-07T23:41:46+08:00'
author: hiyoung
# layout: post
# permalink: /archives/7
views:
    - '200'
categories:
    - 编程基础
    - 数据库
---

> 该文章由 [Hiyoung](https://blog.hiyoung.xyz/) 编写

在安装完 SQL server 和 navicat 后在 navicat 中添加数据库：

1.连接名无要求，按照自己需要命名

2.打开安装好的 SQL server 配置管理器

注意 SQL Server（SQLEXPRESS）要保证在运行中，否则 navicat 无法连接  
双击打开后点击服务，可以看到自己的主机名

3.此时打开 navicat 在主机的地方填上：主机名 \\SQLEXPRESS (格式)

4.用户名填 sa (为安装 SQL server 时的默认用户名，具体 SQL server 网上教程很多可以自己参考) ，密码是自己设置的 (同样在 SQL server 安装时设置的密码)

5.测试连接成功即可使用

注：仅个人在安装过程中遇到的问题，具体安装教程请参考网络

附上 navicat 15 及注册机：<https://pan.baidu.com/s/1cJ1EZ9Gyz6Jp6J03VqcDHA>

提取码：3n7g
