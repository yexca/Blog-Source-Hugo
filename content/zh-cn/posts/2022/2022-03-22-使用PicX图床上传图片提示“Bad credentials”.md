---
slug: 35
title: '使用 PicX 图床上传图片提示 "Bad credentials"'
date: '2022-03-22T16:30:12+08:00'
author: yexca
# layout: post
# permalink: /archives/35
views:
    - '374'
categories:
    - 折腾经验
tags:
    - Github
    - PicX
    - 图床
---

## 引言

今日我写文章时，发现 PicX 图床无法使用并提示 `Bad credentials`，于是便寻找解决方法

## 结论

其实就是 Github 的 Token 到期了，然后在邮箱里会收到一封邮件，名为 `[GitHub] Your personal access token has expired`  
邮件有三行，第二行 `If this token is still needed`，后面有个链接，点击打开并重新创建即可

注意设置 `Expiration` 即 Token 期限  
重新创建后需要在 PicX 将图床配置重置下

具体参考：[使用PicX自建免费图床 – yexca’Blog](https://blog.yexca.net/archives/27)
