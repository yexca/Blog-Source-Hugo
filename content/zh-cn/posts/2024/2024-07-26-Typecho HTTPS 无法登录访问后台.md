---
slug: 175
# layout: post
title: 'Typecho HTTPS 无法登录访问后台'
author: yexca
date: 2024-07-26T22:22:22+08:00
# permalink: /archives/175
categories:
    - 折腾经验
tags:
    - Typecho
    - 故障排查
---  

## 引言

之前在移植 typecho 到 Docker 容器后，在开启 HTTPS 后登录后台将会报错，而将 HTTPS 关闭后则可以正常访问。因为在之前非 Docker 部署的时候可以正常访问，我以为是 Docker 网络的问题，而当时的修改是一次性的，不会再进行更新，所以我在关闭 HTTPS 修改完成后便不再管理。如今再次使用 typecho 再次遇到相同问题，考虑到需要更新文章，遂寻解决之法

## 解决方法

解决方法也非常简单，在 `data/config.inc.php` 文件最后添加如下代码

```php
define('__TYPECHO_SECURE__', true);
```

然后重新启动即可

## 成因分析

参考资料中猜测是因为用户与浏览器之间是 HTTPS 交互，但实际上 PHP 接收到的是来自 Cloud Flare 的 HTTP 交互，所以 PHP 使用了 HTTP 进行响应从而构成了这个问题

## 参考文章

[Typecho HTTPS 无法登陆后台](https://blog.lucien.ink/archives/523/)
