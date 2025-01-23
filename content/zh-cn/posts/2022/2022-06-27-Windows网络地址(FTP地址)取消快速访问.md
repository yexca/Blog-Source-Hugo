---
slug: 50
title: Windows 网络地址( FTP 地址)取消快速访问
date: '2022-06-27T13:31:58+08:00'
author: yexca
# layout: post
# permalink: /archives/50
views:
    - '252'
categories:
    - 日常
tags:
    - Windows
---

好像并没有解决方法，不过可以把快速访问中自己添加的全删除 (恢复默认)

定位至 `C:\\Users\\用户名\\AppData\\Roaming\\Microsoft\\Windows\\Recent\\AutomaticDestinations`，将此文件夹目录下的文件进行备份后全部删除

## 参考文章

[ftp地址不能从快速访问中删除,其他的文件夹可以 – Microsoft Community](https://answers.microsoft.com/zh-hans/windows/forum/all/ftp%E5%9C%B0%E5%9D%80%E4%B8%8D%E8%83%BD%E4%BB%8E/835ef23c-3d44-4fdd-8389-ae47bb696e73)
