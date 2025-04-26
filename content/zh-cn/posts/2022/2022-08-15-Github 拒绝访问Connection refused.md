---
slug: 58
title: 'Github 拒绝访问 Connection refused'
date: '2022-08-15T02:44:47+08:00'
author: yexca
# layout: post
# permalink: /archives/58
views:
    - '385'
categories:
    - 折腾经验
tags:
    - Github
---

## 引言

今日使用 Git 推送提示`fatal: unable to access 'https://github.com/yexca-VRChat/yexca-VRChat.github.io.git/': Failed to connect to 127.0.0.1 port 1081 after 2074 ms: Connection refused`，重启电脑也没用，遂寻找解决方法 ~~(为什么不让爷访问自己的仓库)~~

## 解决过程

经查阅相关资料得知与代理有关，可我代理是放路由器啊

于是我连接到另一个普通路由器再次推送还是同样问题

然后尝试了设置 Git 的代理也是无果

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

最后想到我 WinXray 貌似好像大概开过吧，然后打开一看，果然开着 pac，关闭后再次推送成功

## 参考文章

[fatal: unable to access 'https://github.com/fmoraless/e-commerce.git/': Failed to connect to 127.0.0.1 port 56832: Connection refuse · Issue #11981 · desktop/desktop](https://github.com/desktop/desktop/issues/11981)

[解决git下载出现：Failed to connect to 127.0.0.1 port 1080: Connection refused拒绝连接错误_点亮～黑夜的博客-CSDN博客](https://blog.csdn.net/weixin_41010198/article/details/87929622)

[git 报错:解决拒接接入问题_Huang_milk的博客-CSDN博客](https://blog.csdn.net/Huang_milk/article/details/121291273?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-121291273-blog-87929622.pc_relevant_multi_platform_whitelistv4&spm=1001.2101.3001.4242.1&utm_relevant_index=3)
