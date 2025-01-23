---
slug: 61
title: Fiddler 抓包 HTTPS
date: '2022-09-01T08:17:32+08:00'
author: yexca
# layout: post
# permalink: /archives/61
views:
    - '181'
categories:
    - 软件工具
    - 编码相关
tags:
    - HTTP
---

## 引言

默认情况下 Fiddler 仅抓包 HTTP，需要设置后才能捕获 HTTPS。现在大部分网站都是使用 HTTPS 或者 HSTS，所以开启 HTTPS 抓包是很必要的

## Fiddler设置

在 *设置-HTTPS* 中勾选 *Capture HTTPS traffic* 即可，下一个 *Ignore server certificate errors(unsafe)* 也可勾选，不过可能不安全吧，然后保存

## 浏览器设置

在勾选抓取 HTTPS 后使用浏览器可能会出现证书错误，即提示连接不安全或不是私密连接之类的，此时需要导入相关证书，以 Firefox 为例

首先下载 Fiddler 证书，在上一部设置处有 *Export root cerificate to Desktop*，点击即可将证书导出至桌面(`~/Desktop/`)处

然后进入Firefox的设置，在*隐私与安全-证书*处查看证书，然后导入刚刚下载的证书，将弹窗的信任什么什么全勾选

导入完成后就可以正常访问了，Fiddler也可以正常抓取HTTPS请求与响应了
