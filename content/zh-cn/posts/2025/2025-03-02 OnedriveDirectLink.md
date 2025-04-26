---
slug: 237
title: '获取 Onedrive 下载直链'
# draft: true
author: yexca
date: '2025-03-02T12:58:57+09:00'
categories:
    - 折腾记录
tags:
    - OneDrive
---

## 引言

最近下载 Onedrive 分享的东西发现不能被 IDM 自动捕捉，而浏览器下载不稳定老是下载失败，于是我便想着看看有没有办法获取下载直链

## 扩展问题

刚开始看到不支持了以为是要重新安装呢，结果删除后去 chrome 扩展商店显示无法安装，啊这，删早了

不过 IDM 作为付费软件，居然没有跟进更新

从 [简悦项目问题](https://github.com/Kenshin/simpread/discussions/6633) 知道只是按钮整了个 `disabled` 属性禁用可还行，去掉属性就能正常安装

## 获取直链

但，IDM 还是无法捕捉，那只能寻找直链了

进入分享页面，预览某个文件，右上角 `Share` - `Copy link` 可以获取文件分享链接，类似于

```markdown
https://xxx-my.sharepoint.com/:u:/r/personal/xxx/Documents/xxx/xxx.rar?csf=1&web=1&e=OTZZbx
```

将其中的 `web` 替换为 `download`，类似于

```markdown
https://xxx-my.sharepoint.com/:u:/r/personal/xxx/Documents/xxx/xxx.rar?csf=1&download=1&e=OTZZbx
```

复制到 IDM 的 `Add URL` 下载即可

---

参考文章: <https://techcommunity.microsoft.com/discussions/onedriveforbusiness/onedrive-direct-download-link/4226744>

---

![yexca-237](https://count.getloli.com/@yexca-237)
